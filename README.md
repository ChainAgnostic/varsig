# Varsig Specification v0.1.0

## Editors

* [Irakli Gozalishvili](https://github.com/Gozala), [DAG House](https://dag.house/)
* [Brooklyn Zelenka](https://github.com/expede/), [Fission](https://fission.codes/)

## Authors

* [Irakli Gozalishvili](https://github.com/Gozala), [DAG House](https://dag.house/)
* [Joel Thorstensson](https://github.com/oed), [3Box Labs](https://3boxlabs.com/)
* [Quinn Wilton](https://github.com/QuinnWilton/), [Fission](https://fission.codes/)
* [Brooklyn Zelenka](https://github.com/expede/), [Fission](https://fission.codes/)

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

## Dependencies

* [IPLD]
* [Multicodec]

# 0 Abstract

Varsig is a [multiformat][Multiformats] for describing signatures over IPLD data and raw bytes in a way that preserves information about the payload and canonicalization information.

# 1 Introduction

[IPLD] is a deterministic encoding scheme for data expressed in [common types][IPLD Data Model] plus content addressed links. 

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

Data must first be rendered to binary before it is signed. This means imposing an encoding. There is no standard way to include the encoding that some IPLD was encoded with other than a CID. In IPFS, CIDs imply a link, which can have implications for network access and storage. Further, generating a CID means producing a hash, which is then potentially rehashed by the cryptographic signature library.

To remedy this, Varsig includes the encoding information used in production of the signature.

## 1.2 Canonicalization Attacks

Since IPLD is deterministically encoded, it can be tempting to rely on canonicalization at validation time, rather than rendering the IPLD to inline bytes or a CID and signing that. Since the original payload can be rederived from the output, this seems like a clean option:

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

This opens the potential for [canonicalization attacks]. Parsers for certain formats — such as JSON — are known to handle duplicate entries differently. IPLD needs to be serialized to a canonical form before checking the signature. Without careful handling, it is possible to fail to check if any additional fields have been added to the payload which will be parsed by the application. 

> An object whose names are all unique is interoperable in the sense that all software implementations receiving that object will agree on the name-value mappings.  When the names within an object are not unique, the behavior of software that receives such an object is unpredictable.  Many implementations report the last name/value pair only.  Other implementations report an error or fail to parse the object, and some implementations report all of the name/value pairs, including duplicates.
>
> — [RFC8259]

``` json
{
  "role": "user",  // Parsed by an IPLD implementation
  "role": "admin", // Malicious duplicate field, omitted by the IPLD parser, accepted by the browser
  "links": [
    {"/": "bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"},
    {"/": "bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"},
    {"/": "bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"}
  ],
  "sig": "8ufaS9w3CGN8cbQTUSoL1i7eaKiWLSXsD2LbZVmvM9zF"
}
```

In the above example, the canonicalization step MAY lead to the signature validating, but the client parsing the `role: "admin"` field instead.

## 1.2.1 Example

The above can be [quite subtle][PKI Layer Cake]. Here is a step by step example of one such scenario.

An application receives some block of data, as binary. It checks the claimed CID, which validates.

```
0x7ba202022726f6c65223a202275736572222ca202022726f6c65223a202261646d696e222ca2020226c696e6b73223a205ba202020207b222f223a20226261666b72656964623271336b7467746c6d3579696f3762756a337379707967686a7466683565726e737465716d616b66347032633562776d7969227d2c20202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020202020207b222f223a20226261666b72656963373579646735766b773332346f716b636d716c74667663336b6976796e67716b69626a6f7973647769696c616b68347a356665227d2ca202020207b222f223a20226261666b726569666664697a3672616634367a727233623275737566677a35666f34346167676d6f637a347a61707072366b6868686c6a63647079227da20205d2ca202022736967223a2022387566615339773343474e386362515455536f4c31693765614b69574c53587344324c625a566d764d397a4622a7d
```

Decoded to a string, the above reads as follows:

```
"{\n
  "role": "user",\n
  "role": "admin",\n
  "links": [\n
    {"/": "bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"},\n
    {"/": "bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"},\n
    {"/": "bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"}\n
  ],\n
  "sig": "8ufaS9w3CGN8cbQTUSoL1i7eaKiWLSXsD2LbZVmvM9zF"\n
}"
```

Note that the JSON above contains a duplicate `role` key a `sig` field with a base64 signature.

Next, the application parses the JSON with the browser's native JSON parser.

``` json
{
  "role": "admin", // Picked the second key
  "links": [
    {"/": "bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"},
    {"/": "bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"},
    {"/": "bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"}
  ],
  "sig": "8ufaS9w3CGN8cbQTUSoL1i7eaKiWLSXsD2LbZVmvM9zF"
}
```

The application needs to check the signature of all field minus the `sig` field. Under the assumption that the binary input was safe, and that canonicalization allows for the deterministic manipulation of the payload, the object is parsed to an internal IPLD representation using Rust/Wasm.

``` Rust
Ipld::Assoc([
    ("role", Ipld::String("user")),
    (
        "links",
        Ipld::Array([
            Ipld::Cid("bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"),
            Ipld::Cid("bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"),
            Ipld::Cid("bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"),
        ]),
    ),
    (
        "sig",
        Ipld::Binary([
            0xf2, 0xe7, 0xda, 0x4b, 0xdc, 0x37, 0x08, 0x63, 0x7c, 0x71, 0xb4, 0x13, 0x51, 0x2a,
            0x0b, 0xd6, 0x2e, 0xde, 0x68, 0xa8, 0x96, 0x2d, 0x25, 0xec, 0x0f, 0x62, 0xdb, 0x65,
            0x59, 0xaf, 0x33, 0xdc, 0xc5,
        ]),
    ),
]);
```

Note that the IPLD parser has dropped the `role: "admin"` key.

The `"sig"` field is then removed, and the remaining fields serialized to binary;

``` Rust
Ipld::DagJson::serialize(
    Ipld::Assoc([
        ("role", Ipld::String("user")),
        (
            "links",
            Ipld::Array([
                Ipld::Cid("bafkreidb2q3ktgtlm5yio7buj3sypyghjtfh5ernsteqmakf4p2c5bwmyi"),
                Ipld::Cid("bafkreic75ydg5vkw324oqkcmqltfvc3kivyngqkibjoysdwiilakh4z5fe"),
                Ipld::Cid("bafkreiffdiz6raf46zrr3b2usufgz5fo44aggmocz4zappr6khhhljcdpy"),
            ]),
        )
    ])
);
```

The signature is then checked against the above fields, which passes since there's only a `role: "user"` entry. The application then uses the original JSON with the `role: "admin"` entry.

# 2 Safety

Data that has already been parsed to an in-memory IPLD representation can be canonically encoded trivially: it has already been through a [parser / validator][Parse Don't Validate].

Data purporting to conform to an IPLD encoding (such as [DAG-JSON] MUST be validated prior to signature verification. This can be as simple as round-trip decoding/encoding the JSON and checking that the hash matches. A validation error MUST be signalled if it does not match. 

> [Implementers] may provide an opt-in for systems where round-trip determinism is a desireable [sic] feature and backward compatibility with old, non-strict data is unnecessary.
>
> — [DAG-JSON Spec][DAG-JSON]

As it is critical for guarding against various attacks, the assumptions around canonical encoding MUST be enforced.

## 2.1 Signing CIDs

Rather than validating the inline IPLD, replacing the data with a CID link to the content MAY be used instead. Note while this is very safe (as it impractical to alter a signed hash), this approach mixes data layout with security, and may have a performance, disk, and networking impacts.

### 2.1.1 Caching & Invalidation

Signing CIDs has two additional caching effects

* Signing CIDs enables a simple strategy for caching validation by CID
* Such a strategy also requires accounting for revocation of the signing keys, and so need to be tagged with this additional metadata

## 2.2 Raw (Noncanonicalized) Data

Canonicalization is not required if data is encoded as raw bytes (multicodec `0x55`). The exact bytes are already present, and MUST not be changed.

# 3 Varsig Format

After being decoded from [unsigned varint]s, a varsig includes the following segments:

```abnf
varsig = %x34 varsig-content-encoding varsig-header varsig-body
varsig-content-encoding = unsigned-varint ; The multicodec describing what's being signed over
varsig-header = unsigned-varint ; Usually the public key code from Multicodec
varsig-body = *unsigned-varint; Zero or more segments required by the kind of varsig (e.g. raw bytes, hash algorithm, etc)
```

For example, here is a EdDSA signature for content encoded as DAG-PB:

`0x3470ed01ae3784f03f9ee1163382fa6efa73b0c31ecf58c899c836709303ba4621d1e6df20e09aaa568914290b7ea124f5b38e70b9b69c7de0d216880eac885edd41c302`

### 3.1 Varsig Prefix

The varsig prefix MUST be `0x34`.

### 3.2 Content Multicodec Prefix

The [multicodec] prefix of the encoding scheme used before it's signed over. The default encoding SHOULD be [`0x55` "raw binary"][raw binary multicodec].

Many signature schemes depend on a hash function. Algorithm-sensitive hashing MUST be captured in the Varsig [header algorithm](#3-1-3-signature-header) or [body](#3-1-4-varsig-body), and so MUST NOT be captured in the Content Multicodec Prefix field.

Some examples include:

* `0x55` for raw bytes (no special encoding)
* `0x70` for DAG-PB
* `0x0129` for DAG-JSON
* `0x0202` for [CAR] files

### 3.3 Signature Header

The prefix of the signature algorithm. This is often the [multicodec] of the associated public key, but MAY be unique for the signature type. The code MAY live outside the multicodec table. This field MUST act as a discriminant for how many expected fields come in the varsig body, and what each of them mean.

### 3.4 Varsig Body

The varsig body MUST consist of zero or more segments required by the signature algorithm.

Some examples include:

* Raw signature bytes only
* CID of [DKIM] certification transparency record, and raw signature bytes
* Hash algorithm multicodec prefix, signature counter, HMAC, and raw signature bytes

# 4 Further Reading

* [Canonicalization Attacks Against MACs and Signatures](https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/)
* [How (not) to sign a JSON object](https://latacora.micro.blog/2019/07/24/how-not-to.html)
* [A Taxonomy of Attacks against XML Digital Signatures & Encryption](https://www.blackhat.com/presentations/bh-usa-07/Hill/Whitepaper/bh-usa-07-hill-WP.pdf)
* [PKI Layer Cake]

[CAR]: https://ipld.io/specs/transport/car/
[DAG-JSON]: https://ipld.io/specs/codecs/dag-json/spec/
[DKIM]: https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail
[IPLD Data Model]: https://ipld.io/docs/data-model/kinds/
[IPLD]: https://ipld.io/docs/
[Multicodec]: https://github.com/multiformats/multicodec
[Multiformats]: https://multiformats.io
[PKI Layer Cake]: https://link.springer.com/chapter/10.1007/978-3-642-14577-3_22
[Parse Don't Validate]: https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/
[RFC 2119]: https://datatracker.ietf.org/doc/html/rfc2119
[RFC8259]: https://www.rfc-editor.org/rfc/rfc8259#page-10
[canonicalization attacks]: https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/
[multicodec]: https://github.com/multiformats/multicodec
[raw binary multicodec]: https://github.com/multiformats/multicodec/blob/master/table.csv#L40
[unsigned varint]: https://github.com/multiformats/unsigned-varint
