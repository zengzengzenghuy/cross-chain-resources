# Hyperlane
## Architecture

1. Mailbox is the endpoint of the a dapp that wants to send cross chain.
    1. Send: mailbox.dispatch
    2. Receive: IMessageRecipient.handle
    3. [Message](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/libs/Message.sol)
2. Fee:
    1. mailbox.quoteDispatch(): quote the fee needed to dispatch message. This fee include the requiredHook fee and defaultHook fee. Extra fee will get refunded, insufficient fee will result in reverted tx. Relayer get the fee and relay the message.
    2. Interchain fee [formula](https://docs.hyperlane.xyz/docs/reference/hooks/interchain-gas)
3. Hook: an extra configuration for your dapp that allows you to integrate with third party/native bridge. You need to inherit interface for Mailbox to call.
4. Router: a library pattern that allows the dapp to register it’s counterpart dapp on different chain, without having to specify the address on different chain for each call. It inherits MailboxClient.sol and will be called by Mailbox.
    1. The mapping of destination chain Router’s info is stored in [EnumerableMap](https://docs.openzeppelin.com/contracts/3.x/api/utils#EnumerableMap) from Openzeppelin. 
5. ISM: a contract defined by developer to customize the verification logic for their dapp. Inherit [IISM.so](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/interfaces/IInterchainSecurityModule.sol)l.
6. Warp Router: a token bridge contracts that mint and burn wrapped version of ERC20 and ERC721 token on destination chain. 
    1. `Collateral`: source chain token. Called HypERC20 Collateral.
        1. Collateral is deployed and keep track of the ERC20 contract. Token is transfer from ERC20 contract to collateral contract. → Mailbox and relayer pass the message → call HypERC20 mint on destination chain.
    2. `Synthetic`: wrapped version token in destination chain. Called HypERC20 or HypNative.
7. [Validator](https://docs.hyperlane.xyz/docs/operate/validators/run-validators): off chain validator signs the root of the Merkle tree of the source chain 
    1. the message signed is stored in storage (i.e. AWS S3)
    2. validator don’t need to call any function on chain, just need to listen to event from source chain. 
8. [Relayer](https://docs.hyperlane.xyz/docs/operate/relayer/run-relayer): relayer check if validator has signed the message and call Mailbox.handle on dest chain.

## Dev
### Interfaces
1. Send message: [source code](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/Mailbox.sol#L102)     
<details>
  <summary>Mailbox.dispatch</summary>

    ```
    contract Mailbox {

     /**
     * @notice Dispatches a message to the destination domain & recipient
     * using the default hook and empty metadata.
     * @param _destinationDomain Domain of destination chain
     * @param _recipientAddress Address of recipient on destination chain as bytes32
     * @param _messageBody Raw bytes content of message body
     * @return The message ID inserted into the Mailbox's merkle tree
     */
    function dispatch(
        uint32 _destinationDomain,
        bytes32 _recipientAddress,
        bytes calldata _messageBody
    ) external payable override returns (bytes32) {
        return
            dispatch(
                _destinationDomain,
                _recipientAddress,
                _messageBody,
                _messageBody[0:0],
                defaultHook
            );
    }

    /**
     * @notice Dispatches a message to the destination domain & recipient.
     * @param destinationDomain Domain of destination chain
     * @param recipientAddress Address of recipient on destination chain as bytes32
     * @param messageBody Raw bytes content of message body
     * @param hookMetadata Metadata used by the post dispatch hook
     * @return The message ID inserted into the Mailbox's merkle tree
     */
    function dispatch(
        uint32 destinationDomain,
        bytes32 recipientAddress,
        bytes calldata messageBody,
        bytes calldata hookMetadata
    ) external payable override returns (bytes32) {
        return
            dispatch(
                destinationDomain,
                recipientAddress,
                messageBody,
                hookMetadata,
                defaultHook
            );
    }

    }
    ```
    
</details>

2. Receive message: [source code](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/v3/solidity/contracts/interfaces/IMessageRecipient.sol#L4)     
   IMessageRecipient.handle is called by [Mailbox.process](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/v3/solidity/contracts/Mailbox.sol#L233).     
<details>
  <summary>IMessageRecipient.handle</summary>

    ```
    contract RecipientContract is IMessageRecipient{

        function handle(
        uint32 _origin,
        bytes32 _sender,
        bytes calldata _message
    ) external payable{

        // handle logic here
    }
    }
    ```
    
</details>

3. Message Format: [source code](https://github.com/hyperlane-xyz/hyperlane-monorepo/blob/main/solidity/contracts/libs/Message.sol)    
```
    function formatMessage(
        uint8 _version,
        uint32 _nonce,
        uint32 _originDomain,
        bytes32 _sender,
        uint32 _destinationDomain,
        bytes32 _recipient,
        bytes calldata _messageBody
    ) internal pure returns (bytes memory) {
        return
            abi.encodePacked(
                _version,
                _nonce,
                _originDomain,
                _sender,
                _destinationDomain,
                _recipient,
                _messageBody
            );
    }
```

4. Domain Identifier (uint32)    
Hyperlane uses domain id to identify each chain. Referece to [here](https://docs.hyperlane.xyz/docs/reference/domains)



## Hashi ISM

1. Hook contract
    1. on source chain
    2. called by Mailbox to pass the messageId, message
    3. the message in Hyperlane Mailbox.dispatch can be used to construct Hashi Message.
2. ISM contract
    1. on destination chain
    2. called by Mailbox to verify.
3. Recipient contract 
    1. inherit `ISpecifiesInterchainSecurityModule.interchainSecurityModule` and specify the ISM.

[Reference](https://docs.hyperlane.xyz/docs/reference/ISM/specify-your-ISM)


## Reference
1. General Message Passing: https://docs.hyperlane.xyz/docs/reference/messaging/messaging-interface
2. ISM: https://docs.hyperlane.xyz/docs/reference/ISM/specify-your-ISM
3. Contracts: https://github.com/hyperlane-xyz/hyperlane-monorepo/tree/main/solidity/contracts
