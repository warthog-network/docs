# Warthog WebRTC communication draft
We propose a protocol upgrade that supports establishing WebRTC connections between nodes. It is therefore necessary for nodes to honor the version of peers to know which protocol version to use. This is necessary to ensure compatibility of new version with current version nodes. A we call a peer that understands this new protocol a v2 peer. Peers that only understand the old protocol are v1 peers.


## WebRTC needs to be signaled
When RTC node A wants to connect to another RTC node C, SDP (session description protocol) data needs to be exchanged over a signaling server. In Warthog, each node can act as a signaling server and can assist in establishing direct P2P connection between two of its peers. We denote this node by *B*. Node B must be already connected with node A and C.
```
                 SDP Offer             SDP Offer
        Node A ------------> Node B -------------> Node C
                                                        | creates
                 SDP Answer            SDP Answer       | Answer
        Node A <------------ Node B <------------- Node C
```
After that a direct WebRTC connection between nodes A and C can be established.

In this setting we say node A initiated an outgoing WebRTC connection and node C accepted an incoming WebRTC connection. Node B signaled the WebRTC connection.

## Node dictionary
RTC nodes periodically inform each peer about how many incoming WebRTC connections are accepted to be signaled via that peer. This number can be different for each peer.


## Connection life cycle


## P2P Messages
### Overview
The following message types are used by nodes to allow WebRTC peer discovery and signaling:


| Type | Meaning |
| ---- | ------- |
|`RTCIdentity`| Notify the node about STUN server result     |
| `RTCQuota` | Notify allowed RTC quota |
| `RTCSignalingList` | Per peer signaling offer list ("mix all peers' quotas" and annotate with IP)
| `RTCRequestForwardOffer` | Ask for forwarding of SDP offer to some peer from `RTCSignalingList`
| `RTCForwardedOffer` | Forward SDP offer to intended peer, subtracts from that peer's quota.
| `RTCRequestForwardAnswer` | Ask for forwarding of SDP answer.
| `RTCForwardedAnswer` | Forward SDP answer.

In the above setting, the message transmission scheme looks like this:


```
             RTCInfo                 RTCQuota
Node A <---------------- Node B <----------------- Node C

     RTCRequestForwardOffer      RTCForwardedOffer
Node A ----------------> Node B -----------------> Node C
                                                        | creates
       RTCForwardedAnswer     RTCRequestForwardAnswer   | Answer
Node A <---------------- Node B <----------------- Node C
```

### Message structure

#### `RTCIdentity` message
##### Description
RTC nodes can send a `RTCIdentity` message once. It contains results of a STUN server request to pass own IP to other nodes.
##### Rate limit
This message can only be sent once.

#### `RTCQuota` message
##### Description
Every RTC node sends quota messages to each v2 peer. The quota refers the number of `RTCForwardedAnswer` messages it accepts from each peer, i.e. to the number of "creates Answer" actions in the above protocol that each peer can trigger.
##### Rate limit
This message can only be sent at specific frequency.
##### TODO: 
We still need to implement a mechanism to increase the quota when some messages are consumed.

#### `RTCSignalingList` message
##### Description
Nodes send this message out to all V2 peers. It provides a list of RTC connections they are willing to help establish as signaling server. Nodes create this list by respecting and mixing RTC peers' quotas. 

For example assume our node has a quota of 
* 1 allowed `RTCForwardedOffer` from peer A and 
* 10 from peer B.

It might then send out a `RTCSignalingList` messages to peers C,D, E:

##### List C
* Peer A
* Peer B

##### List D
* Peer B

##### List E
* Peer B

Note that above, peer A was only offered one time because the total quota is only 1 while peer B was offered 3 times which respects the limit 10. Each peer C,D,E can select peer addresses from the list they wish to connect to using our node as signaling server by sending a `RTCRequestForwardOffer` message to our node and our node will contact the selected nodes by issuing `RTCForwardedOffer` to the intended node in order to let it create an SDP answer. It is essential that our node only offers those peers in the `RTCSignalingList` messages such that the total sum of offered connections respects each peers quota of accepted `RTCForwardedOffer` messages.

##### Rate limit
This message can only be sent at specific frequency.




#### `RTCRequestForwardOffer` message
##### Description
From peers announced in a `RTCSignalingList` message nodes select peers they wish to connect to and ask the sender of the `RTCSignalingList` message to act as signaling server by sending a generated SDP offer in a `RTCRequestForwardOffer` message. The receiver of that message will forward that SDP offer to the offered peer via a `RTCForwardedOffer` message, see below.

##### Rate limit
The node acting as signaling server keeps track of options offered in the `RTCSignalingList`` message and disconnects nodes sending duplicate `RTCRequestForwardOffer` messages or when they are for nodes that were not offered before.

#### `RTCForwardedOffer` message
##### Description
This message forwards the SDP offer received in a `RTCRequestForwardOffer` message to the intended destination, destination is specified in the received `RTCRequestForwardOffer` by referring an offer entry index from the previously sent `RTCSignalingList` message. Therefore nodes keep track on previously issued `RTCSignalingList` messages.
##### Rate limit
The receiver of `RTCForwardedOffer` messages counts and compares the number of these messages for each peer with the quota previusly assigned via a `RTCQuota` message. 
#### `RTCRequestForwardAnswer` message
##### Description
When a node receives a `RTCForwardedOffer`, it extracts the contained SDP offer and uses WebRTC library functions to create an SDP answer. When that answer is transmitted to and used by the original node that created the SDP offer, the direct connection over WebRTC can be established. Since there is no direct connection yet, the message needs to be forwarded again by the signaling node that sent the `RTCForwardedOffer`. The `RTCRequestForwardAnswer` message contains the SDP answer is sent to the signaling node to be forwarded to the original creator of the SDP offer.

##### Rate limit
For each sent `RTCForwardedOffer` one `RTCRequestForwardAnswer` message is expected. When more such messages are received, the connection to the sender is closed.

#### `RTCForwardedAnswer` message
##### Description
A `RTCRequestForwardAnswer` message contains an SDP answer and when a noce receives such a message, it looks up the original sender of the corresponding `RTCRequestForwardOffer` message (origin of the SDP offer) and forwards to that node. To achieve this, nodes must save and match the following information:
* For each sent `RTCForwardedOffer`, in response to a received `RTCRequestForwardOffer` message from some node A it must save the origin A.
* For each received `RTCRequestForwardAnswer` message it must match the corresponding `RTCForwardedOffer` previously sent. This can be realized using a queue because we expect the peer to create SDP answers in order.

This information is sufficient for the node to complete the signaling server task and forward the SDP answer back to the original node `A` in using a `RTCForwardedAnswer` message.

The receiving node can then call WebRTC API functions to assign the SDP answer to the matching pending connection to trigger WebRTC connection attempts.

##### Rate limit
The receiver of a `RTCForwardAnswer` expects at most one such message for each sent `RTCRequestForwardOffer` and disconnects nodes that send unrequested SDP answers in such messages.
