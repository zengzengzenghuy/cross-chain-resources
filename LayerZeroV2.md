# LayerZero v2

## Architecture 
1. Endpoint: the main contract to interact with during message relaying / token bridging.
    1. `_lzSend`: internal function from endpoint called by sender dapp.
    2. `_lzReceive`: internal function from receiver dapp called by receiving chain endpoint.
2. MesssageLib: contract that manages configuration of bridging process, includes security stack, executors, and verification. It is the contract that endpoint talks to. There are two messageLib registry: Ultra Light Node 302 (default on EndpointV2) and Ultra Light Node 301.    
3. Security Stack (DVNs):  a set of onchain and offchain components that maange the security of bridging process, configurable by dapp. Each DVN has their own mechanism to verify the integrity of a message (w.r.t payloadHash).    
4.  Executor: an entity that will call `lzReceive` from endpoint contract to execute the message, configurable by dapp. 
5. Oapp: Omnichain dapp.    
6. OFT: Omnichain fungible token.    
6.

##  Dev
1. Packet struct [code](https://github.com/LayerZero-Labs/LayerZero-v2/blob/main/protocol/contracts/interfaces/ISendLib.sol#L8-L16)
```
struct Packet {
    uint64 nonce;
    uint32 srcEid;
    address sender;
    uint32 dstEid;
    bytes32 receiver;
    bytes32 guid;
    bytes message;
}
```
2. Packet Header (bytes) [code](https://github.com/LayerZero-Labs/LayerZero-v2/blob/main/protocol/contracts/messagelib/libs/PacketV1Codec.sol#L41)
```
 abi.encodePacked(
    PACKET_VERSION,
    _packet.nonce,
    _packet.srcEid,
    _packet.sender.toBytes32(),
    _packet.dstEid,
    _packet.receiver
 )
```
3. DVNAdapter Message 
- [encode](https://github.com/LayerZero-Labs/LayerZero-v2/blob/main/messagelib/contracts/uln/dvn/adapters/libs/DVNAdapterMessageCodec.sol#L24C15-L24C75):  `abi.encodePacked(_receiveLib, _payloadHash, _packetHeader);`    
4. setDVN: https://docs.layerzero.network/contracts/configure-dvns
5. setExecutor: https://docs.layerzero.network/contracts/executor-configuration    

## Reference:
1. LayerZero v2 Whitepaper: https://layerzero.network/publications/LayerZero_Whitepaper_V2.1.0.pdf
2. Docs: https://docs.layerzero.network/    
3. Github: https://github.com/LayerZero-Labs/LayerZero-v2    


