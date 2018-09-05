---
eip: To be assigned
title: A means to provide natural-language descriptions of data in Ethereum signing prompts
author: Koh Wei Jie <weijie.koh@consensys.net>
discussions-to: Todo: add URL here
status: Draft
type: Standards Track
category: ERC
created: 2018-09-05
requires: EIP712
---

## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->

This standard specifies how to translate data in wallet signing prompts into meaningful natural-language descriptions. This allows dApps to convey the intent behind each signature request, and users to make better-informed decisions about whether to sign a piece of data. It also reduces the risk of signing-prompt phishing.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->

This standard specifies the means by which EIP712-compatible structured data should be converted into a text string. It includes:

* a simple *template processor* which translates data to be signed into a natural language string, including internationalisation support;
* an contract interface to get and set a translation template;
* a simple way to verify translation templates;
* suggestions for future extensions to the `eth_signTypedData` web3 call, particulary for dApps with contracts that are already deployed and cannot be upgraded.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->

Users of distributed applications (dApps) currently do not have a straightforward way to understand data in signing prompts. Although the [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md) standard for typed structured data signing allows dApps to clearly lay out such data, users have to rely on the dApp or their own research to find out what their signatures will be used for. This is less than ideal especially because signatures can authorise value transfers in certain smart contracts. As such, there is a need to effectively convey the meaning and intent behind a signature request,  in order to improve the user experience for such potentially consequential operations.

To rely on dApps to do so creates a phishing risk, as malicious dApps may request that users sign data meant for a third-party contract, but misrepresent this in their off-wallet user interfaces.

As such, Ethereum wallet signing prompts should be respnsible for displaying these natural-language explanatory blurbs. Furthermore, phishing can be prevented by validating translation templates off their cryptographic hash obtained from the contract specified in the EIP712 `domain` field.

## Example

Raw EIP712 typed data:

```
message: {
    from: {
        name: 'Cow',
        wallet: '0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826',
    },
    to: {
        name: 'Bob',
        wallet: '0xbBbBBBBbbBBBbbbBbbBbbbbBBbBbbbbBbBbbBBbB',
    },
    contents: 'Hello, Bob!',
}
```

Translation template:

```
{{message.from.name}} ({{message.from.wallet | shorten}}) is sending the message "{{message.contents}}!" to {{message.to.name}} ({{message.to.wallet | shorten}})."
```

Translated result:

```
Cow (0xCD2a3d9F... ) is sending the message "Hello, Bob!" to Bob (0xbBbBBBBb...)."
```

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->

### Definition of terms

**Data to be signed**

𝕊 (henceforth *S*): EIP712-compatible typed structured data. Refer to the [EIP712 specification](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-typed-structured-data-%F0%9D%95%8A).

**Explanatory blurb**

*E*: a short UTF-8 text string which explains, in natural language, what S means or does.

**Template**

*T*: a generic explanatory blurb with placeholders for data.

**Template language**

A simple set of rules which govern the syntax and grammar of a template. This EIP proposes a simple default template language (see the *Default Template Langauge* section), but can be extended by future EIPs to allow dApps to specify other template languages.

**Template processor**

*P*: a function that takes a template and EIP712-compatible data structure S to produce and explanatory blurb *E*.

*P(T, S) → E*

**Language tag**

