---
eip: To be assigned
title: A means to provide natural-language descriptions of data in Ethereum signing prompts
author: Koh Wei Jie <weijie.koh@consensys.net>
discussions-to: Todo: add URL here
status: Draft
type: Standards Track
category: ERC
created: 2018-08-31
requires: EIP712
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->

This standard specifies how to translate data in wallet signing prompts into meaningful natural-language descriptions. This allows dApps to convey the intent behind each signature request, and users to make better-informed decsions about whether to sign a piece of data. It also reduces the risk of signing-prompt phishing.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

This standard specifies the means by which EIP712-compatible structured data should be converted into a text string. It includes:

* a simple *template processor* which translates data to be signed into a natural language string, including internationalisation support;
* an contract interface to set, get, and modify a translation template;
* suggestions for future extensions to the `eth_signTypedData` web3 call, particulary for dApps with contracts that are already deployed and cannot be upgraded.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->

Users of distributed applications (dApps) currently do not have a simple way to understand the data in a signing prompt. Although the [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) standard for typed structured data signing allows dApps to clearly lay out data in signing prompts, users have to rely on the dApp or their own research to find out what their signatures will be used for. This is not ideal especially because signatures can authorise value transfers in certain smart contracts. As such, there is a need to effectively convey the meaning and intent behind a signature request,  in order to improve the user experience for such potentially consequential operations.

To rely on dApps to do so creates a phishing risk, as malicious dApps may request that users sign data meant for a third-party contract, but misrepresent this in their off-wallet user interfaces.

Ethereum wallet signing prompts should therefore be respnsible for displaying these natural-language explanatory blurbs. Phishing can be prevented by validating translation templates off the contract whose address is specified in the EIP712 `domain` field.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->

### Definition of terms

**Data to be signed**

𝕊: EIP712-compatible typed structured data. Refer to the [EIP712 specification](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-typed-structured-data-%F0%9D%95%8A).

**Explanatory Blurb**

*E*: a short UTF-8 text string which explains, in natural language, what 𝕊 means or does.

**Template**

*T*: a generic explanatory blurb with placeholders for data.

**Template language**

A simple set of rules which govern the syntax and grammar of a template. Examples include [Mustache](https://mustache.github.io/) and [Jinja2](http://jinja.pocoo.org).

**Template processor**

*P*: a function that takes a template and EIP712-compatible data structure 𝕊 to produce and explanatory blurb *E*.

*P(T, 𝕊) → E*

**Language tag**

*g*: A [BCP 47 identifier ](https://tools.ietf.org/rfc/bcp/bcp47.txt) for a language, such as "en-US" or "zh-CN".

**Domain separator**

*domainSeparator*: as defined in [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-domainseparator).

**Multihash**

*m*: A self-describing hash protocol as defined [here](https://github.com/multiformats/multihash). It should use SHA256 and produce 32-byte base58 outputs.

### Mechanism of action

#### *Scenario 1*: the dApp provides *T*, and its contract provides *m(T)*

First, A dApp makes an`eth_signTypedData` RPC call to its Ethereum wallet provider. The parameters to this call should contain *domainSeparator* and 𝕊 in the `domain` and `message` fields respectively.

*domainSeparator* must contain the `verifyingContract`field that contains a valid Ethereum contract address *C*.

Additionally, the call should contain the `template` field alongside `domain` and `message`:
	
```
domain: { ... },
message: { ... }
template: {
	multihash: T,
	defaultLang: g
}
```

The wallet provider then calls *C* (see the *Ethereum contract interface*  section below) to retrieve *m(T)*, and uses it to verify *T*. It may use `g` as specified in `defaultLang`, or the user's preferred languge code.

Next, the wallet provider performs *P(T, 𝕊)* and displays the blurb *E*, as well as an option to view the raw data *𝕊*.

Finally, the wallet provider should indicate that *E* is can be trusted insofar as contract *C* is trustworthy.

#### *Scenario 2*: the dApp does not provide *T*, but its contract provides *m(T)*

In this case, the wallet should retrieve *T* by looking up *m(T)* via an [IPFS](https://ipfs.io/) gateway. The signing prompt, however, should inform the user that this translation has not been verified by the contract specified in `domainSeparator`.

#### *Scenario 3*: the dApp does not provide *T*, and contract *C* does not provide *m(T)*

1. The wallet's signing prompt should behave as per vanilla EIP712.

### Ethereum contract interface

#### Retrieve m(T)

```js
function eipXXXGetTemplateHash(string _g) returns bytes32
```

#### Set m(T)

```js
function eipXXXSetTemplateHash(string _g, bytes32 _mHash)
```

This should trigger the event:

```js
EipXXXTemplateHashSet(string indexed _g, bytes32 indexed _mHash)
```


## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

## Backwards Compatibility
<!--All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions
without a sufficient backwards compatibility treatise may be rejected outright.-->

This EIP is fully compatible with EIP712.

## Test Cases
<!--Test cases for an implementation are mandatory for EIPs that are affecting consensus changes. Other EIPs can choose to include links to test cases if applicable.-->

TODO

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

TOD

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
