
|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 1  | Zilliqa Address Standard | Draft | Standards Track  | Ian Tan <iantanwx@gmail.com> <br> Han Wen Chua <hanwen@zilliqa.com> <br> Jun Hao Tan <junhao@zilliqa.com> | 2019-06-23 | 2020-01-30


## Abstract

This ZIP details the address standard adopted by the Zilliqa blockchain. This ZIP provides the concise details of the `bech32` address format customized for the Zilliqa protocol.

## Motivation

Ethereum's`EIP-55` proposed mixed-case checksum address encoding. Zilliqa uses a variant of `EIP-55` mixed-case checksum address.

The rationale of using a different mixed-case checksum address are as such:
- to prevent confusion with Ethereum mixed-case checksum address
- to prevent the loss of funds arising from sending to the wrong network address

> Note: Using Zilliqa’s native protocol address format that consists of 20 bytes led to some confusion due to similarity with the Ethereum address format. Any tokens mistakenly sent to the wrong address would have led to an irreversible loss of the tokens. This is because the private key corresponding to an address on Zilliqa does not correspond to the same private key on Ethereum, due to the difference of the underlying hash function used (`sha256` in Zilliqa vs `keccak` in Ethereum).

However, `EIP-55` is still not widely adopted by wallets and exchanges within the Ethereum ecosystem. Checks for checksum are not present in some implementation wallets or exchanges integration. As such, the original intention of using a different mixed-cased checksum variation for Zilliqa address format to prevent the loss of funds become not viable.

Hence, this ZIP proposes that Zilliqa adopts a variation of the `bech32` format on the wallets/SDKs level to prevent users from sending interim ERC20 ZIL tokens from their Ethereum wallets (i.e. MyCrypto/MyEtherWallet) to a native $ZIL address and vice versa.

The native protocol continues utilize the 20-bytes `base16` checksum on the backend. As such, this is a cosmetic change of the 20-bytes `base16` checksum address to `bech32` format on the wallets and SDKs level only. It is only be visible to end-users.

## Specification

Please refer to [`bip-0173`](https://github.com/bitcoin/bips/blob/master/bip-0173.mediawiki#bech32) for more details of the `bech32` technical specification.

A Zilliqa `bech32` checksummed address consists of the following aspects:

|               | Human-readable prefix | Separator | `bech32` formatted address         | Checksum |
| ------------- | --------------------- | --------- | ---------------------------------- | -------- |
| **Example 1** | `zil`                 | `1`       | `02n74869xnvdwq3yh8p0k9jjgtejruft` | `268tg8` |
| **Example 2** | `zil`                 | `1`       | `48fy8yjxn6jf5w36kqc7x73qd3ufuu24` | `a4u8t9` |
| **Example 3** | `zil`                 | `1`       | `fdv7u7rll9epgcqv9xxh9lhwq427nsql` | `58qcs9` |

- Do note that the human-readable prefix for [Zilliqa mainnet](https://viewblock.io/zilliqa) is `zil`, and the human-readable prefix for [Zilliqa Developer testnet](https://viewblock.io/zilliqa?network=testnet) is also `zil`.
- Do also note that the last six characters of the data part form a checksum and contain no information.

## Rationale

### Bech32 address format

Other address format such as `Base32` were explored. However, `Bech32` address format turns out to be a better fit due to
- human-readable prefix explictly convey that the address is a Zilliqa address
- Prevent confusion with Ethereum's checksummed address format

## Backward Compatibility

This ZIP is backward compatible as the required changes are only on the wallet and SDKs level. There is no change required on the core protocol layer.

## Test Cases 

This ZIP uses the public key `039fbf7df13d0b6798fa16a79daabb97d4424062d2f8bd4e9a7c7851e732a25e1d` as a test case and example:

- Mainnet legacy `base16` checksummed address: `0x7Aa7eA9f4534d8D70224b9c2FB165242F321F12b`
- Mainnet `bech32` checksummed address: `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8`
- Testnet `bech32` checksummed address: `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8`

## Implementations

### Reference encoder and decoder

#### SDKs

  - [For JavaScript](https://github.com/Zilliqa/Zilliqa-JavaScript-Library/blob/dev/packages/zilliqa-js-crypto/src/bech32.ts)
  - [For Golang](https://github.com/Zilliqa/gozilliqa-sdk/blob/master/bech32/bech32.go)
  - [For Java](https://github.com/FireStack-Lab/LaksaJ/blob/master/src/main/java/com/firestack/laksaj/utils/Bech32.java)
  - [For Python](https://github.com/deepgully/pyzil/blob/master/pyzil/crypto/bech32.py)

#### Online utility tool

  - [Zilliqa Address Tool](https://www.coinhako.com/zil-check)

#### Sample sanity implementation

In order to support both `bech32` (default) and legacy `base16` (optional) address formats, it is recommended to refer to the code snippet below to perform a sanity check with the utility tools provided by the [`zilliqa-js` SDK](https://github.com/Zilliqa/Zilliqa-JavaScript-Library):

```javascript
  private normaliseAddress(address: string) {
    if (validation.isBech32(address)) {
      return fromBech32Address(address);
    }

    if (isValidChecksumAddress(address)) {
      return address;
    }

    throw new Error('Wrong address format, should be either bech32 or checksummed address');
  }
```

### Explorer

[Viewblock explorer](https://viewblock.io/zilliqa) supports both the new `bech32` and legacy `base16` checksummed address formats in the search bar to ease the transition to `bech32` checksummed addresses for all Users.

## References

- [Zilliqa Migrates to New Address Format](https://blog.zilliqa.com/zilliqa-migrates-to-new-address-format-bf1fa6d7e41d)
- [Zilliqa Address Standard wiki (Depreciated)](https://github.com/Zilliqa/Zilliqa/wiki/Address-Standard)

## Copyright Waiver 

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
