---
description: registered protocol identity associated with a physical metering endpoint.
icon: hexagon-vertical-nft
cover: ../.gitbook/assets/Screenshot 2023-12-18 at 05-48-17 1000x Podcast.png
coverY: 132.37112081584854
---

# M3ters

<figure><img src="../.gitbook/assets/twitter-avatar.png" alt=""><figcaption></figcaption></figure>

<div><figure><img src="../.gitbook/assets/png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/[0_0].png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/{0_0}.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/georg.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/kingslays.png" alt=""><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/ichristwin.png" alt=""><figcaption></figcaption></figure></div>

A M3ter is the persistent onchain identity of a smart meter in the M3tering Protocol. It is the protocol object through which a particular physical interface becomes identifiable, ownable, addressable, and usable by smart contracts.

### &#x20;ERC-721 Non-Fungible Tokens&#x20;

Every meter participating in the protocol must be distinguishable from every other meter. Its measurements contribute to a particular energy account, its cryptographic key must be discoverable, and applications need a durable identifier through which to reference it.

An ERC-721 non-fungible token (NFT)  provides these properties in a standard form.

{% hint style="info" %}
NFTs are unique digital assets that cannot be exchanged on a one-to-one basis unlike fungible tokens. Instead, each NFT has a distinct and indivisible identity, making it unique and valuable in its own right. Learn more [here](https://www.investopedia.com/non-fungible-tokens-nft-5115211)
{% endhint %}

Unlike a fungible token, each NFT has a unique `tokenId`. That identity can persist even when the physical asset changes operator, joins a different application, rotates its signing key, or accumulates a longer operating history. In the protocol, non-fungibility reflects a physical reality. Two devices of the same model are not interchangeable in protocol state because each measures a different system boundary and signs with a different key.

The M3ter NFT therefore provides:

* **unique identity** — one stable token ID for one registered meter identity;
* **key discovery** — a canonical onchain location for the device's public key;
* **ownership and administration** — a standard account with authority over defined identity operations;
* **contract composability** — a reference that application contracts can use in agreements and accounting;
* **provenance** — an onchain history of minting, ownership transfers, and key updates; and
* **extensibility** — a link to descriptive metadata and independently issued attestations.

### Technical functions

The protocol separates these responsibilities: a Nouns-derived ERC-721 system handles issuance, ownership, metadata, and governance, while a separate keystore maps the same token IDs to meter public keys. The **M3ter NFT contract** handles issuance, ERC-721 ownership and transfers. The **keystore contract** associates the same token IDs with Ed25519 device public keys.

```
NFT tokenId 42  ↔  keystore slot 42  ↔  account state index 42
```

The keystore does not issue another token. Entry 42 identifies M3ter NFT #42 because authorization is resolved against the NFT contract and the key is stored at the slot derived from that same ID.

> Separating these responsibilities keeps the cryptographic registry small, stable and most importantly lets the signature authentication (during state evaluation) depend on a predictable key layout used to verify Merkel trie proofs, that include the keys in the Ethereum state root.

#### Minting a M3ter

The Nouns-derived issuance system mints M3ters through recurring auctions rather than an unrestricted public mint. Auction settlement assigns the next token ID, transfers the new NFT to the winner, establishes its governance power, and starts the next auction according to the configured rules.

The exact cadence, reserve price, auction duration, founder allocation, renderer, and proceeds recipient are deployment parameters and should be documented from the deployed Nouns Builder contracts.

#### Registering or rotating the device key

The token owner calls:

```solidity
setPublicKey(uint256 tokenId, bytes32 newKey)
```

The contract binds a 32-byte Ed25519 public key to the token ID and emits:

```solidity
event NewKey(
    uint256 indexed tokenId,
    bytes32 indexed publicKey,
    address from,
    uint256 timestamp
);
```

The corresponding private key remains in the meter's secure environment and must never be published onchain. The meter uses it to sign energy payloads. Verifiers retrieve the registered public key and use it to check that a payload was signed by the device identity associated with the token.

Allowing the owner to change the public key supports commissioning, hardware replacement, recovery, and key rotation without discarding the token's stable identity. Applications that depend on key history should follow `NewKey` events and define how they treat measurements around a rotation. The keystore should also enforce unique keys and invalidate stale reverse mappings during rotation. Otherwise an old key may continue to resolve to a token even after `publicKey(tokenId)` has changed. Zero keys and the ambiguity between an absent entry and token ID `0` should be handled explicitly.

#### Looking up identity

The contract exposes both directions of the identity relationship:

```solidity
publicKey(uint256 tokenId) → bytes32
tokenID(bytes32 publicKey) → uint256
```

The first resolves a known M3ter to its current key. The second resolves a public key to its token ID. This allows gateways, provers, explorers, and applications to begin with either identifier.

#### Ownership and discovery

As an ERC-721, the contract supports the standard ownership, approval, and transfer interface. It additionally exposes enumeration functions for discovering the total token supply, tokens by global index, and tokens owned by an account.

Ownership determines who may perform owner-authorized NFT operations such as changing the registered key. Application-specific authority should be checked separately. A controller should not treat every NFT transfer as an unconditional physical operating command unless the application and installation explicitly define that behavior.

#### Metadata lookup

The contract exposes:

```solidity
tokenURI(uint256 tokenId) → string
```

The returned URI points to descriptive metadata for the M3ter. In the current contract, this URI is supplied during minting and there is no public function for changing it later.

### What does a M3ter NFT represent?

Token ownership and physical ownership may coincide, but they are not automatically the same legal fact. Holding a token proves control of the ERC-721 identity under the contract's rules. Whether it also conveys title to hardware, revenue rights, operating authority, a lease, or another legal interest depends on the agreement and law surrounding a particular deployment.

Applications may deliberately attach such rights to the NFT. For example, a project contract could direct revenue associated with a meter to the current token owner, or require the owner to authorize a key rotation. But those effects arise from the application contract and its real-world agreement; they should not be assumed merely from ERC-721 ownership.

The common element is not the type of equipment. It is that the meter observes energy crossing a defined boundary and can authenticate its reports.

In one sentence

> **A M3ter NFT is the persistent onchain identity and cryptographic key registry for a physical metering endpoint, linking its ownership, descriptive context, verified energy state, and application agreements through one stable token ID.**

The NFT makes the meter addressable. The device signature makes its reports authentic. Together, these functions allow a physical energy resource to participate as a first-class actor in the energy internet.

{% content-ref url="v2.0/" %}
[v2.0](v2.0/)
{% endcontent-ref %}
