| ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 13  | P2PSeed Transport Re-architecture | Draft | Standards Track  | Chetan Phirke <chetan@zilliqa.com> | 2020-12-10 | 2020-12-10


## Abstract
The zip introduces re-architecture of transport for seed to seed socket communication.With the implementation of the zip, exchanges no longer need to open `33133` port on their end to receive response and request response model would follow like client server just in the case of HTTP.Exchange nodes will send request and receive response on the same socket just like  in the case of HTTP.


## Motivation:
Exchange seed nodes always pull data from pubseed nodes and send transactions to pubseed nodes on 33133 port.In the current  p2p comm architecture, the node operates in server-server mode.The message is unidirectional and socket communication is terminated immediately after sending request from requestor.For receiving response, the requestee nodes connects to 333133 of the requestor, send response data and immediately terminates the socket connection.This was mainly designed keeping in miners in the mind.

In perspective of exchange nodes ,it's a security issue as they have to open outgoing port on 33133 to receive response.They want us to address this issue.


## Specification:

The zip proposes reusing the existing transport mechanism and leveraging on top of it.Currently we use `libevent` for handling network events for eg read/write/error.Libevent has good support for buffer events where application no longer deals with underlying kernel read and write buffers.Libevent mangages the reading and writing on kernel buffers.

Existing architecture png.


Proposed architecture png


Since it’s big transport revamp and we do not want things to break in the existing architecture of internal nodes communication, we tried to separate out seed to seed communication and the callbacks part as well.With that in mind, the pub seed will open another listening port `33135` on their end additional to 33133.Any data received on 33135 will be handled in another callbacks(AccepCb/ReadCb/WriteCb)

Inorder to also distinguish message from exchange nodes, new message start type is introduced for seed to seed communication.
START_BYTE_SEED_TO_SEED_REQUEST
START_BYTE_SEED_TO_SEED_RESPONSE


## Rationale
TODO

## Backward Compatibility
TODO

## Copyright Waiver 

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
