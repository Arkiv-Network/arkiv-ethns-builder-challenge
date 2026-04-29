# Reference Pattern — DePIN Sensor Archive

A starter pattern for the **DePIN** theme. Use it as the on-ramp; depart from it as your build demands.

The goal: a fleet of devices submits time-stamped, signed readings to Arkiv. Anyone can query the readings by device, time window, and metadata. Operators retain ownership of their devices and data.

> Code samples are illustrative. Confirm exact SDK API against the [Arkiv TypeScript SDK docs](https://arkiv.network/getting-started/typescript) — version `@arkiv-network/sdk` v0.6.0 or newer.

---

## Entity schema

Three entity types — two required, one strongly recommended.

### `Device`

One per physical device or operator. Long-lived.

| Field | Type | Notes |
|------|------|-------|
| `deviceType` | string | `air-quality`, `solar-meter`, `gateway`, ... |
| `serial` | string | Hardware serial / unique ID |
| `location` | object | `{ lat, lng, altitude, h3 }` — coarse geo for queries |
| `capabilities` | string[] | What this device measures |
| `pubkey` | bytes | Device signing key (so readings can be verified independently) |
| `owner` | wallet address | Operator wallet |
| `expiration` | timestamp | Long — devices last years |

### `Reading`

The frequent entity. **High volume.**

| Field | Type | Notes |
|------|------|-------|
| `deviceId` | reference → Device | Parent device |
| `timestamp` | timestamp | When the reading was taken (queryable) |
| `readingType` | string | `pm25`, `temperature`, `kwh`, `signal-strength`, ... |
| `value` | number / bytes | The reading itself |
| `unit` | string | `μg/m³`, `°C`, `kWh`, ... |
| `signature` | bytes | Device's signature over `(timestamp, value, unit, deviceId)` |
| `tags` | string[] | Optional — `quality:high`, `interpolated`, ... |
| `expiration` | timestamp | **Short for raw readings** (e.g., 30 days) |

### `Aggregate` (recommended)

Roll-ups of readings. Computed on-device or off-device, written before raw readings expire.

| Field | Type | Notes |
|------|------|-------|
| `deviceId` | reference → Device | |
| `bucket` | string | `2026-05-16T10:00Z` (hourly) or `2026-05-16` (daily) |
| `readingType` | string | Same as raw readings |
| `aggregateType` | string | `avg`, `sum`, `max`, `p95`, ... |
| `value` | number | Computed value |
| `count` | int | Number of raw readings folded in |
| `expiration` | timestamp | **Long** — hourly: 1y, daily: 5y |

---

## Tiered expiration matters here

This is the canonical place where Arkiv's expiration model earns its keep.

| Entity | Volume | Expiration |
|--------|--------|-----------|
| `Reading` (raw) | Highest — many per device per hour | 30–90 days |
| `Aggregate` (hourly) | Medium | 1 year |
| `Aggregate` (daily) | Lower | 5+ years |
| `Device` | One per operator | Long-lived |

The pipeline: raw readings get aggregated before they expire; raw readings expire (cheap on mainnet); aggregates persist. Judges look for this on the **Expiration dates** sub-criterion.

---

## Minimal code outline (TypeScript)

```typescript
import { Arkiv } from "@arkiv-network/sdk";
import { sign, verify } from "@noble/ed25519";

const arkiv = new Arkiv({ rpc: "https://kaolin.hoodi.arkiv.network/rpc" });

// 1. Operator registers a device
async function registerDevice(deviceConfig: {
  deviceType: string;
  serial: string;
  location: { lat: number; lng: number };
  pubkey: Uint8Array;
}) {
  return arkiv.entities.create({
    type: "Device",
    payload: deviceConfig,
    expiration: addYears(new Date(), 5),
  });
}

// 2. Device submits a reading (signed by device key)
async function submitReading(
  deviceId: string,
  reading: { readingType: string; value: number; unit: string },
  devicePrivateKey: Uint8Array,
) {
  const timestamp = new Date().toISOString();
  const message = JSON.stringify({ deviceId, timestamp, ...reading });
  const signature = await sign(message, devicePrivateKey);

  return arkiv.entities.create({
    type: "Reading",
    payload: { ...reading, timestamp, signature },
    references: { deviceId },
    expiration: addDays(new Date(), 60), // raw readings expire fast
  });
}

// 3. Query readings by device, time window, type
async function queryReadings(opts: {
  deviceId?: string;
  readingType?: string;
  since?: Date;
  until?: Date;
}) {
  return arkiv.entities.query({
    type: "Reading",
    filters: {
      "references.deviceId": opts.deviceId,
      "payload.readingType": opts.readingType,
      "payload.timestamp": opts.since && opts.until
        ? { gte: opts.since, lte: opts.until }
        : undefined,
    },
    orderBy: { "payload.timestamp": "asc" },
    limit: 1000,
  });
}

// 4. Compute hourly aggregate (run on a cron, before raw readings expire)
async function rollupHourly(deviceId: string, readingType: string, hourBucket: Date) {
  const hourEnd = new Date(hourBucket.getTime() + 60 * 60 * 1000);
  const readings = await queryReadings({ deviceId, readingType, since: hourBucket, until: hourEnd });
  if (readings.length === 0) return;

  const values = readings.map(r => r.payload.value);
  const avg = values.reduce((a, b) => a + b, 0) / values.length;

  return arkiv.entities.create({
    type: "Aggregate",
    payload: {
      bucket: hourBucket.toISOString(),
      readingType,
      aggregateType: "avg",
      value: avg,
      count: readings.length,
    },
    references: { deviceId },
    expiration: addYears(new Date(), 1),
  });
}

// 5. Verify a reading came from the device that owns it (off-chain anti-spoofing)
async function verifyReading(readingId: string): Promise<boolean> {
  const reading = await arkiv.entities.get({ type: "Reading", id: readingId });
  const device = await arkiv.entities.get({ type: "Device", id: reading.references.deviceId });
  const message = JSON.stringify({
    deviceId: reading.references.deviceId,
    timestamp: reading.payload.timestamp,
    readingType: reading.payload.readingType,
    value: reading.payload.value,
    unit: reading.payload.unit,
  });
  return verify(reading.payload.signature, message, device.payload.pubkey);
}
```

---

## Demo flow

1. Register 1+ devices (real ESP32 / Raspberry Pi, or simulated in a Node script).
2. Stream readings for at least 1 hour at sub-minute frequency. **Show the volume** — judges should see the system handles meaningful throughput, not 5 readings total.
3. Run the hourly aggregator. Show that aggregates appear, count matches.
4. Wait until readings approach expiration (or set short expirations for the demo) — show that aggregates remain queryable while raw readings have rolled off.
5. Build a public dashboard or API that surfaces both raw readings (recent) and aggregates (historical), filterable by device + time + reading type.
6. **Verify a reading's signature** on the dashboard — clicking a reading shows it was signed by the device's pubkey, not spoofed.

---

## Where to go from here

- **Real hardware:** Connect an actual sensor — ESP32 with a PMS5003 air-quality sensor is ~$40 and ships in a weekend. Use the device's mobile-app pairing flow to register it on first boot.
- **Map visualisation:** H3 indexing on `Device.location` plus a Mapbox / Leaflet client that pulls live readings.
- **Operator rewards:** A reward calculator that pulls aggregates from Arkiv and computes operator payouts (uptime, reading volume, accuracy vs. nearby devices).
- **Anomaly detection:** Tag readings with `quality:suspect` if they deviate significantly from neighbouring devices in the same H3 cell.
- **Multi-network:** Same schema can serve a Helium-style proof-of-coverage network OR a solar tracker OR a weather network — generalise the `readingType` and you've built the infrastructure for many DePINs at once.

---

## Scoring tips for this theme

- Tiered expiration with a working aggregator pipeline = top marks on **Expiration dates** + **Advanced features**.
- Volume — a system that gracefully handles thousands of readings, not just a handful — = high score on **Data integrity** + **Functionality**.
- Real hardware integration is rare and impressive; if you can pull it off, surface it in the demo.
- Public verifiability of reading signatures = high score on **Arkiv integration depth** (proves the data is operator-owned, not platform-issued).
