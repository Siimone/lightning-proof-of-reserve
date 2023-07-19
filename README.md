# Lightning Proof of Reserve
Simple trust-minimized layer for a Proof of Reserve on the Lightning Network based on Digital Signatures.

## How it works
![Scheme 1](docs/img/1.png?raw=true "Scheme #1")

Let's take the following scheme of nodes.
Node `A` can ask `B` a cryptographically signed attestation of funds. More precisely, `A` can ask `B` for a cryptographically signed message where `B` states the remote balance of the channel with `A`. In this way, Lightning nodes can gather independent proofs from every node they have a channel with.

![Scheme 2](docs/img/2.png?raw=true "Scheme #2")

The scheme above shows a more complex Lightning Network topography.  
In this case, `A` will ask for _a proof of remote balance_ to `B`,`C`,`D` and `E`.
Each node is going to produce a cryptographically signed message (with LN node Private Key) where it _attests_ `A` remote balance. `A` is going to collect these proofs and save them on a local DB, ready to be shown on some public web application or CSV file.

The above scheme will produce a safer Proof of Reserve. Why?
## Trust minimized process
We can't force nodes to be honest with code, but social pressure can help us to disincentive bad actors.
If `A` wants to produce a lie, for example trying to declare more local balance than it has, it needs to corrupt a lot of nodes.  
The more channels someone has, the more secure the process will be.


## Digital Signature
Every Lightning Network node has a Private and a Public Key and every common Lightning implementation offers gRPC/REST API to sign and verify messages. Proof messages can be verified from everyone with the public keys of their nodes.

## Payload example
When a node want to collect attestations of the proofs, it will send a JSON request to every node they have a channel with on a specified endpoint, for example `GET http://SERVER_IP:SOME_PORT/proofs`
```
{
    "requestId": "some_unique_hash",
    "callerData": "A_pub_key+signature(A_pub_key)"
}
```
- `requestId` is just some unique identifier for the request.  
- `callerData` is used to create authenticated requests, an anti-spoofing mechanism to protect nodes from leaking remote balances information to unauthorized nodes. 

The node will then provide a response containing records about caller channels they have in common. 
Something like
```
{
    "proofs": [
        {
            "channelId": "some_channel_id",
            "funds": remote_msats,
        },
        {
            "channelId": "another_channel_id",
            "funds": remote_msats,
        }
    ],
    "snapshot_date": 1689625343 // unix timestamp of the provided snapshot,
    "signature": "signature_of_above_data"
}
```

The application inside this repo will contain tools needed to represent the requested data, for example a simple Web Application or CSV dump.

## Other considerations

- We need to think about Proof of Reserve for SLPs. We could implement the same messages format between SLP and mobile nodes (for example Breez/Phoenix/Bitkit) using sockets instead of rest.

- In the second scheme, node `A` doesn't need that every node run our script. In the scenario where 50% of the connected nodes run our software, A Proof of Reserve is going to display _x% coverage of funds_.