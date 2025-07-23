# Meter Payload

### Meter Payload Overview

Every meter in the  periodically emits a **meter payload**—a compact, signed bundle of data that proves both freshness (via a nonce) and the amount of electricity measured (in kWh, to micro‑kWh precision). By packing fields into fixed‑length byte‑segments, we minimize bandwidth requirement and storage costs. Beyond the core data, the payload format is also designed to be **extendable**: future fields (e.g. voltage, geo-location, device identifiers) can be tacked on without breaking the primary payload parser.

At its heart, the meter payload is:

```python
4 bytes   ‖ 4 bytes       ‖ 64 bytes
└─ Nonce ─┴─ Energy int ──┴─ Ed25519 signature ──┘
```

With extension enabled, the trailing bytes become:

```python
… ‖ 2 bytes voltage ‖ 32 bytes device ID ‖ 3 bytes longitude ‖ 3 bytes latitude
```

***

### Core Fields: Nonce & Energy

#### Nonce (4 bytes)

The **nonce** is a strictly increasing 32‑bit unsigned integer. It guards against replay: every new reading increments this counter. Encoded as big‑endian, it occupies bytes 0–3 of the payload.

#### Energy (4 bytes)

The **energy** field records the cumulative kilowatt‑hours (kWh) measured, with **six decimal places** of precision. To pack this into 4 bytes (a 32‑bit unsigned integer):

1.  Take the floating‑point kWh value, e.g.

    > `12.345678 kWh`
2.  Multiply by 10⁶ to shift six decimals:

    > `12.345678 × 10⁶ = 12,345,678`
3. Convert to a 32‑bit big‑endian integer.

Those 4 bytes occupy offsets 4–7 of the payload.

***

### Ed25519 Signing & Verification

Once the first 8 bytes (nonce‖energy) are assembled, the meter uses its **Ed25519** private key to compute a 64‑byte signature over exactly those 8 bytes.

*   **Signing**

    ```
    message   = nonce_bytes || energy_bytes     (8 bytes total)
    signature = Ed25519.sign(privateKey, message)  (64 bytes)
    payload   = message || signature
    ```
*   **Verification**

    ```
    received_nonce_energy = payload[0..7]
    received_sig          = payload[8..71]
    valid = Ed25519.verify(publicKey, received_nonce_energy, received_sig)
    if valid:
        decode_nonce(received_nonce_energy[0..3])
        decode_energy(received_nonce_energy[4..7])
    else:
        reject("Invalid signature")
    ```

Since signature covers only the **fresh** data, any tampering (or out‑of‑order replay) is immediately detected. Once verified, the nonce and raw energy integer are unpacked and then the energy value is re‑scaled by 10⁻⁶ to recover the original kWh reading.

***

### Payload Layout at a Glance

```mathematica
Byte Offset  Length  Field        Description
–––––––––––  ––––––  ––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
 0           4 B     Nonce        Big‑endian uint32, strictly increasing counter
 4           4 B     Energy       Big‑endian uint32, kWh × 10⁶
 8           64 B    Signature    Ed25519 signature over bytes 0–7
[72]         …       Extension    Optional extra fields (see below)
```

***

### Example Payload (Hex)

Imagine a meter reporting nonce= 42, energy= 7.000123 kWh:

1. **Nonce** = 42 → `0x0000002A`
2. **Energy** = 7.000123 → 7.000123 × 10⁶ = 7,000,123 → `0x006ABF53`
3. **Message** = `0000002A‖006ABF53`
4.  **Signature** = 64 bytes (example truncated)

    ```python
    A1B2C3… (64 bytes)
    ```
5.  **Full payload** (hex):

    ```css
    0000002A006ABF53A1B2C3…[total length = 8+64 = 72 bytes]
    ```

***

### Extensions: Voltage, Identifier & Geolocation

To support richer on‑chain logic or off‑chain analytics, we allow a fixed extension block appended to the core payload. Parsers that don’t recognize it can simply ignore any trailing bytes beyond offset 72.

1. **Voltage** (2 bytes)
   * Precision: **0.1 V**
   * Encoding: big‑endian uint16 of (V × 10)
   * Offsets: 72–73
2. **Device Identifier** (32 bytes)
   * Either the meter’s **public key** (32 bytes) or an on‑chain **tokenId** (padded/truncated to 32 bytes)
   * Offsets: 74–105
3. **Longitude** (3 bytes)
   * Precision: **5 decimal places**
   * Encoding: big‑endian uint24 of (degrees × 10⁵ plus offset)
   * Offsets: 106–108
4. **Latitude** (3 bytes)
   * Same scheme as longitude
   * Offsets: 109–111

A full‐length extended payload therefore runs to 112 bytes.

***

### Illustrative Diagram

```mathematica
┌───────────────────┬───────────────────┬──────────────────────────┐
│ Bytes 0–3         │ Bytes 4–7         │ Bytes 8–71               │
│ ┌───────────────┐ │ ┌───────────────┐ │ ┌──────────────────────┐ │
│ │ Nonce (4 B)   │ │ │ Energy (4 B)  │ │ │ Signature (64 B)     │ │
│ └───────────────┘ │ └───────────────┘ │ └──────────────────────┘ │
├───────────────────┴───────────────────┴──────────────────────────┤
│ (Optional) Extension – Voltage, ID, Longitude, Latitude         │
└─────────────────────────────────────────────────────────────────┘
```
