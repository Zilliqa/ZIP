| ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 13  | P2PSeed Transport Re-architecture | Draft | Standards Track  | Chetan Phirke <chetan@zilliqa.com> | 2020-12-10 | 2020-12-10


## Abstract
The zip introduces re-architecture of transport for seed to seed socket communication.With the implementation of the zip, exchanges no longer need to open `33133` port on their end to receive response for the request.This would address security issue on exchange side to open 33133 port and also enhance our transport system.

## Motivation
Exchange seed nodes always pull data from pubseed nodes and send transactions to pubseed nodes on 33133 port.In the current  p2p comm architecture, the node operates in server-server mode.The message is unidirectional and socket communication is terminated immediately after sending request from requestor.For receiving response, the requestee node again open one tcp connection to 333133 of the requestor, write message on socket and immediately terminates the socket connection.This architecture is mainly designed keeping in miners in the mind as they have to communicate with hundreds of nodes.

In perspective of exchange nodes ,it's a security issue as they have to open incoming port on 33133 to receive response.They want us to address this issue.

### Note
From here onward, i will be referring to exchange seed as `seed` which is referred as client and Zilliqa public seed as `source seed` which is referred as server.In communications perspective, seed node will be the client and source seed node will behave as server.


## Specification:

The zip proposes reusing the existing transport mechanism and leveraging on top of it.Currently we use `libevent` on server side for handling network events for eg read/write/error.On client side, we use unix system calls(connect, write) for connecting and writing data on socket.Libevent has good support for buffer events where application no longer deals with underlying kernel read and write buffers.Libevent mangages the reading and writing on kernel buffers.

### Current transport architecture

![image01](../assets/zip-13/Existing_Transport_Architecture.png)


### Proposed transport architecture(Applicable only for seed to seed communication)

Existing P2P communication will be same for all other types nodes except for seed to seed messaging.With that in mind, the pub seed will open another listening port `33135` on their end additional to 33133.Any data received on 33135 will be handled in another callbacks(AccepCb/ReadCb/Event)

Inorder to also distinguish message from exchange nodes at application level, new message start type is introduced for seed to seed communication.
START_BYTE_SEED_TO_SEED_REQUEST
START_BYTE_SEED_TO_SEED_RESPONSE

#### Note
All socket communcations will be intiated and terminated by client only.Server node will act on events received from client.

In terms of changes, following nodes will be affected.

#### Seed Nodes(client)
Following are the implementation considerations:
1) For sending messsage to source seed , seed node will use startbyte `START_BYTE_SEED_TO_SEED_REQUEST` during `SendMessage` function call.
2) In addition there will be a time-out for response which will be initialized to `SEED_SYNC_LARGE_PULL_INTERVAL`.This is required as it is expected that for some message, there will be no response.In order to cleanup the bufferevent, we have to set timeout for messages.
3) Transport thread is running the event loop and is reasponsible for sending messages.


#### Source Seed Nodes(server)
Following are the implementation considerations:
1) Server node will open another port on 33135.All messages will startByteType  `START_BYTE_SEED_TO_SEED_REQUEST` will be handled and rest all are dropped.
2) Server node will store the bufferevent for the request in `bufferEventMap`.The key of the map would be key = `IP:PORT`, value = `bufferevent*`
3) For sending the response server node will use `START_BYTE_SEED_TO_SEED_RESPONSE` start byte in `SendMessage` function call.



Following sequence diagrams depicts the interaction between client and server in different messaging scenarios.

#### Successful Bidirectional Message

![image02](../assets/zip-13/P2PSeedCommunication_Scenario-1_Successful.png)



#### Error During Socket Connect



![image02](../assets/zip-13/P2PSeedCommunication_Error-Scenario.png)



#### No response from server



![image02](../assets/zip-13/P2PSeedCommunication_No_Response_From_Server_Scenario.png)



## Rationale
TODO

## Backward Compatibility
TODO

## Copyright Waiver 

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
