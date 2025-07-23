# Overview

The M3tering Protocol specifies a framework for secure and transparent metering of electricity consumption and tracking of the associated transactions, using a combination of secure smart meter hardware and an Arweave-based smart contracts known as SmartWeave contracts.

<figure><picture><source srcset="../../.gitbook/assets/Screenshot 2024-07-06 at 10-26-26 M3tering - draw.io.png" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/M3tering.svg" alt=""></picture><figcaption></figcaption></figure>

Each smart meter cryptographically signs its data payload locally before it is broadcasted to node operators for contract execution and to the larger Arweave network for permanent storage. This provides the relevant stakeholders with a way to trustlessly verify the integrity and authenticity of activity on the M3tering protocol. The protocol ensures secure and decentralized interactions with metering data. Each function within the protocol serves a distinct purpose, contributing to a secure, transparent, and decentralized energy metering system.\
\
This following document ion sections outlines the protocol’s core specification, including the format of energy payloads, signature verification, and the interaction between M3ter data and SmartWeave contracts. This specification lays out the technical aspects of the M3tering Protocol to guide developers and integrators in implementing and interacting with this decentralized energy metering solution.
