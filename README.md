# EIP-7749: wallet_signIntendedValidatorData 🚀

## Table of Contents

* [Active Links](#active-links)
* [Summary](#summary)
* [History](#history)
* [Motivation](#motivation)
* [Specification](#specification)
* [Support](#support)
* [Discussions](#discussions)

## Links

* PR to [add `wallet_signIntendedValidatorData` JSON-RPC method](https://github.com/ethereum/EIPs/pull/8774)
* [Ethereum Magicians Discussion](https://ethereum-magicians.org/t/eip-7749-add-wallet-signintendedvalidatordata-method/20693)
* [Discussion in Metamask Forum](https://community.metamask.io/t/add-support-for-eipe-191-version-0-intended-validator-data/28940) (+15 votes)
* Discussion in viem to support [wallet_signIntendedValidatorData](https://github.com/wevm/viem/discussions/2225)

## Summary

This proposal adds a new JSON-RPC method, `wallet_signIntendedValidatorData`, which allows signing data with an intended validator address using [EIP-191] version [0x00] with this format:

```bash
0x19 <0x00> <intended validator address> <data to sign>
```

## History

There has always been work to integrate signing methods in various applications, but the focus was on `eth_sign` as it was already in use, and everyone adopted it. Although [EIP-191] defines different versions serving different purposes, versions `0x00` and `0x45` (E) do not serve the same purpose.

In this issue, we can see that [EIP-191] was almost fully supported, but precedence led to only versions [0x45] and [0x01] being implemented, dropping version [0x00].

Due to this, several apps have deviated from version [0x00], although it makes more sense to use it than [0x45]. For example, in the issue mentioned above, the team deviated from [0x00] and chose [0x45], even though a smart contract account (Multisig) should handle verification and signing for one intended validator address.

This was the issue where support for [EIP-191] Full was being worked on: [GitHub Issue](https://github.com/nucypher/nucypher/issues/1566)

## Motivation

Currently, signing messages relies heavily on [EIP-191] version [0x45] (`eth_sign`) and [EIP-712] (`eth_signTypedDataV4`). While [EIP-712] provides a more structured approach, it is often seen as complex. On the other hand, [EIP-191] version [0x45] is widely used but poses significant phishing risks due to the lack of data parsing.

EIP-191 defines three versions: [0x45], [0x01], and [0x00]. This proposal aims to fully support [EIP-191] by introducing the rpc call for [0x00] version, which enables signing data with an intended validator address. This new method will:

- Enable more dApps to use [EIP-191] version [0x00] without using raw signing methods which might be dangerous and restricted in few wallets.
- Enhance security by parsing data and displaying the intended validator address, reducing phishing risks.
- Provide a simpler alternative to [EIP-712], offering a balance between usability and security.
- Be particularly relevant for smart contract accounts, allowing signing with a specific intended validator address.

With the rise of smart contract accounts and the reliance on signatures to improve UX, the need for supporting [EIP-191] version [0x00] increases, especially given the prevalence of verifier smart contracts, such as Entry Points, Smart Contract Accounts, Key Managers, etc.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### `wallet_signIntendedValidatorData`

MUST calculate an Ethereum signature using `sign(keccak256("\x19\x00<signature validator address><data to sign>"))`.

This method adds a prefix to the message to prevent malicious dApps from signing arbitrary data (e.g., a transaction) and using the signature to impersonate the victim.

#### Parameters

1. `DATA` - 20-byte account address: The address signing the constructed message.
2. `DATA` - 20-byte account address: The intended validator address included in the message to sign.
3. `DATA` - Data string: The data to sign.

#### Returns

`DATA` - Signature.

#### Example

**Request:**

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"wallet_signIntendedValidatorData","params":["0x6aFbBC5e6AFcB251371711a6551E60ead2779Dc0", "0x345B918b9E06fAa7B0e56bd71Ba418F31F47FED4", "0x59616d656e"], "id":1}'
```

```json
{
  "jsonrpc": "2.0",
  "method": "wallet_signIntendedValidatorData",
  "params": [
    "0x6aFbBC5e6AFcB251371711a6551E60ead2779Dc0",
    "0x345B918b9E06fAa7B0e56bd71Ba418F31F47FED4",
    "0x59616d656e"
  ],
  "id": 1
}
```

**Result:**

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "0x4355c47d63924e8a72e509b65029052eb6c299d53a04e167c5775fd466751c9d07299936d304c153f6443dfa05f40ff007d72911b6f72307f996231605b915621c"
}
```

## Rationale

The `wallet_signIntendedValidatorData` method aims to bridge the gap between the simplicity of [EIP-191] version [0x45] and the structured approach of [EIP-712]. By specifying the intended validator address, it reduces phishing risks and provides a more secure signing method for smart contract accounts and other use cases requiring a specific validator address.

## Backwards Compatibility

No backward compatibility issues found.

## Security Considerations

Users should exercise caution when signing messages. Double-check the address of the verifier and ensure trust in the dApp triggering the sign request.

To protect against replay attacks and cross-chain replay attacks, include chainId and nonce in the validator data to sign.

## Support

We see many members of the ETH ecosystem tired of the hacks that happen due to blind signing. Even if this standard is not perfect, it at least provides an idea of which verifier will verify the message. If you trust the verifier, then it's one extra step towards safety (not full safety as users can't know what bytes they are signing, but still).

In the Metamask forum, it received support and was praised by [nick.eth](https://x.com/nicksdjohnson), Lead developer of ENS & Ethereum Foundation alum.

* [Nick's Tweet](https://x.com/0xYamen/status/1790736305539661977)
* [Metamask Forum - 15+ Votes](https://community.metamask.io/t/add-support-for-eipe-191-version-0-intended-validator-data/28940)

## Discussions

Discussions are enabled for this repo. Feel free to ask or discuss anything.

[EIP-191]: https://eips.ethereum.org/EIPS/eip-191
[EIP-712]: https://eips.ethereum.org/EIPS/eip-712
[0x00]: https://eips.ethereum.org/EIPS/eip-191#version-0x00
[0x01]: https://eips.ethereum.org/EIPS/eip-191#version-0x01
[0x45]: https://eips.ethereum.org/EIPS/eip-191##version-0x45-e
