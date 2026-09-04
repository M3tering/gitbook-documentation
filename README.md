---
description: What is it and why?
icon: circle-question
cover: >-
  .gitbook/assets/Screenshot 2023-12-18 at 05-59-33 (351) Dear Alice -
  YouTube.png
coverY: 0
---

# What is the M3tering Protocol?&#x20;

Conventional energy infrastructure is usually built from the center outward. A large institution finances generation, extends transmission and distribution, installs meters, and enrolls people as customers. This model has built reliable systems in many places. But in several others, the cost of sustaining centralized infrastructure has exceeded the capacity of institutions and customers expansion is too slow, communities face chronic outages, and weak accountability. People may wait decades for dependable service. Distributed energy makes another direction possible

### Reimagining The Grid

Every part (or participant) in an energy network can be understood as a system with a boundary. Energy crosses that boundary as an input or an output. The participating equipment may be radically different, and may play several roles over the course of a day. But it all boils down to a simpler fact: _**An identified system exchanged a measured quantity of energy with the infrastructure around it.**_&#x20;

{% hint style="info" icon="thought-bubble" %}
A home imports electricity from a network and exports electricity from its solar panels. A battery receives energy when it charges and supplies energy when it discharges. A transmission line accepts energy at one point and delivers it at another. An electric vehicle, generator, factory, appliance, microgrid, or entire city can be viewed through the same basic model.
{% endhint %}

The M3tering protocol provides a common way to count these exchanges, authenticate their source, and incorporate them into a shared record. From that simple foundation, much more complex energy systems can be built. It gives various participants in the energy supply chain a common way to establish identity, report activity, and participate in programmable agreements. It turns measurements from real-world energy resources into shared, verifiable records that software can act on. This allows people, devices, applications, and organizations to coordinate around energy and build their own energy networks

A participant may be compensated by their connected peers for producing or transmitting electricity with their assets. Other participants compensate their peers for electricity they've consumed or stored. Across the entire energy supply chain, each participant's meter operates a bilateral contract with each of it's connected peers.&#x20;

Functioning energy networks can begin with a few connected participants, meters, and their contracts. A solar installation supplies several homes. A shop adds a generator. A clinic adds storage that receives priority during an outage. New participants connect. Additional owners finance new capacity. Eventually, one local network coordinates with another or with the wider grid. At every stage, the shared ledger answers the questions: who contributed energy, who received it, how the network's state changed, and which agreements should respond.

This is especially powerful in places that have never had reliable energy infrastructure. They can bypass legacy centralized institutions, begin locally, expand incrementally with more participant, and connect outward.

> The M3tering Protocol is an open state and communication protocol for energy systems: it turns authenticated energy inputs and outputs into shared state that smart contracts can interpret and participating systems can act upon

One measurement would only describe an event. A sequence of measurements describes how a resource's energy position changes over time. Measurements from many resources describe the evolving state of a network. The protocol is how we maintain a stateful ledger of those counted energy flows. It is state that programs can use. A smart contract can read an authenticated change in energy state and apply rules agreed beforehand. Depending on the application, it might:

* compensate a resource for energy supplied; or establish that an energy-service obligation was met;
* release revenue after an asset demonstrates performance; or the divide revenue among the owners of an asset;
* deduct consumed electricity from a prepaid balance; or reward a participant for reducing demand at a critical time;

The ambition is to provide a minimal, open primitive from which different energy infrastructures can be assembled: locally, incrementally, and according to the needs of the people who depend on them.

{% content-ref url="developer-docs/v2.0/" %}
[v2.0](developer-docs/v2.0/)
{% endcontent-ref %}

