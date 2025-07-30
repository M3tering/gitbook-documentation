---
description: A Verifiable Computation Layer for Meter Data
cover: >-
  ../../.gitbook/assets/bafybeig7vgexhnre4doaqowyrhppq4mxrmy5h2kteaur73jakskhx7gcaq.png
coverY: 0
---

# Rollup

The M3tering Protocol introduces a specialized, high-throughput system for aggregating vast amounts of time-series data from smart meters and making it verifiably available to on-chain applications. At its core, the system functions as an **Application-Specific** [**Rollup**](https://ethereum.org/en/developers/docs/scaling/zk-rollups/), leveraging the power of zero-knowledge proofs to execute complex computations off-chain while inheriting the full security and data availability guarantees of the Ethereum mainnet.&#x20;

The primary challenge this system addresses is the impracticality of processing high-frequency Internet of Things (IoT) data directly on a Layer 1 blockchain. A smart meter might report energy consumption every minute, and a network of thousands of such meters would generate an overwhelming volume of transactions, leading to prohibitive gas costs and network congestion. Our solution is to aggregate this data off-chain and then periodically commit a succinct, cryptographically secured proof of the new global state to Ethereum. This allows smart contracts—for applications like prepaid energy billing, tokenized carbon credits, or renewable energy certificates—to access trustworthy, up-to-date energy consumption data without bearing the cost of processing every individual data point.

## System Architecture

The protocol is logically divided into two main domains: the on-chain contracts that provide trust and data anchoring on Ethereum, and the off-chain infrastructure that performs the heavy computational lifting.

#### **M3ter NFTs (ERC721)**

Introduced on page[m3ter-nfts.md](../../token-economics/m3ter-nfts.md "mention"), each smart meter is represented on-chain as a unique Non-Fungible Token (NFT). The metadata for each NFT crucially contains the meter's public key, which is used to verify the authenticity of its data transmissions. This on-chain registry of public keys serves as the root of trust for all off-chain signature verifications.

#### **State Blobs (SSTORE2 Contracts)**

The state blobs are simple, contiguous blocks of data. Each meter is allocated a 6-byte slice within the blob, indexed by its `tokenId`. For a meter with `tokenId = i`, its nonce can be read from `bytes 6*i` to `6*(i+1)-1` of the nonce blob. The aggregated state of all meters is stored on-chain using the [SSTORE2](https://www.google.com/search?q=https://github.com/solidstate-network/solidstate-solidity/blob/master/contracts/utils/SSTORE2.sol) pattern. This technique stores data in the bytecode of a deployed contract, offering significant gas savings for writing and reading large, static data blobs compared to traditional `SSTORE` operations. This structure is highly efficient for on-chain parsing but imposes a hard limit on the system's capacity. Due to the Ethereum contract code size limit of 24kb ([EIP-170](https://eip.tools/170)), the maximum indexable byte is `0x6000`. This translates to a maximum `tokenId` of `0x1000`, limiting the system to 4096 meters per rollup instance. Two separate contracts are maintained for each state update: one for meter **nonces** (to prevent replay attacks) and one for their **cumulative energy sums**.

#### **Verifier Gateway**

A smart contract that implements the verifier for the specific zero-knowledge proof system being used. In this architecture, it is an [SP1](https://www.google.com/search?q=https://succinct.xyz/blog/sp1) [Groth16](https://www.google.com/search?q=https://electriccoin.co/blog/snark-explain-part-5/) Verifier Gateway, which is responsible for checking the validity of the proofs submitted by the off-chain prover.

#### **SP1 Prover Program**

A Rust program built using the SP1 toolchain, which allows for the creation of provable programs. This program contains the core logic for validating meter transactions, aggregating their data, and generating a Groth16 zk-SNARK proof of the entire computation.

#### **Peer-to-Peer Network**

Nodes running the off-chain client software form a p2p network using protocols like [Waku](https://waku.org/) or [Streamr](https://streamr.network/). This network is used to gossip and share pending meter data transactions before they are aggregated into a proof.

## The Off-Chain Proving Cycle: From Data to Proof

The heart of the system is the recurring cycle of off-chain computation performed by the prover. This process is designed to be entirely stateless, meaning the prover does not need to maintain any historical data. It relies solely on Ethereum as its source of truth for the previous state and its destination for the newly computed state.

A prover begins its work by gathering a set of inputs. These inputs are the foundation upon which the new state will be verifiably built. They include the last known state blobs for both nonces and energy, a recent Ethereum blockhash to act as a trust anchor, a collection of unprocessed meter data transactions, and cryptographic proofs of the meters' public keys.

The prover fetches the bytecode of the latest nonce and energy `SSTORE2` contracts directly from Ethereum. These blobs represent the starting point for the new computation. The prover then selects a recent block from the Ethereum chain, typically one within the last 256 blocks, and retrieves its blockhash. This blockhash is critical, as it corresponds to a specific, immutable state of the Ethereum world computer. It is against this state that all public key validations will be performed.

With the state anchor established, the prover uses the `eth_getProof` JSON-RPC method. This powerful function allows the prover to request a [Merkle-Patricia Trie (MPT)](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/) proof for a specific piece of data within Ethereum's state. For each meter with pending transaction it intends to process, the prover fetches an MPT proof that demonstrates the value of the meter's public key as stored in the m3ter NFT contract at the checkpoint block. This mechanism ensures the prover is using the authentic, on-chain registered public keys for signature verification.

Diagram: Public Key Validation via Merkle-Patricia Trie Proof

This diagram illustrates how eth\_getProof provides a verifiable link from a known blockhash down to a specific storage slot containing a meter's public key.

```
+--------------------------+
| Checkpoint Block         |
| (Block Number: N)        |
| Blockhash: 0xabc...      |
+-------------|------------+
              |
              v
+-------------|------------+
| State Root Hash          |  <-- Contained in the block header
+-------------|------------+
              | (Path determined by m3ter contract address)
              v
+-------------|------------+
| Account Trie Node        |
| (for m3ter NFT Contract) |
+-------------|------------+
              |
              v
+-------------|------------+
| Storage Root Hash        |  <-- Root of the contract's storage
+-------------|------------+
              | (Path determined by storage slot of the meter's tokenId)
              v
+-------------|------------+
| Storage Slot Value       |
| Public Key: 0x123...     |  <-- The proven public key
+--------------------------+

```

Once all inputs are gathered, the SP1 program executes its core logic. It iterates through the list of unseen data transactions. For each transaction, it performs two critical checks. First, it verifies the [ED25519](https://www.google.com/search?q=https://en.wikipedia.org/wiki/EdDSA%23Ed25519) signature of the transaction using the public key whose authenticity was just confirmed via the MPT proof. Second, it checks the transaction's nonce, ensuring it is exactly one greater than the last known nonce for that meter from the input blob. This sequential nonce prevents replay attacks and guarantees that each transaction is processed exactly once.

If a transaction passes both validations, its energy consumption value is added to the meter's cumulative total, and its nonce is incremented. After processing all valid transactions in the batch, the prover constructs the `new_nonce_blob` and `new_energy_blob`. These are formed by concatenating the updated 6-byte values for each meter, indexed by their `tokenId`.

Finally, the SP1 program generates a Groth16 zk-SNARK proof of this entire computation. The proof cryptographically attests that the new state blobs were correctly derived from the initial state blobs according to the rules of the protocol. The program's output consists of the proof itself and a set of public outputs that the proof commits to. These public outputs are the keccak256 hashes of the initial and new state blobs, along with the checkpoint blockhash, which are essential for on-chain verification.

## On-Chain State Update and Verification

With the proof and new state blobs in hand, the prover submits a transaction to the M3tering rollup contract on Ethereum. This transaction includes the proof, committed state blobs, and the anchor block number.

The rollup contract first uses the `BLOCKHASH` opcode to retrieve the blockhash for the specified anchor block number. It then passes this retrieved blockhash, along with the other public outputs and the proof, to the SP1 Groth16 verifier. If the verifier confirms the proof's validity, the rollup contract proceeds to deploy the new nonce and energy blobs.

The deployment uses `CREATE2` for deterministic addressing. The unique salt for the deployment is the `chain length`—the sequential number of this state update since the genesis. This elegant mechanism ensures that the address of each historical state blob can be easily recomputed, allowing applications to "time-travel" and query the system's state at any point in its history simply by knowing the update number.

The raw transaction data processed in the batch is also posted to Ethereum's Proto-Danksharding blob sidecar for cost-effective, short-term data availability, and simultaneously archived to long-term storage solutions like Arweave or Filecoin for permanent verifiability.

Diagram: The Stateless Prover Loop

This diagram shows how the prover relies entirely on Ethereum for state, enabling a robust and decentralized network of participants.

```
      +-------------------------------------------------+
      |                                                 |
      |             Ethereum Blockchain (L1)            |
      |                                                 |
      |  +-----------------+   +----------------------+ |
      |  | SSTORE2         |   | Verifier & Deployer  | |
      |  | (State Blobs)   |   | Contract             | |
      |  +-----------------+   +----------------------+ |
      |                                                 |
      +-------^-----------------|-----------^-----------+
              | 1. Read Last    |           | 4. Write New State
              |    State Blobs  |           |    & Proof
              |                 |           |
+-------------+-----------------v-----------+-------------+
|                                                         |
|                  Off-Chain Prover Node                  |
|                   (Stateless Client)                    |
|                                                         |
|  2. Get Pending Txs   +-----------------------------+   |
|     from P2P Network  |                             |   |
|  -------------------> |   SP1 Proving Environment   |   |
|                       |                             |   |
|  3. Compute New State |  - Verify Signatures (MPT)  |   |
|     & Generate Proof  |  - Verify Nonces            |   |
|  <------------------- |  - Aggregate Energy         |   |
|                       |  - Generate Groth16 Proof   |   |
|                       +-----------------------------+   |
|                                                         |
+---------------------------------------------------------+


```
