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

Varsig is a [multiformat](https://multiformats.io) for describing signatures over IPLD data and raw bytes in a way that preserves information about the payload and cacnonicalization information.

# 1 Introduction

[IPLD](https://ipld.io/docs) is a deterministic encoding scheme for data expressed in [common types](https://ipld.io/docs/data-model/kinds/) plus content addressed links. 

Common formats such as JWT use encoding (e.g. base64) and text separators (e.g. `"."`) to pass around encoded data and their signatures:

``` js
// JWT
eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCIsInVjdiI6IjAuOC4xIn0.eyJhdWQiOiJkaWQ6a2V5Ono2TWtyNWFlZmluMUR6akc3TUJKM25zRkNzbnZIS0V2VGIyQzRZQUp3Ynh0MWpGUyIsImF0dCI6W3sid2l0aCI6eyJzY2hlbWUiOiJ3bmZzIiwiaGllclBhcnQiOiIvL2RlbW91c2VyLmZpc3Npb24ubmFtZS9wdWJsaWMvcGhvdG9zLyJ9LCJjYW4iOnsibmFtZXNwYWNlIjoid25mcyIsInNlZ21lbnRzIjpbIk9WRVJXUklURSJdfX1dLCJleHAiOjkyNTY5Mzk1MDUsImlzcyI6ImRpZDprZXk6ejZNa2tXb3E2UzN0cVJXcWtSbnlNZFhmcnM1NDlFZnU2cUN1NHVqRGZNY2pGUEpSIiwicHJmIjpbXX0.SjKaHG_2Ce0pjuNF5OD-b6joN1SIJMpjKjjl4JE61_upOrtvKoDQSxZ7WeYVAIATDl8EmcOKj9OqOSw0Vg8VCA
```

Many encodings are less efficient and inconvenient, so others use canonicalization and a tag. This can be 

FIXME

``` js
const payload = {"hello": "world", "count": 42}.toString()

{
  payload: payload,
  sig: key.sign(payload + sha256(payload)
}
```


Directly signing over IPLD introduces new problems: loss of metadata and canonicalization attacks. Varsig aims to aleviate both.

## 1.1 Loss of Metadata

## 1.2 Canonnicalization Attacks

Since IPLD is deterministically encoded, it can be tempting to not sign the IPLD data directly, and pass the signature around in the same payload rather than wrapping it. Since the original paylaod can be rederived from the output, this seems like a clean option:

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

However, since it requires canonicalization before checking the signature, it is very easy to miss extra fields that have been added to the payload, and that were lost in canonicalization. Since different parsers _________. From [RFC8259](https://www.rfc-editor.org/rfc/rfc8259#page-10):

> An object whose names are all unique is interoperable in the sense that all software implementations receiving that object will agree on the name-value mappings.  When the names within an object are not unique, the behavior of software that receives such an object is unpredictable.  Many implementations report the last name/value pair only.  Other implementations report an error or fail to parse the object, and some implementations report all of the name/value pairs, including duplicates.

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

In the above exmaple, the canonicalization step MAY lead to the signature validating, but the client parsing the `role: "admin"` field instead.

### 1.2.1 Remedies

Since canonicalizable data MUST be round-trippable by definition, one solution is to round-trip the data at the application layer: encode the data to binary, validate the signature, decode the validated binary, and use that at the application layer. This is a very manual process, and may require replacing existing in-memory data, which can have perfromance and memory lifetime implications.

Another approach is to sign CIDs. Being a hash, it is impractical to tamper with CIDs. However, passing CIDs around is often inconvenient, as it may require an additional network round trip and mixes concerns of signatures with data layout. If the payload is ever inlined to replace the CID, the earlier canonicalization concerns recur. In terms of data overhead, this is the same as including the inlined HMAC in the payload (below).

The solutiom in Varsig is to include an integrity hash with the signature. This increases the signature length; if a short signature scheme like EdDSA is used, this can be a substancial by percentage.

# 2 Integrity Hash

As 

## 2.1 Raw (Noncanonicalized) Data

If raw data is used, then an integrity hash is not required






FIXME if an EIP-191 or FIDO encolope is used, it is strongly recommended that the content be a CID.

# 2 Format


Signing binary data does not require including a separate hash, because it's not canonicailzated

## 2.1 Byte Segments

```xml
<multibase_prefix><varint 0xd0><varint multicodec_key_prefix><varint multicodec_hash_prefix><varint multicodec_hash_length><varint multicodec_prefix><varint raw_hash><varint raw_signature>
```

## 2.2 IPLD Schema

``` ipldsch
type TaggedVarsig union {
  | Varsig "d0"
} representation bytesprefix

type Varsig struct {
  -- Signature type
  keyPrefix    Multicodec.PublicKey -- Public key type for signature 
  hashPrefix   Multicodec.Multihash -- Hash prefix
  digestLength Integer
  
  -- Content Metadata
  contentEnc Multicodec -- Content multicodec prefix

  -- Crypto
  rawHash    Bytes -- Integrity hash, MAY be 0 if contentEnc is 0x00
  rawSig     Bytes -- Raw signature bytes
} representation stringjoin {
  join ""
}
```

* Signatures for IPLD SHOULD be over the CID
* Signing raw data for a non-IPLD use case is harder:
  * Need to sign the non-ipld raw bytes
  * Integrity hash helps
  * MUST check the 
  
FIXME put EIP-191 and EIP-712 paylaods on the multicodec table diretcly

# 7 Prior Art

# 8 Further Reading

* https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/
* https://latacora.micro.blog/2019/07/24/how-not-to.html
* https://www.blackhat.com/presentations/bh-usa-07/Hill/Whitepaper/bh-usa-07-hill-WP.pdf

# 8 Acknowledgements

Many thanks to [Quinn Wilton](https://github.com/QuinnWilton) for her many recommendations and stories of times she's used JSON signing exploits in [CTF](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) competitons.
