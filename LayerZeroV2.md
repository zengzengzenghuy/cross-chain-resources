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

## Function/Event signatures
1. EndpointV2
```shell
| Function Name | Sighash    | Function Signature | 
| ------------- | ---------- | ------------------ | 
| quote | ddc28c58 | quote((uint32,bytes32,bytes,bytes,bool),address) |
| send | 2637a450 | send((uint32,bytes32,bytes,bytes,bool),address) |
| verify | a825d747 | verify((uint32,bytes32,uint64),address,bytes32) |
| lzReceive | 0c0c389e | lzReceive((uint32,bytes32,uint64),address,bytes32,bytes,bytes) |
| lzReceiveAlert | 6bf73fa3 | lzReceiveAlert((uint32,bytes32,uint64),address,bytes32,uint256,uint256,bytes,bytes,bytes) |
| clear | 2a56c1b0 | clear(address,(uint32,bytes32,uint64),bytes32,bytes) |
| setLzToken | c28e0eed | setLzToken(address) |
| recoverToken | a7229fd9 | recoverToken(address,address,uint256) |
| nativeToken | e1758bd8 | nativeToken() |
| setDelegate | ca5eb5e1 | setDelegate(address) |
| initializable | 861e1ca5 | initializable((uint32,bytes32,uint64),address) |
| verifiable | c9a54a99 | verifiable((uint32,bytes32,uint64),address) |
event PacketSent 0x1ab700d4ced0c005b164c0f789fd09fcbb0156d4c2041b8a3bfbcd961cd1567f
```
2. SendLib
```shell

| Function Name | Sighash    | Function Signature | 
| ------------- | ---------- | ------------------ | 
| send | 4389e58f | send((uint64,uint32,address,uint32,bytes32,bytes32,bytes),bytes,bool) |

ISendLib.sol
function send(
        Packet calldata _packet,
        bytes calldata _options,
        bool _payInLzToken
    ) external returns (MessagingFee memory, bytes memory encodedPacket);
    
    
 abstract SendLibBase
 
| Function Name | Sighash    | Function Signature | 
| ------------- | ---------- | ------------------ | 
| setDefaultExecutorConfigs | c14c4349 | setDefaultExecutorConfigs((uint32,(uint32,address))[]) |
| setTreasuryNativeFeeCap | d15b0d49 | setTreasuryNativeFeeCap(uint256) |
| getExecutorConfig | 188183f4 | getExecutorConfig(address,uint32) |

SendLibBaseE2
| _payVerifier | 895ceeb0 | _payVerifier((uint64,uint32,address,uint32,bytes32,bytes32,bytes),(uint8,bytes)[]) |
```