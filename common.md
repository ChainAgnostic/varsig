# Appendix: Common Varsig Headers

Below are a few common signature headers and their fields.

## RSA

[RSASSA-PKCS #1 v1.5] signatures MUST include the following segments:

``` abnf
rsa-varsig = rsa-varsig-header rsa-hash-algorithm signature-byte-length
rsa-prefix = %x1205 ; RSASSA-PKCS #1 v1.5
rsa-hash-algorithm = unsigned-varint
signature-byte-length = unsigned-varint
```

### Example: RS256

| Segment              | Hexadecimal | Unsigned Varint | Comment                                 |
|----------------------|-------------|-----------------|-----------------------------------------|
| `rsa-prefix`         | `0x1205`    | `0x8524`        | RSASSA-PKCS #1 v1.5 [multicodec] prefix |
| `rsa-hash-algorithm` | `0x12`      | `0x12`          | SHA2-256 [multicodec] prefix            |

### Example: RS512

| Segment              | Hexadecimal | Unsigned Varint | Comment                                 |
|----------------------|-------------|-----------------|-----------------------------------------|
| `rsa-prefix`         | `0x1205`    | `0x8524`        | RSASSA-PKCS #1 v1.5 [multicodec] prefix |
| `rsa-hash-algorithm` | `0x13`      | `0x13`          | SHA2-512 [multicodec] prefix            |

## [EdDSA]

``` abnf
eddsa-varsig = ed25519-prefix eddsa-curve eddsa-hash-algorithm
eddsa-prefix = %xED
eddsa-curve = unsigned-varint
eddsa-hash-algorithm = unsigned-varint
```

### Example: Ed25519

| Segment                | Hexadecimal | Unsigned Varint | Comment                        |              |
|------------------------|-------------|-----------------|--------------------------------|--------------|
| `eddsa-prefix`         | `0xED`      | `0xED01`        | EdDSA prefix                   |              |
| `eddsa-curve`          | `0xEC`      | `0xEC01`        | Curve25519 [multicodec] prefix | DOUBLE CHECK |
| `eddsa-hash-algorithm` | `0x13`      | `0x13`          | SHA2-512 [multicodec] prefix   |              |

### Example: Ed448

0x1203

| Segment                | Hexadecimal | Unsigned Varint | Comment                       |
|------------------------|-------------|-----------------|-------------------------------|
| `eddsa-prefix`         | `0xED`      | `0xED01`        | EdDSA prefix                  |
| `eddsa-curve`          | `0x1203`    | `0x8324`        | Curve448 [multicodec] prefix  |
| `eddsa-hash-algorithm` | `0x19`      | `0x19`          | SHAKE-256 [multicodec] prefix |

## ECDSA

ECDSA defines a general mechanism over many elliptic curves. 

``` abnf
ecdsa-varsig = ecdsa-varsig-header ecdsa-hash-algorithm encoding-info sig-bytes

ecdsa-varsig-header = unsigned-varint
ecdsa-hash-algorithm = unsigned-varint
encoding-info = 1*unsigned-varint
sig-bytes = *OCTET
```

Here are a few examples encoded as varsig:

### Example: ES256

``` abnf
es256-varsig = es256-varsig-header es256-hash-algorithm encoding-info sig-bytes
es256-prefix = %x1200 ; P-256 multicodec prefix
es256-hash-algorithm = %x12 ; SHA2-256
```

| Segment                | Hexadecimal | Unsigned Varint | Comment                      |
|------------------------|-------------|-----------------|------------------------------|
| `ecdsa-prefix`         | `0xEC`      | `0xEC01`        | ECDSA                        |
| `es256-prefix`         | `0x1200`    | `0x8024`        | P-256 [multicodec] prefix    |
| `es256-hash-algorithm` | `0x12`      | `0x12`          | SHA2-256 [multicodec] prefix |

### Example: ES256K

``` abnf
es256k-varsig = es256k-varsig-header es256k-hash-algorithm encoding-info sig-bytes

es256k-varsig-header = %xe7 ; secp256k1 multicodec prefix
es256k-hash-algorithm = %x12 ; SHA2-256
encoding-info = 1*unsigned-varint
sig-bytes = 64(OCTET)
```

| Segment                 | Hexadecimal | Unsigned Varint | Comment                       |
|-------------------------|-------------|-----------------|-------------------------------|
| `ecdsa-prefix`          | `0xEC`      | `0xEC01`        | ECDSA                         |
| `es256k-varsig-header`  | `0xE7`      | `0xE701`        | secp256k1 [multicodec] prefix |
| `es256k-hash-algorithm` | `0x12`      | `0x12`          | SHA2-256 [multicodec] prefix  |

### Example: ES512

``` abnf
es512-varsig = es512-varsig-header es512-hash-algorithm encoding-info sig-bytes

es512-varsig-header = %x1202 ; P-521 multicodec prefix
es512-hash-algorithm = %x13 ; SHA2-512
encoding-info = 1*unsigned-varint
sig-bytes = 128(OCTET)
```

| Segment                | Hexadecimal | Unsigned Varint | Comment                      | 
|------------------------|-------------|-----------------|------------------------------|
| `ecdsa-prefix`         | `0xEC`      | `0xEC01`        | ECDSA                        |
| `es512-varsig-header`  | `0x1202`    | `0x8224`        | P-521 [multicodec] prefix    |
| `es512-hash-algorithm` | `0x13`      | `0x13`          | SHA2-512 [multicodec] prefix |

<!-- Internal Links -->

[Header]: #header
[Signature]: #signature
[Common Signature Algorithms]: #common-signature-algorithms

<!-- External Links -->

[CAR]: https://ipld.io/specs/transport/car/
[CID]: https://docs.ipfs.tech/concepts/content-addressing/
[DAG-JSON]: https://ipld.io/specs/codecs/dag-json/spec/
[DKIM]: https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail
[EIP-191]: https://eips.ethereum.org/EIPS/eip-191
[EdDSA]: https://datatracker.ietf.org/doc/html/rfc8032
[How (not) to sign a JSON object]: https://latacora.micro.blog/2019/07/24/how-not-to.html
[IPLD Data Model]: https://ipld.io/docs/data-model/kinds/
[IPLD]: https://ipld.io/docs/
[JWT]: https://www.rfc-editor.org/rfc/rfc7519
[Multicodec]: https://github.com/multiformats/multicodec
[Multiformats]: https://multiformats.io
[PKI Layer Cake]: https://link.springer.com/chapter/10.1007/978-3-642-14577-3_22
[Parse Don't Validate]: https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/
[RFC 2119]: https://datatracker.ietf.org/doc/html/rfc2119
[RFC 7519]: https://www.rfc-editor.org/rfc/rfc7519
[RFC 8259]: https://www.rfc-editor.org/rfc/rfc8259#page-10
[RSASSA-PKCS #1 v1.5]: https://www.rfc-editor.org/rfc/rfc2313
[Taxonomy of Attacks]: https://www.blackhat.com/presentations/bh-usa-07/Hill/Whitepaper/bh-usa-07-hill-WP.pdf
[`secp256k1`]: https://en.bitcoin.it/wiki/Secp256k1
[base64]: https://en.wikipedia.org/wiki/Base64
[canonicalization attacks]: https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/
[multibase]: https://github.com/multiformats/multibase
[multicodec]: https://github.com/multiformats/multicodec
[raw binary multicodec]: https://github.com/multiformats/multicodec/blob/master/table.csv#L40
[unsigned varint]: https://github.com/multiformats/unsigned-varint

