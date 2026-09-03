---
description: 'Defining: Rollup transaction data'
cover: ../../.gitbook/assets/uplink.png
coverY: 275.4842924501505
---

# Data Payload Format

## Overview

At the heart of the protocol is the meter payload, a tightly packed binary data structure that carries essential information about energy consumption. By packing fields into fixed‑length byte‑segments, we minimize bandwidth requirement and storage costs. Beyond the core data, the payload format is also designed to be **extendable**: future fields (e.g. voltage, geolocation, device identifiers) can be tacked on without breaking the primary payload parser.

### Payload Structure

At its core, the meter payload is a tightly packed sequence of bytes designed for minimal overhead and efficient transmission. The base payload consists of a nonce, an energy value, and a digital signature. This structure is visualized below:

```
Byte Offset  Length  Field        Description
–––––––––––  ––––––  ––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
 0           4 B     Nonce        Big‑endian uint32, strictly increasing counter
 4           4 B     Energy       Big‑endian uint32, kWh × 10⁶
 8           64 B    Signature    Ed25519 signature over bytes 0–7
[72]         …       Extension    Optional extra fields (see below)
```

***

### Core Payload Fields

```python
4 bytes   ‖ 4 bytes       ‖ 64 bytes
└─ Nonce ─┴─ Energy int ──┴─ Ed25519 signature ──┘
```

The two most critical components of the core payload are the nonce and the energy value.

* **Nonce (4 bytes)**: The **nonce** is a strictly increasing 32‑bit unsigned integer (number used once). It ensures that each payload is unique, preventing replay attacks where an attacker could resend a previously captured payload: every new reading increments this counter. Encoded as big‑endian, it occupies bytes 0–3 of the payload.
* **Energy (4 bytes)**: The **energy** field records the cumulative kilowatt‑hours (kWh) measured, with **six decimal places** of precision. However, for the purpose of data transmission and signing, this decimal value is converted into an integer. This is achieved by multiplying the kWh value by 10<sup>6</sup> to shift six decimals and converted to a 32‑bit big‑endian integer. Those 4 bytes occupy offsets 4–7 of the payload.

#### Signing and Verification

To ensure the integrity and authenticity of the meter data, each payload is digitally signed. The process ensures that the data originates from a trusted meter and has not been tampered with in transit. The signature is generated using the **Ed25519 digital signature algorithm**, known for its high performance and strong security guarantees.

{% hint style="info" %}
you can learn more about ED25519 and other comparable schemes [here](https://soatok.blog/2022/05/19/guidance-for-choosing-an-elliptic-curve-signature-algorithm-in-2022/)\
The blog also provides a rational for our adoption of the ED25519 signature scheme.
{% endhint %}

Once the first 8 bytes (nonce‖energy) are assembled, the meter uses its private key to compute a 64‑byte signature over exactly those 8 bytes. The resulting 64-byte signature is appended to the nonce and energy values, forming the complete 72-byte core payload.

```
message   = nonce_bytes || energy_bytes     (8 bytes total)
signature = Ed25519.sign(privateKey, message)  (64 bytes)
payload   = message || signature
```

To verify the payload, a verifier first unpacks the first 8 bytes message of the payload (containing the nonce and energy), followed by 64 bytes that contains the corresponding signature to the message. Using the meter's public key, the verifier checks if the signature is valid for the extracted 8-byte message. Once verified, the nonce and raw energy integer are unpacked from the first 8 bytes of the payload and then the energy value is re‑scaled by 10<sup>-6</sup> to recover the original kWh reading.

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

***

### Payload Extension

To support richer logic and analytics, the payload can be extended to include additional data points beyond the core nonce and energy values. When extended, the additional fields are appended after the core payload's signature. The extension provides supplementary information such as voltage measurement, the meter's identity and location information. Parsers that don’t recognize it can simply ignore any trailing bytes beyond offset 72.&#x20;

{% hint style="info" %}
It is important to note that the extension data is **not** covered by the core signature. The primary signature only guarantees the integrity of the nonce and energy.&#x20;
{% endhint %}

#### Common Extension Data

1. **Voltage** (2 bytes): The current voltage reading.
2. **Identifier** (32 bytes): A unique identifier for the meter, which can be either its public key or a specific Token ID.
3. **Longitude** (3 bytes): The geographic longitude of the meter.
4. **Latitude** (3 bytes): The geographic latitude of the meter.

```python
… ‖ 2 bytes voltage ‖ 32 bytes device ID ‖ 3 bytes longitude ‖ 3 bytes latitude
```

#### Extension Data Encoding

The data within the extension block follows a similar integer-encoding scheme to maintain efficiency.

* **Voltage:** The voltage has a precision of one decimal place. It is multiplied by 10 to be encoded as a 2-byte integer. **Example:** A voltage of `230.5 V` is encoded as `2305`.
* **Longitude & Latitude:** Geographic coordinates have a precision of five decimal places. They are multiplied by 10<sup>5</sup> to be encoded as 3-byte integers. **Example:** A latitude of `-4.81667` is encoded as `-481667`.
