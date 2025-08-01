---
cover: ../../.gitbook/assets/monzov2-OG.png
coverY: -67.4265339626058
---

# V2.0 Specs

The M3tering 2.0 (“V2”) represents the next evolution of decentralized energy metering and market infrastructure. Building on the learnings of the genesis version and the foundational principles of secure, verifiable metering for peer-to-peer energy service markets, V2 introduces a suite of enhancements designed to meet the demands of broader protocol adoption and cross-chain deployments. It introduces a redesigned meter payload format, a trustless state evaluation engine powered by zk-rollup, and enhanced cross‑chain interoperability.

This documentation is intended for protocol integrators, application developers, infrastructure operators, and energy service providers. It outlines the core architectural changes in V2, details new on‑chain and off‑chain components, and provides guidance for deploying and interacting with the M3tering protocol.

By integrating seamlessly with Ethereum Mainnet and a broad spectrum of Layer 2 environments—such as Base, Arbitrum, Optimism, Celo, and UniChain—M3tering 2.0 delivers a unified framework for decentralized energy applications.

## **Key Features**

&#x20;**M3ter Rollup, Ethereum Mainnet and L2 Integration** \
In V2, all meter state is now committed to Ethereum using succinct zero knowledge proofs, which ensures that applications and smart contracts across Ethereum L1 and L2s can access a single, canonical source of truth. Through trustless interoperability primitives like [CCIP Read (ERC-3668)](https://eip.tools/eip/3668), [L1SLOAD (RIP-7728)](https://eip.tools/rip/7728), and [Crosschain Tokens (ERC-7802)](https://eip.tools/eip/7802), developers can build cross-chain more powerful apps that interoperate across layer-1 and layer-2 environments—unlocking use cases that were previously infeasible onchain. By coming home to Ethereum Mainnet for ultimate security, and integrating  and a broad spectrum of Layer 2 environments—such as Base, Arbitrum, Optimism, Celo, and UniChain—V2 delivers a unified framework for decentralized energy applications.

**Compact and Extensible Meter Payloads**\
A redesigned payload format uses tightly packed byte arrays—4 bytes for nonce, 4 bytes for energy reading, and 64 bytes for signature—with optional extension slots (voltage, device ID, geolocation) to future-proof data capture without bloating transaction size.

**Modular Client Node: P2P Gossip Networking and Data Publishing**\
The M3tering client node "Console" is designed as a modular and extensible component, enabling direct participation in the network without relying on intermediaries. Each participant can run a lightweight client node—no centralized relayer needed. By enabling p2p operation and diversity of implementation, V2 strengthens the protocol’s neutrality, reliability, and developer adoption across ecosystems. The modularity of the node software also supports optional plug-ins for&#x20;

* data propagation via gossip and pub/sub protocols (eg Libp2p, Waku or Streamr)
* custom archiving strategies (e.g. Ethstorage, Ethswarm, Filcoin/IPFS, Arweave or Cloud providers).&#x20;

Together, these improvements make M3tering Protocol the most versatile, and developer-friendly stack for decentralized energy metering, energy asset tokenization, and P2P energy markets. In the pages that follow, you will find detailed specifications and reference implementations to help you integrate, extend, and contribute to the M3tering ecosystem.
