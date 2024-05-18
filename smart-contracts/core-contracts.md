---
description: >-
  Overview of the 4 main source codes repositories implemented under the
  M3tering Protocol
cover: >-
  ../.gitbook/assets/Screenshot 2023-12-18 at 05-59-33 (351) Dear Alice -
  YouTube.png
coverY: 114
---

# Core Contracts

{% embed url="https://github.com/orgs/M3tering/repositories" %}

## 🤖 M3ter: ERC-721 contract&#x20;

#### Deployment Address: [<mark style="color:green;">0x39fb420Bd583cCC8Afd1A1eAce2907fe300ABD02</mark>](https://gnosis.nftscan.com/0x39fb420bd583ccc8afd1a1eace2907fe300abd02?module=NFTs) 

M3ter is an ERC-721 (NFT) contract where each NTF represent a smart metering device registerd on-chain by it's manufacturer. Similar to Helium device NFTs on Solana, or Wicrypt device NFTs on polygon.  These devices contain cryptographic secure elements used to sign energy consumption data they measure. the purpose of `keyByTokenId` is to map an NFT's `tokenId` to the cryptographic `PublicKey` for a device (bytes 32 key for ED25519 digital signature scheme), while the `tokenIdByKey` is used to reverse map the above mentioned `PublicKey` to the NTF's `tokenId`

{% embed url="https://github.com/M3tering/M3ter" %}

## 🪙 Solaxy: ERC-20 as ERC-4626 contract

#### Deployment Address: [<mark style="color:green;">0xF4F3c1666E750E014DE65c50d0e98B1263E678B8</mark>](https://gnosis.blockscout.com/token/0xF4F3c1666E750E014DE65c50d0e98B1263E678B8?tab=holders)

Solaxy is an ERC-20 token contract that implements a linear bonding curve with sDAI as the reserve currency. The bonding curve allows users to buy and sell Solaxy tokens directly from/to the contract at a dynamic price determined by the curve's slope.&#x20;

Solaxy extends its functionality by providing support for [`ERC-4626`](https://eips.ethereum.org/EIPS/eip-4626); a tokenized vault interface. This interface allows the Solaxy contract to interact with other DeFi protocols and platforms seamlessly. `ERC-4626` integration enhances the capabilities of Solaxy in the context of token bonding curves.

{% embed url="https://github.com/M3tering/Solaxy" %}

## ⚙️ Protocol: PPA pre-payment contract

#### Deployment Address: [<mark style="color:green;">0x2b3997D82C836bd33C89e20fBaEF96CA99F1B24A</mark>](https://gnosis.blockscout.com/address/0x2b3997D82C836bd33C89e20fBaEF96CA99F1B24A?tab=contact\_code)

Represents the onchain components of the Power Purchase Agreements (PPAs) between the Providers and Off-takers. The `pay` function is executed by the end users to pay xDAI tokens to the owner of their assigned M3ter at the time of payment.

{% embed url="https://github.com/M3tering/Protocol" %}

### 📃 Listings: M3ter ERC-721 marketplace

#### Deployment Address: [<mark style="color:green;">0x1f15510D538bEC23E0ea81600EEe826514866204</mark>](https://gnosis.blockscout.com/address/0x1f15510D538bEC23E0ea81600EEe826514866204)

Simple NFT marketplace contract for the sale of M3ter NFTs. Allows users to list M3ters for free and to purchase other M3ters with xDAI . &#x20;

{% embed url="https://github.com/M3tering/Listings" %}
