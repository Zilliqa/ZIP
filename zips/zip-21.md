|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 21  | EVM/Scilla interwork, phase 1 | Implemented | Standards Track  | Richard Watts <richard@zilliqa.com>, Bartosz Zawistowski <bartosz@zilliqa.com>  | 2023-07-17 | 2023-07-17

## Abstract

This ZIP details the facilities that are available in Zilliqa v9.2.0 and above for interwork between Scilla and EVM contract execution.

## Motivation

Zilliqa now supports the execution of EVM contracts. Whilst this allows users to treat native ZIL as native ETH (subject to some rounding due to the different granularities of ETH and ZIL), it does not allow native Zilliqa and EVM contracts to call each other or read each others' data.

This functionality will be delivered in two phases, of which this is the first, implemented in Zilliqa v9.2.0. This phase contains sufficient mechanism to allow a ZRC-2 token to appear as an ERC-20 token to other contracts.

Precompiles are specified as solidity function prototypes.

## Specification

This ZIP deals only with EVM -> Scilla calls (that is, where an EVM contract calls a Scilla contract) and specifies two new precompiles.

### Precompiles

#### `scilla_call_2`

```
scilla_call_2(contract_address, transition_name, call_mode, params...) [0x5a494c53]
```

This precompile allows EVM to invoke a Scilla contract transition.

Calling this precompile causes us to:

 * Marshal the given parameters into Scilla (see [Format Translation](#format) ).
 * Check `call_mode`:
   * `call_mode=0` - set `_sender` to the address of the contract that called `scilla_call_2` (like an EVM `call`)
   * `call_mode=1` - set `_sender` to `msg.sender` from the call of `scilla_call_2` (like `delegatecall`)
   * All other values produce undefined behaviour.
 * Sends a message to the contract at `contract_address` containing the `_tag` from `transition_name`, with parameters given in order by the `params`

The `_origin` is not changed and if Scilla code invoked by this call reverts, the entire transaction reverts.

#### `scilla_read`

```
scilla_read(contract_address, field_name, idx1, idx2, .. ) returns (value) [0x5a494c92]
```

This precompile allows EVM to read data from a scilla contract. Calling this precompile:

 * Looks up the `contract_address` and attempts to read the field `field_name`, indexed by `idx1`, `idx2`, .. etc if they exist.
 * If the field `field_name` does not exist, it will attempt to read the init parameter `field_name` in a similar way.
 * If this succeeds, it will translate the result to EVM representation (see below), place the result in `output` and return `true`
 * If it fails (because the field name or contract didn't exist, or because the type couldn't be translated), it will return `false`

### <a name="format">Format translation</a>

Format translation is done like this:

| Scilla type | Solidity type |
| -- | -- |
| IntX | IntX |
| UintX | UintX |
| String | String |
| ByStrX | String |
| ByStr20 | Address |

Maps are not directly supported, but you can read elements of a map.

When accessing maps, indices must be `Uint`, `Address`, or some kind of `String`.

### Callbacks

Many Scilla contracts issue callbacks to check that their counterparty is ready to receive funds. Without a way to handle events (yet) in EVM contracts, this would doom many useful interactions to failure if we didn't provide a way to cope with those callbacks.

`scilla_call_2` deals with this by:

  * Checking if the recipient of a Scilla message executed in the course of `scilla_call_2` is to an EVM contract.
  * If not, handle it as Zilliqa normally would.
  * Otherwise, check if the recipient implements the `ScillaReceiver` interface via EIP-165.
  * If the recipient supports `ScillaReceiver`, the transaction reverts (because there is no way to dispatch the callback)
  * If the recipient does not support `ScillaReceiver`, the callback is silently ignored (as though the recipient were an EOA).

The `ScillaReceiver` interface is declared as:

```
interface ScillaReceiver {
function handle_scilla_message(string tag, calldata rest) public payable;
}
```

In a future version of this specification, it will allow EVM contracts to accept Scilla callbacks.

### Events

Scilla events can now appear in an EVM transaction receipt.

Because popular libraries would object if they were faced with a "native" Scilla event, we encode them by treating a Scilla event `ev` as though it had been issued by a Solidity event `event ev(string)` with the `string` containing the JSON representing the Scilla event, just as it would have been represented in the `event_logs` array of a Scilla transaction receipt.

## Examples

### Calls

To call a two-argument Scilla transition from EVM with 21000 gas:

```
function _call_scilla_two_args(string memory tran_name, address recipient, uint128 amount) private {
   bytes memory encodedArgs = abi.encode(_zrc2_address, tran_name, CALL_MODE, recipient, amount);
   uint256 argsLength = encodedArgs.length;
   bool success;
   assembly {
       success := call(21000, 0x5a494c53, 0, add(encodedArgs, 0x20), argsLength, 0x20, 0)
   }
}
```

To read a `uint128` from a scilla contract:

```
    function _read_scilla_uint128(string memory variable_name) private view returns (uint128 supply) {
        bytes memory encodedArgs = abi.encode(_zrc2_address, variable_name);
        uint256 argsLength = encodedArgs.length;
        bool success;
        bytes memory output = new bytes(36);
        assembly {
            success := staticcall(21000, 0x5a494c92, add(encodedArgs, 0x20), argsLength, add(output, 0x20), 32)
        }
        require(success);
        (supply) = abi.decode(output, (uint128));
        return supply;
    }
```

### Events

The Scilla event

```
{ _eventname: “eventme”, _recipient: “0x8dfa2b543edb112ca80d46848aaaff14236a37a5”, test: “me” }
```

Can be encoded as the EVM event:

```
"events": [
    {
      "transactionIndex": 0,
      "blockNumber": 10451,
      "transactionHash": "0x017b2570cec5850f65519417042bcf43f5a3f568e71700591087fae2715fb06b",
      "address": "0xb3EaCf51AC95480a64763DA854A21FF92D537a75",
      "topics": [
        "0xd7089fd7b866cbe3193f0f744ed04c06e2d49a01b526b16fd281bb2fd341b4cd"
      ],
      "data": "0x0000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000005e7b225f6576656e746e616d65223a226576656e746d65222c225f726563697069656e74223a22307838646661326235343365646231313263613830643436383438616161666631343233366133376135222c2274657374223a226d65227d0000",
      "logIndex": 1,
      "args" : [
      "blockHash": "0xd18dbfa4dcb51a05c432275d4449c88119bb6136d0f380dbf80c955903f766c6",
      }
  ]
```

Which you can parse with (hardhat) code like:

```
function validateScillaEvent(scillaEventName: string, contractAddress: string, event: any) {
  expect(event["address"].toLowerCase()).to.eq(contractAddress.toLowerCase());
  const EXPECTED_TOPIC_0 = utils.keccak256(toUtf8Bytes(scillaEventName + "(string)"));
  expect(event["topics"][0].toLowerCase()).to.eq(EXPECTED_TOPIC_0.toLowerCase());
  const decodedData = defaultAbiCoder.decode(["string"], event["data"]);
  const scillaEvent = JSON.parse(decodedData.toString());
  expect(scillaEvent["_eventname"]).to.be.equal(scillaEventName);
}
```

### Code samples

These will appear in the Zilliqa [ZRC](https://github.com/zilliqa/zrc) repository.

## References

- [EIP-165](https://eips.ethereum.org/EIPS/eip-165)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
