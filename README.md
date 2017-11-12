Improving LoRaWan
---

> 2017 SNU Computer Network Term Project

## Project Milestones

### Progress Report 1

- due: 11/13

#### Todo

- Study LoRaWAN Specification and Source Code
  * Class A, Transmission parameters, OTAA join procedure
- Project plan & LoRaWAN specification

### Progress Report 2 - Presentation

- due: 11/22

#### Todo

- Install and run LoRa end node, gateway, network server
- Design beacon based bi-directional communications

### Progress Report 3

- due: 12/06

#### Todo

- Project status & source code

### Final Report & Demo

- due: 12/18

#### Todo

- report
  * Detailed Instruction of implementation
  * Performance evaluation
- demo


----

## Goal

### Design and implement bi-directional communications
- End-nodes can initiate upstream communication as needed
- Enables downstream communications based on duty-cycles

&rarr;Similar to WiFi PSM mechanism

### Beacon

- Gateway and end-nodes agree on beacon intervals
- An end-node seeks beacon and wakes up every beacon interval
- A gateway transmits beacons periodically to alert end-nodes with pending downstream frames
- An end node with pending frames listens the medium until it receives frames or to the next beacon
  * Other nodes enter into sleep mode until the next beacon

## Specification
