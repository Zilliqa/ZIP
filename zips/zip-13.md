| ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 13  | Transport Re-architecture for Pull Messages Between Seed Nodes | Draft | Standards Track  | Chetan Phirke <chetan@zilliqa.com> | 2020-12-10 | 2020-12-10

## Abstract

This ZIP introduces the re-architecture of the transport layer for seed-to-seed socket communication, specifically for pull messages (as required during data syncing). With the implementation of this ZIP, seed node operators no longer need to open port `33133` to incoming connections on their end to receive responses for their requests. This would address security requirements on parties (such as exchanges) that may not be able to open port `33133` for incoming connections. This would also enhance Zilliqa's transport system by reducing the amount of TCP connections.

## Motivation

In the Zilliqa protocol, seed nodes follow a three-level hierarchy:

* Zilliqa Research-operated **lookup** nodes sit closest to the mining network and directly communicate with it (e.g., for dispatching transactions)
* Zilliqa Research-operated public seed nodes provide blockchain data to the entire community (e.g., through the official API and through port `33133` requests) and relay transaction requests to lookup nodes
* Community-operated seed nodes provide blockchain data to the node host (e.g., crypto exchange) and relay transaction requests to lookup nodes

In order to remain synced, community-operated seed nodes regularly **pull** data from Zilliqa Research-operated public seed nodes. In the current P2P communication architecture, however, **all nodes operate in server-server mode**. The message is unidirectional and socket communication is terminated immediately after sending the request. Thus, in order to send the response, the requestee node opens a second TCP connection on `333133` in the requestor, writes the message on the socket, and immediately terminates the socket connection after sending the response. In effect, bidirectional messaging for these seeds (such as in the case of pull data requests) is reduced to two unidirectional messages.

This communication architecture was mainly designed with miner nodes in mind, as they have to communicate concurrently with hundreds of nodes in the network. In the perspective of seed nodes, certain seed node operators may have a policy that disallows the opening of any listening port to the public domain. As such, running a seed node has not been viable for these operators right now. What is needed is a way to support bidirectional seed-to-seed communication that removes the port opening requirement.

>**Note:** From hereon, when discussing seed-to-seed communication, a community-operated seed node will be simply referred to as `seed` (which acts as client) and a Zilliqa Research-operated public seed will be referred to as `source seed` (which acts as server).

## Specification

This ZIP proposes reusing the existing transport mechanism and leveraging on top of it. Currently, we use `libevent` on the server side for handling network events, e.g., read/write/error. On the client side, we use UNIX system calls (`connect()` and `write()`) for connecting and writing data on the socket. `Libevent` has good support for buffer events where the application no longer deals with the underlying kernel read and write buffers. `Libevent` manages the reading and writing on kernel buffers.

### Current Transport Architecture

![image01](../assets/zip-13/P2PSeedComm_Existing_Transport_Architecture.png)

### Proposed Transport Architecture (Applicable Only for Bidirectional Seed-to-Seed Communication)

In the proposed re-architecture, the existing P2P layer will be the same for all other types of inter-node communication except for bidirectional seed-to-seed messaging. In that particular scenario, the `source seed` will open another listening port (such as `33135`) on its end to cater only to bidirectional seed-to-seed messages. Any data received on port `33135` will be handled in another set of callback methods (accept/read/event).

In order to distinguish these messages at the application layer, the transport layer must introduce the two start byte values below for request and response messages:

* `START_BYTE_SEED_TO_SEED_REQUEST`
* `START_BYTE_SEED_TO_SEED_RESPONSE`

> **Note:** All socket communications will be initiated and terminated by the client (`seed`) only. Server node (`source seed`) will act on events received from the client.

### Implementation Considerations

In terms of changes, the following nodes will be affected.

#### Seed Node (Client)

1. Seed node will always connect to `33135` for pull (bidirectional) messages. Port `33133` will continue to handle unidirectional messages (like forwarding transactions).
2. Seed node key should be whitelisted in our lookups and seedpubs.Request from non whitelisted seed nodes will cause connection termination from source seed server.
3. For sending a message to source seed for pull messages, seed node will use start byte `START_BYTE_SEED_TO_SEED_REQUEST` during `SendMessage()` function call.
4. In addition, there will be a timeout for accepting response message, which will be initialized to `SEED_SYNC_LARGE_PULL_INTERVAL` during `bufferevent` creation through API `bufferevent_set_timeouts`. This timeout is required as it is expected that for some messages there may be no response. In order to clean up the `bufferevent` at the sender side, we have to also set a timeout for messages or wait for timer expiration at the application level, and then do the cleanup.
5. In order to keep the event loop running during creation, it is required to add a timer event and the call `event_base_dispatch()`. If this is not done, the event loop terminates immediately and kills the main thread after creation.
6. The main thread runs the event loop and is responsible for handling network events.

#### Source Seed Node (Server)

1. Server node will open another port on `33135` and the events for it will be dispatched to the same event loop handling listening events on port `33133`.
2. If the client key is not whitelisted in the server, the connection will be immediately closed by the server for the request.
3. Besides, `MAX_PEER_CONNECTION` is populated with 100 connections.At maximum of 100 simultaneous connections can be there from a particular IP.
4. All messages with start byte `START_BYTE_SEED_TO_SEED_REQUEST` will be handled on new accept/read/event callbacks, and the rest are all dropped.
5. Server node will store the `bufferevent` for the request in `bufferEventMap`, which stores `key=IP:PORT` and `value=bufferevent*`.
6. For sending the response, server node will use `START_BYTE_SEED_TO_SEED_RESPONSE` start byte in `SendMessage()` function call.
7. In case server node does not send any response for incoming request, the server node will just remove the `bufferevent` from `bufferEventMap`. Termination of socket connection will only be done by events from client side.
8. `Bufferevent` cleanup will only be done by events from client side (i.e., socket closure and socket error).

The following sequence diagrams depict the interaction between client and server in different messaging scenarios.

#### Scenario 1: Successful Bidirectional Message

![image02](../assets/zip-13/P2PSeedComm_Successful_Scenario.png)

#### Scenario 2: Errored Socket Events

![image03](../assets/zip-13/P2PSeedComm_Error_Scenario.png)

#### Scenario 3: No Response From Server

![image04](../assets/zip-13/P2PSeedComm_No_Response_From_Server_Scenario.png)

## Rationale

The rearchitecture proposed in this ZIP addresses the infrastructure restriction for some seed node operators where they are unable to listen on incoming port `33133`. It also improves the Zilliqa core network layer architecture by reducing the number of TCP connections per message.

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