*g*: A [BCP 47 identifier ](https://tools.ietf.org/rfc/bcp/bcp47.txt) for a language, such as `en-US` or `zh-CN`.

<!-- **Domain separator**

*domainSeparator*: as defined in [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md#definition-of-domainseparator). -->

**Multihash**

*m*: A self-describing hash protocol as defined [here](https://github.com/multiformats/multihash). It should use the SHA256 hash function. [Excluding the hash function and size bytes](https://ethereum.stackexchange.com/a/17112), it should produce 32-byte, base58 outputs which can be later reconstructed to match the [IPFS CIDV0 format](https://docs.ipfs.io/guides/concepts/cid/#version-0).

### Mechanism of action

In the context of this EIP, there are 5 possible scenarios for when a dApp makes an`eth_signTypedData` RPC call to its Ethereum wallet provider.

For all scenarios, we assume that the parameters of the call contain the `template` field alongside the EIP712`domain` and `message` fields.

We also assume that `domain` must contain a `verifyingContract`field that contains a valid Ethereum contract address *C*.


```
domain: {
    verifyingContract: C,
    ...
},
message: S,
template: {
    template: T,
    defaultLang: g
}
```

dApps must include `defaultLang`, or risk a degraded user experience as the wallet provider has nothing to fall back upon if the contract has no template which corresponds to the user's preferred language code.

|  Scenarios | dApp provides *T*  | Contract provides *m(T)*  |
|---|---|---|
| 1 and 5  | Y  | Y  |
| 2 | Y | N |
| 3  | N  | Y  |
| 4 | N  | N  |

#### Scenario 1: the dApp provides the template *T*, and its contract provides the template multihash *m(T)*.

The wallet provider should call *C* (see the *Ethereum contract interface*  section below) through an Ethereum node with the `keccak256` hash of the desired language code *g* to obtain *m(T)*. It then uses *m(T)* to verify *T*.

Next, the wallet provider applies the template to the data to obtain the blurb and display it: **P(T, S) → E*. It must also give the user the option to view the raw data *S*.

Finally, the wallet provider should indicate that *E* is can be trusted as long as as contract *C* is trustworthy.

#### Secnario 2: the dApp provides the template *T*, but the contract *C* does not provide its multihash *m(T)*.

The wallet should warn the user that the translated blurb *E* is unverified, and give the user the option to view it. Otherwise, the blurb should be hidden from the user by default.

#### Scenario 3: the dApp does not provide the template *T*, but its contract provides the template multihash *m(T)*.

Since IPFS uses multihashes for content addressing, the wallet should retrieve *T* via [IPFS](https://ipfs.io/).

#### Scenario 4: the dApp does not provide the template *T*, and contract *C* does not provide its multihash *m(T)*.

The wallet's signing prompt should behave as per vanilla EIP712.

#### Scenario 5: the multihash *m(T)* provided by the contract *C* does not match the calculated multihash of the dApp-provided template *T*.

The wallet should warn the user that the dApp may be attempting a phishing attack.

### Ethereum contract interface

#### Retrieve the template multihash *m(T)*

TODO: replace eipXXX with the this EIP number

```js
function eipXXXGetTemplateHash(string _g) returns bytes32
```

#### Set template multihash *m(T)*

This should allow the contract owner(s) to modify *m(T)* stored in the contract if they update *T* provided by the dApp or upload a new version to IPFS.

`_gHash` is the `keccak256()` hash of `g`. This allows contracts to maintain an internal `mapping(bytes32 => bytes32)` to store template hashes with minimal gas use.

```js
function eipXXXSetTemplateHash(bytes32 _gHash, bytes32 _mHash)
```

It should trigger the event:

```js
EipXXXTemplateHashSet(bytes32 indexed _gHash, bytes32 indexed _mHash)
```

## Default template language

This EIP proposes a very simple default language. A trimmed down version of [Jinja2](http://jinja.pocoo.org/docs/2.10/templates), it only includes variable subsitution, conditionals, and a limited number of text filters.

### Variable subsitution

Use double curly braces: `{{ variable_name }}`

Variable scope is the typed data *S* passed to the wallet via `eth_signTypedData`.

Following the [Ether Mail example in EIP712](https://github.com/ethereum/EIPs/blob/master/assets/eip-712/Example.js), take the following as valid examples:

`{{ message.from.name }}` → `Cow`

`{{ message.from.wallet }}` → `0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826`

`{{ domain.name }}` → `Ether Mail`

All text is HTML-escaped by default. `<b>hello</b>` will display exactly as `<b>hello</b>`.

To avoid recursion, variables that reference`template` field will be ignored.

### Expressions

Almost all [Jinja2 2.10](http://jinja.pocoo.org/docs/2.10/templates/#expressions) expressions are supported, except `()`.

List indices start from 0 and use square brackets: `var[0]`.

Supported expressions:

| Expression | Type |
|---|---|  
| `"abc"`| String |
| `42` or `42.23` | Number |
| `[‘list’, ‘of’, ‘objects’]` | List |
| `true` or `false` | Boolean |
| `+` | Add |
| `-` | Subtract |
| `*` | Multiply |
| `/` | Divide |
| `//` |Divide and floor |
| `%` | Modulo  |
| `**` |  Raise to the power of |
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `>=` | Greater than or equal to |
| `<` | Smaller than |
| `<=` | Smaller than or equal to |
| `in` | Contains |

### Conditionals

Use `if`, `elif`, and `else`.

```
{% if message.temp < 0 %}
    It's below freezing.
{% elif message.temp == 0 %}
    It's exactly 0 degrees.
{% else %}
    It's above freezing.
{% endif %}
```

For simplicity and security, it delibrately excludes:

1. Custom formatting
2. Anchor links
3. Loops

### Filters

Use the `|` symbol to modify a variable using a filter.

The following filters are supported:

| Filter | Only for | Description |
|---|---|---|
| `shorten`| ETH addresses | Display in a truncated format |

For simplicity, this EIP will not support any other filters; these will have to come in a future EIP.

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

The following assumptions and considerations shaped this EIP:

1. IPFS is an ideal means of template storage as it is decentralised and censorship-resistant.
2. To save gas, templates should not be stored in contracts. Instead, contracts should store their multihash, so they can be retrieved from IPFS.
3. A simpler default template language is more secure and phishing-resistant than a complex one.
4. In the context of this EIP, wallet providers are only responsible for ensuring that messages are translated according to the contract's specified template. Wallet providers are not responsible for what a said contracts do with signed data.

## Backwards Compatibility
<!--All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions
without a sufficient backwards compatibility treatise may be rejected outright.-->

This EIP is fully compatible with EIP712.

## Future work

There is a need to provide a means for wallet providers to verify templates for contracts which have not implemented the interface described above, or cannot be [upgraded](https://eips.ethereum.org/EIPS/eip-897) to do so. This can be done in a future EIP.

Future EIPs may also extend the `template` field to specify non-default template processing languages.

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

TODO


## Credits

Many thanks to Raman Shalupau, ___, ___, and for their feedback and comments.

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
