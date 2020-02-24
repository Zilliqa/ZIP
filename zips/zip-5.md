|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 1  | Zilliqa Scilla External Library Support | Draft | Standards Track  | Haichuan Liu <haichuan@zilliqa.com> <br> Vaivaswatha Nagaraj vaivaswatha@zilliqa.com | 2020-02-20 | 2020-02-20

## Abstract

This ZIP details how Scilla External Library is supported in Zilliqa blockchain.

## Motivation

Since Scilla started to support importing library contract which has been deployed, Zilliqa blockchain also needs an update for supporting this feature. 

## Specification

There are some changes to `init.json` to make if you are trying to **deploy a library contract** or **import a library contract**:

- One new field in `init.json` is expected if deploy a **library contract**:
	```
	{
		"vname" : "_library",
		"type" : "Bool",
		"value": { "constructor": "True", "argtypes": [], "arguments": [] }
	}
	```
	- The `constructor` must be `True` for libray contract.
	- This field is optional for non-library contract.
- Another new field in `init.json` is expected if deploy a **contract which imports library contracts**:
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
	- The first element in `arguments` represents the name of the library to import, the second points to the address of that library contract in the blockchain.

For how to write Scilla library contract and import library in the contract code, please refer to [User-defined Libraries](https://scilla.readthedocs.io/en/latest/scilla-in-depth.html#user-defined-libraries)  .

## Rationale
## Backward Compatibility

Scilla external library contract is supported starting from Zilliqa version 6.1.1 and Scilla version 0.5.2.

## Test Cases

Here we demonstrate how library contracts are deployed and imported by other smart contract, all the contracts are deployed based on fixed order:
- The first library contract with address `0x986556789012345678901234567890123456abcd`:
	- [code](https://github.com/Zilliqa/scilla/blob/master/tests/contracts/0x986556789012345678901234567890123456abcd.scillib).
	- [init.json](https://github.com/Zilliqa/scilla/blob/master/tests/runner/0x986556789012345678901234567890123456abcd/init.json).
- The second library contract with address `0x111256789012345678901234567890123456abef`,  it imports the first library contract:
	- [code](https://github.com/Zilliqa/scilla/blob/master/tests/contracts/0x111256789012345678901234567890123456abef.scillib).
	- [init.json](https://github.com/Zilliqa/scilla/blob/master/tests/runner/0x111256789012345678901234567890123456abef/init.json).
- The normal smart contract account, which imports the second library contract:
	- [code](https://github.com/Zilliqa/scilla/blob/master/tests/contracts/import-test-lib.scilla).
	- [init.json](https://github.com/Zilliqa/scilla/blob/master/tests/runner/import-test-lib/init.json).

## References
- [PR for Integrating Scilla External Library in Zilliqa Repo](https://apidocs.zilliqa.com/#gettransaction)
- [Scilla documentation for user-defined library](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output)
- [Library import mechanism in Scilla]()

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
