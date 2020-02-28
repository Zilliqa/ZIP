|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 5  | Zilliqa Scilla External Library Support | Draft | Standards Track  | Haichuan Liu <haichuan@zilliqa.com> <br> Vaivaswatha Nagaraj <vaivaswatha@zilliqa.com> | 2020-02-20 | 2020-02-20

## Abstract

This ZIP details how Scilla External Library is supported in the Zilliqa core protocol.

## Motivation

With the Scilla External Library feature, Scilla is able to support the importing of library contracts that have been deployed. In order to fully support this feature, some changes need to be implemented in the Zilliqa core protocol.

## Specification

The Scilla External Library feature requires some changes to the `init.json`, which the core protocol must recognize during smart contract parsing. The changes to `init.json` to **deploy a library contract** or **import a library contract** are:

- One new field in `init.json` is expected if the deployment is a **library contract**:
	```
	{
		"vname" : "_library",
		"type" : "Bool",
		"value": { "constructor": "True", "argtypes": [], "arguments": [] }
	}
	```
	- The `constructor` must be `True` for library contract.
	- This field is optional for non-library contract.
- Another new field in `init.json` is expected if the deployment is a **contract which imports library contracts**:
	```
	{
		"vname" : "_extlibs",
		"type" : "List(Pair String ByStr20)",
		"value" : [
			{
				"constructor" : "Pair",
				"argtypes" : ["String", "ByStr20"],
				"arguments" : ["TestLib1", "0x986556789012345678901234567890123456abcd"]
			}
		]
	}
	```
	- The `value` field is an array, which supports importing multiple libraries.
	- The first element in `arguments` represents the name of the library to import, and the second points to the address of that library contract in the blockchain.

For writing Scilla library contracts and import libraries in the contract code, please refer to [User-defined Libraries](https://scilla.readthedocs.io/en/latest/scilla-in-depth.html#user-defined-libraries).

## Rationale
The changes in the core protocol are necessary and straightforward.

## Backward Compatibility

The changes described in this ZIP should not cause any compatibility issues with running non-library contracts.

## Test Cases

Here we demonstrate how library contracts are deployed and imported by other smart contracts. All the contracts are deployed based on fixed order:
- The first library contract with address `0x986556789012345678901234567890123456abcd`:
	- [code](https://github.com/Zilliqa/scilla/blob/master/tests/contracts/0x986556789012345678901234567890123456abcd.scillib)
	- [init.json](https://github.com/Zilliqa/scilla/blob/master/tests/runner/0x986556789012345678901234567890123456abcd/init.json)
- The second library contract with address `0x111256789012345678901234567890123456abef`, which imports the first library contract:
	- [code](https://github.com/Zilliqa/scilla/blob/master/tests/contracts/0x111256789012345678901234567890123456abef.scillib)
	- [init.json](https://github.com/Zilliqa/scilla/blob/master/tests/runner/0x111256789012345678901234567890123456abef/init.json)
- The normal smart contract account, which imports the second library contract:
	- [code](https://github.com/Zilliqa/scilla/blob/master/tests/contracts/import-test-lib.scilla)
	- [init.json](https://github.com/Zilliqa/scilla/blob/master/tests/runner/import-test-lib/init.json)

## References
- [PR for integrating Scilla External Library in the Zilliqa Repository](https://github.com/Zilliqa/Zilliqa/pull/1981)
- [Scilla documentation for User-defined Libraries](https://scilla.readthedocs.io/en/latest/scilla-in-depth.html#user-defined-libraries)
- [Scilla documentaion for Libray Import Mechanism](https://github.com/Zilliqa/scilla/wiki/Library-import-mechanism-in-Scilla)
## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
