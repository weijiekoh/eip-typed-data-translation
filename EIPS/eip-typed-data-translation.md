---
```
eip: <to be assigned>
title: A way to provide meaningful descriptions of messages to be signed
author: Koh Wei Jie <weijie.koh@consensys.net>
discussions-to: <URL>
status: Draft
type: Standards Track
category: ERC
created: 2018-08-31
requires: EIP712
```
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough."
Provide a simplified and layman-accessible explanation of the EIP.-->

This ERC specifies how wallet providers and dApp authors should translate data
meant to be signed into meaningful natural-language descriptions. dApps can
thereby convey the intent behind each signature request, and users can
make better-informed decsions about whether to sign a piece of data.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
This standard specifies the means by which EIP712-compatible structured data
should be converted into a text string. It includes:

* a simple *template processor* which translates data to be signed into a natural
  language string;
* a standard for template internationalisation;
* a Solidity interface to set, get, and modify a translation template at the
  contract level;
* an extension of the EIP712 `eth_signTypedData` method to allow dApps to
  indicate to a wallet provider which template to use to perform a translation.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum
protocol. It should clearly explain why the existing protocol specification is
inadequate to address the problem that the EIP solves. EIP submissions without
sufficient motivation may be rejected outright.-->

There is a need to enable dApps to effectively convey the *intent* behind
asking for a user's signature.

Users of distributed applications (dApps) do not have a simple way of finding
out what the data they see in a signing prompt means. This degrades the dApp
user experience, especially because some signatures authorise fund transfers
and other operations on value. Although the standard for typed structured data
signing -
[EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) - allows
dApps to format messages to be signed and thereby provide greater clarity about
the composition of said messages, users have to rely on the dApp or their own
research to understand what their signatures will be used for.

## Specification
<!--The technical specification should describe the syntax and semantics of any
new feature. The specification should be detailed enough to allow competing,
interoperable implementations for any of the current Ethereum platforms
(go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and
[others](https://github.com/ethereum/wiki/wiki/Clients)).-->

## Rationale
<!--The rationale fleshes out the specification by describing what motivated
the design and why particular design decisions were made. It should describe
alternate designs that were considered and related work, e.g. how the feature
is supported in other languages. The rationale may also provide evidence of
consensus within the community, and should discuss important objections or
concerns raised during discussion.-->

## Backwards Compatibility
<!--All EIPs that introduce backwards incompatibilities must include a section
describing these incompatibilities and their severity. The EIP must explain how
the author proposes to deal with these incompatibilities. EIP submissions
without a sufficient backwards compatibility treatise may be rejected
outright.-->

## Test Cases
<!--Test cases for an implementation are mandatory for EIPs that are affecting
consensus changes. Other EIPs can choose to include links to test cases if
applicable.-->

## Implementation
<!--The implementations must be completed before any EIP is given status
"Final", but it need not be completed before the EIP is accepted. While there
is merit to the approach of reaching consensus on the specification and
rationale before writing code, the principle of "rough consensus and running
code" is still useful when it comes to resolving many discussions of API
details.-->

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
