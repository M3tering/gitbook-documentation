---
description: >-
  Overview of the 4 main source codes repositories implemented under the
  M3tering Protocol
cover: >-
  ../.gitbook/assets/Screenshot 2023-12-18 at 05-59-33 (351) Dear Alice -
  YouTube.png
coverY: 114
---

# Overview

{% embed url="https://github.com/orgs/M3tering/repositories" %}

## 🤖 M3ter: ERC-721 contract&#x20;

#### Deployment Address: [<mark style="color:green;">0x39fb420Bd583cCC8Afd1A1eAce2907fe300ABD02</mark>](https://gnosis.blockscout.com/token/0x39fb420Bd583cCC8Afd1A1eAce2907fe300ABD02?tab=inventory) 

M3ter is an ERC-721 (NFT) contract where each NTF represent a smart metering device registerd on-chain by it's manufacturer. Similar to Helium device NFTs on Solana, or Wicrypt device NFTs on polygon.  These devices contain cryptographic secure elements used to sign energy consumption data they measure. the purpose of `keyByTokenId` is to map an NFT's `tokenId` to the cryptographic `PublicKey` for a device (bytes 32 key for ED25519 digital signature scheme), while the `tokenIdByKey` is used to reverse map the above mentioned `PublicKey` to the NTF's `tokenId`

{% embed url="https://github.com/M3tering/M3ter" %}

## 🪙 Solaxy: ERC-20 as ERC-4626 contract

#### Deployment Address: [<mark style="color:green;">0xF4F3c1666E750E014DE65c50d0e98B1263E678B8</mark>](https://gnosis.blockscout.com/token/0xF4F3c1666E750E014DE65c50d0e98B1263E678B8?tab=holders)

Solaxy is an ERC-20 token contract that implements a linear bonding curve with DAI (a stablecoin) as the reserve currency. The bonding curve allows users to buy and sell Solaxy tokens directly from/to the contract at a dynamic price determined by the curve's slope.&#x20;

Solaxy extends its functionality by providing support for [`ERC-4626`](https://eips.ethereum.org/EIPS/eip-4626); a tokenized vault interface. This interface allows the Solaxy contract to interact with other DeFi protocols and platforms seamlessly. `ERC-4626` integration enhances the capabilities of Solaxy in the context of token bonding curves.

{% embed url="https://github.com/M3tering/Solaxy" %}

## ⚙️ Protocol: PPA pre-payment contract

#### Deployment Address: [<mark style="color:green;">0x2b3997D82C836bd33C89e20fBaEF96CA99F1B24A</mark>](https://gnosis.blockscout.com/address/0x2b3997D82C836bd33C89e20fBaEF96CA99F1B24A?tab=contact\_code)

Represents the onchain components of the Power Purchase Agreements (PPAs) between the Providers and Off-takers. The `pay` function is executed by the end users to pay DAI tokens to the owner of their assigned M3ter at the time of payment.

{% embed url="https://github.com/M3tering/Protocol" %}

## Strategies: Claim DAI revenues

Customizable contract methods for claiming revenues, used to extend the protocol contract. This allows developers to implement extensions such as DEX swaps, cross-chain messaging, create DeFi loans or to mint the new `Solaxy` tokens.

## Available Strategies

#### CLM-0

Implements a simple transfer of the DAI revenues belonging to `msg.sender` to their account&#x20;

Developer: [https://github.com/iChristwin](https://github.com/iChristwin)

{% embed url="https://github.com/M3tering/CLM-0/" %}

#### CLM-1

Implements a strategy to mint SLX tokens to `msg.sender` account using their DAI revenues as the underlying collateral

Developer: [https://github.com/iChristwin](https://github.com/iChristwin)

{% embed url="https://github.com/M3tering/CLM-1/" %}

#### CLM-2

Implements a strategy to swap the DAI revenues belonging to `msg.sender` for SLX tokens, using the Balancer DEX on Gnosis mainnet

Developer: [https://github.com/iChristwin](https://github.com/iChristwin)

{% embed url="https://github.com/M3tering/CLM-2/" %}

#### CLM-3

Off-ramping revenues from the M3tering protocol via Monerium &#x20;

{% embed url="https://github.com/M3tering/CLM-3/" %}
