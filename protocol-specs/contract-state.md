# Contract State

The M3tering protocol uses the Arweave network to permanently log and archive all interaction data on the protocol, ensuring that a record of energy consumption and associated payments remain available and immutable for the foreseeable future. A novel smart contract standard know as SmartWeave, is used to compute the state of PPA contracts on the protocol.&#x20;

{% hint style="info" %}
SmartWeave contracts on Arweave are a type of smart contract specifically designed for use on the Arweave blockchain. Rather than executing code in real time (like on Ethereum), SmartWeave contracts rely on nodes to read the entire history of a contract’s interactions to compute its current state, an idea know as "Lazy Evaluation". This approach makes SmartWeave contracts scalable and ideal for decentralized, permanent storage applications, as it reduces computational demand on the network while prioritizing data availability and integrity. See more [here](https://dev.to/fllstck/smart-contracts-on-arweave-46l8)
{% endhint %}

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

While payments still happen on a secure, deterministic EVM contract using DAI, SmartWeave contracts are be used to process metering data and manage interactions between users and the smart metering infrastructure.

## Smartweave Contract Functions

The protocol defines three core functions within its Smartweave contracts to handle different types of interactions: `register`, `topup`, and `meter`.

### **The Register Function**

The `register` function is used to register a new smart meter within the M3ter protocol. It expects a JSON object with an EVM transaction hash, which uniquely identifies the registration transaction and allows computation nodes to retrieve the registered public key for authenticating any data received from the smart meter.&#x20;

Example input:

```json
{
  "input": {
    "function": "register",
    "txHash": "0x5cfeb3b6e68c9cbdd96e6f7db033900514009db660adc3c81d98495e6600506b"
  }
}
```

### **The Topup Function**

&#x20;The `topup` function allows registered participants to add balance or credits. Like `register`, this function expects an Ethereum transaction hash to verify the top-up action. This transaction hash is used to retrieve the paid amount and the current electricity tariff set on the EVM contract. The offtakers electricity balance is then incriminated accordingly. &#x20;

Example input:

```json
{
  "input": {
    "function": "topup",
    "txHash": "0x5cfeb3b6e68c9cbdd96e6f7db033900514009db660adc3c81d98495e6600506b"
  }
}
```

### **The Meter Function**

The `meter` function is the primary function that processes data collected from smart metering hardware. It expects the payload from the meter, containing the signed data, signature, and public key. This function would compute the offtaker electricity balances and deduct usage based on real-time electricity consumption data payload posted to Arweave.

Example input:

```json
{
  "input": {
    "function": "meter",
    "payload": [
      "[2,213.7,0.38,0.007420]",
      "9C7lPdznR9pymAIvjDPmm/mVX/uUTemapJRb8yzGKvG8or43u6V97oDPcW7ZP9HeHRZrGEf1iIkyLixAVdWsDg==",
      "JR2VNczJacY86eyyCBr1iTTT7vxKtYbfqegeTkXJD88="
    ]
  }
}
```
