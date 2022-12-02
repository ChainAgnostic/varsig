# Varsig Specification v0.1.0

## Editors

* [Irakli Gozalishvili](https://github.com/Gozala), [DAG House](https://dag.house/)
* [Brooklyn Zelenka](https://github.com/expede/), [Fission](https://fission.codes/)

## Authors

* [Irakli Gozalishvili](https://github.com/Gozala), [DAG House](https://dag.house/)
* [Joel Thorstensson](https://github.com/oed), [3Box Labs](https://3boxlabs.com/)
* [Brooklyn Zelenka](https://github.com/expede/), [Fission](https://fission.codes/)

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

## Dependencies

* [IPLD](https://ipld.io/docs/)
* [Multicodec](https://github.com/multiformats/multicodec)
* [Multihash](https://multiformats.io/multihash/)

# 0 Abstract

Varsig is a [multiformat](https://multiformats.io) for describing signatures over IPLD data and raw bytes in a way that preserves information about the payload and canonicalization information.

# 1 Introduction

[IPLD](https://ipld.io/docs) is a deterministic encoding scheme for data expressed in [common types](https://ipld.io/docs/data-model/kinds/) plus content addressed links. 

Common formats such as JWT use encoding (e.g. base64) and text separators (e.g. `"."`) to pass around encoded data and their signatures:

``` js
// JWT
"eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJhdWQiOiJkaWQ6a2V5Ono2TWtyNWFlZmluMUR6akc3TUJKM25zRkNzbnZIS0V2VGIyQzRZQUp3Ynh0MWpGUyIsImF0dCI6W3sid2l0aCI6eyJzY2hlbWUiOiJ3bmZzIiwiaGllclBhcnQiOiIvL2RlbW91c2VyLmZpc3Npb24ubmFtZS9wdWJsaWMvcGhvdG9zLyJ9LCJjYW4iOnsibmFtZXNwYWNlIjoid25mcyIsInNlZ21lbnRzIjpbIk9WRVJXUklURSJdfX1dLCJleHAiOjkyNTY5Mzk1MDUsImlzcyI6ImRpZDprZXk6ejZNa2tXb3E2UzN0cVJXcWtSbnlNZFhmcnM1NDlFZnU2cUN1NHVqRGZNY2pGUEpSIiwicHJmIjpbXX0.SjKaHG_2Ce0pjuNF5OD-b6joN1SIJMpjKjjl4JE61_upOrtvKoDQSxZ7WeYVAIATDl8EmcOKj9OqOSw0Vg8VCA"
```

Many text encodings of binary are inefficient and inconvenient. Others have opted to use canonicalization and a tag. This can be effective, but requires careful handling and signalling of the specific canonicalization method used.

``` js
const payload = canonicalize({"hello": "world", "count": 42})
{payload: payload, sig: key.sign(sha256(payload))}
```

Directly signing over canonicalized introduces new problems: forced encoding and canonicalization attacks. By taking advantage of existing IPLD canonicalization, varsig aims to alleviate both.

## 1.1 Forced Encoding

Data must first be rendered to binary before it is signed. This means choosing imposing encoding. There is no standard way to include the encoding that some IPLD was encoded with other than a CID. In IPFS, CIDs imply a link, which can have implications for network access and storage. Further, generating a CID means producing a hash, which is then potentially rehashed by the cryptographic signature library.

To remedy this, Varsig includes the encoding information used in production of the signature.

## 1.2 Canonicalization Attacks

Since IPLD is deterministically encoded, it can be tempting to not sign the IPLD data directly, and pass the signature around in the same payload rather than wrapping it. Since the original payload can be rederived from the output, this seems like a clean option:

``` js
// DAG-JSON
{
  "role": "user",
  "links": [
    {"/": "bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"},
    {"/": "bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"},
    {"/": "bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"}
  ],
  "sig": "8ufaS9w3CGN8cbQTUSoL1i7eaKiWLSXsD2LbZVmvM9zF"
}
```

This opens the potential for [canonicalization attacks](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/). Parsers are known to handle duplicate entries differently. IPLD needs to be serialized to a canonical form before checking the signature. Without careful handling, it is possible to fail to check if any additional fields have been added to the payload which will be parsed by the application.

> An object whose names are all unique is interoperable in the sense that all software implementations receiving that object will agree on the name-value mappings.  When the names within an object are not unique, the behavior of software that receives such an object is unpredictable.  Many implementations report the last name/value pair only.  Other implementations report an error or fail to parse the object, and some implementations report all of the name/value pairs, including duplicates.
>
> — [RFC8259](https://www.rfc-editor.org/rfc/rfc8259#page-10)

``` json
{
  "role": "user",  // Parsed by an IPLD implementation
  "role": "admin", // Malicious duplicate field, omitted by the IPLD parser, accpeted by the browser
  "links": [
    {"/": "bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"},
    {"/": "bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"},
    {"/": "bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"}
  ],
  "sig": "8ufaS9w3CGN8cbQTUSoL1i7eaKiWLSXsD2LbZVmvM9zF"
}
```

In the above example, the canonicalization step MAY lead to the signature validating, but the client parsing the `role: "admin"` field instead.

# 2 Safety

Data that has already been parsed to an in-memory IPLD representation can be canonically encoded to a trivially: it has already been through a [parser / validator](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/).

Data purporting to conform to an IPLD encoding (such as [DAG-JSON](https://ipld.io/specs/codecs/dag-json/spec/)) MUST be validated prior to signature verification. This can be as simple as round-trip decoding/encoding the JSON and checking that the hash matches. A validation error MUST be signalled if it does not match. 

> [Implementers] may provide an opt-in for systems where round-trip determinism is a desireable [sic] feature and backward compatibility with old, non-strict data is unnecessary.
>
> — [DAG-JSON Spec](https://ipld.io/specs/codecs/dag-json/spec/)

As it is critical for signatures guard against various attacks, the assumptions around canonical encoding MUST be enforced.

## 2.1 Signing CIDs

Rather than validating the IPLD, replacing the data with a CID link to the content MAY be used instead. Note while this is very safe (as it impractical to alter a signed hash), this approach mixes data layout with security, and may have a performance, disk, and networking impacts.

## 2.2 Raw (Noncanonicalized) Data

Canonicalization is not required if data is encoded as raw bytes (multicodec `0x00`). The exact bytes are already present, and MUST not be changed.

# 3 Varsig Format

A varsig is a bytestring that includes the following information:

| Segment Name              | Type     | Description                                     | Required |
|---------------------------|----------|-------------------------------------------------|----------|
| Varsig Prefix             | `0xD000` | The multicodec varsig prefix                    | Yes      |
| Public Key Prefix         | `Varint` | The multicodec prefix for the public key type   | Yes      |
| Hash Prefix               | `Varint` | The multicodec prefix for the hash algorithm    | Yes      |
| Hash Length               | `Varint` | The hash length                                 | Yes      |
| Content Multicodec Prefix | `Varint` | The IPLD encoding uses to canonicalize the data | Yes      |
| Raw Signature             | `Varint` | The raw signature                               | Yes      |
  
## 3.1 Segments

### 3.1.1 Varsig Prefix

The varsig prefix MUST be `0xD000`.

### 3.1.2 Public Key Prefix

The [multicodec](https://github.com/multiformats/multicodec) prefix for the public key type associated with the signature.

For example: `0xE7` for secp256k1, or `0x1205` for RSA.

### 3.1.3 Hash Prefix

The [multicodec](https://github.com/multiformats/multicodec) hash prefix used by the signature. This is the first segment of a [multihash](https://github.com/multiformats/multihash).

For example: `0x16` for SHA3-256, or `0x1E` for BLAKE3

### 3.1.4 Hash Length

The hash length used by the signature. This is the second segment of a [multihash](https://github.com/multiformats/multihash).

### 3.1.5 Content Multicodec Prefix

The [multicodec](https://github.com/multiformats/multicodec) prefix of the encoding scheme used.

For example: `0x00` for raw bytes (no special encoding), `0x0129` for DAG-JSON, and `0x70` for DAG-PB.

### 3.1.6 Raw Signature

The raw signature bytes

## 3.2 Byte Segments

```xml
<varint 0xD000><varint multicodec_key_prefix><varint multicodec_hash_prefix><varint multicodec_hash_length><varint multicodec_prefix><varint raw_hash><varint raw_signature>
```

## 3.3 IPLD Schema

``` ipldsch
type TaggedVarsig union {
  | Varsig "d0"
} representation bytesprefix

type Varsig struct {
  keyPrefix Multicodec.PublicKey
  hashPrefix Multicodec.Multihash
  digestLength Integer
  contentEnc Multicodec
  rawSig Bytes
} representation stringjoin {
  join ""
}
```

# 4 Further Reading

* https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/
* https://latacora.micro.blog/2019/07/24/how-not-to.html
* https://www.blackhat.com/presentations/bh-usa-07/Hill/Whitepaper/bh-usa-07-hill-WP.pdf

# 5 Acknowledgements

Many thanks to [Quinn Wilton](https://github.com/QuinnWilton) for her many recommendations and stories of times she's used JSON signing exploits in [CTF](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) competitions.
