# Appendix: Common Varsig Headers

Below are a few common signature headers and their fields. These are all given as the parts following the Varsig prefix and version.

## RSA

[RSASSA-PKCS #1 v1.5] signatures MUST include the following segments:

``` abnf
rsa-varsig = rsa-prefix rsa-hash-algorithm rsa-key-length
rsa-prefix = %x1205 ; RSASSA-PKCS #1 v1.5
rsa-hash-algorithm = unsigned-varint
rsa-key-length = unsigned-varint
```

### Example: RS256

| Segment              | Hexadecimal | Unsigned Varint | Comment                                 |
|----------------------|-------------|-----------------|-----------------------------------------|
| `rsa-prefix`     | `0x1205`    | `0x8524`        | RSASSA-PKCS #1 v1.5 [multicodec] prefix |
| `rsa-hash-algorithm` | `0x12`      | `0x12`          | SHA2-256 [multicodec] prefix            |
| `rsa-key-length`     | `varint`    | `varint`        | Length of public key in bytes           |

### Example: RS512

| Segment              | Hexadecimal | Unsigned Varint | Comment                                 |
|----------------------|-------------|-----------------|-----------------------------------------|
| `rsa-prefix`     | `0x1205`    | `0x8524`        | RSASSA-PKCS #1 v1.5 [multicodec] prefix |
| `rsa-hash-algorithm` | `0x13`      | `0x13`          | SHA2-512 [multicodec] prefix            |
| `rsa-key-length`     | `varint`    | `varint`        | Length of public key in bytes           |

## [EdDSA]

``` abnf
eddsa-varsig = eddsa-prefix eddsa-curve eddsa-hash-algorithm
eddsa-prefix = %xED
eddsa-curve = unsigned-varint
eddsa-hash-algorithm = unsigned-varint
```

### Example: Ed25519

| Segment                | Hexadecimal | Unsigned Varint | Comment                          |
|------------------------|-------------|-----------------|----------------------------------|
| `eddsa-prefix`         | `0xED`      | `0xED01`        | EdDSA prefix                     |
| `eddsa-curve`          | `0xED`      | `0xED01`        | edwards25519 [multicodec] prefix |
| `eddsa-hash-algorithm` | `0x13`      | `0x13`          | SHA2-512 [multicodec] prefix     |

### Example: Ed448

| Segment                | Hexadecimal | Unsigned Varint | Comment                        |
|------------------------|-------------|-----------------|--------------------------------|
| `eddsa-prefix`         | `0xED`      | `0xED01`        | EdDSA prefix                   |
| `eddsa-curve`          | `0x1203`    | `0x8324`        | edwards448 [multicodec] prefix |
| `eddsa-hash-algorithm` | `0x19`      | `0x19`          | SHAKE-256 [multicodec] prefix  |

## ECDSA

ECDSA defines a general mechanism over many elliptic curves. The ECDSA spec itself does not define a serialization. Unless otherwise specified, the raw[^raw] encoding MUST be used: take the two integers of known length ($r$ and $s$, length depends on the curve) and directly concatenate them ($r \Vert  s$).

[^raw]: Raw encoding is preferred by the WebCrypto API, JWS, compact JWT, FIDO2, COSE, and so on. 

``` abnf
ecdsa-varsig = ecdsa-prefix ecdsa-curve ecdsa-hash-algorithm
ecdsa-prefix = %xEC
ecdsa-curve = unsigned-varint
ecdsa-hash-algorithm = unsigned-varint
```

Here are a few examples encoded as varsig:

### Example: ES256

``` abnf
es256-varsig = ecdsa-prefix ecdsa-curve ecdsa-hash-algorithm
ecdsa-prefix = %xEC
ecdsa-curve = %x1200 ; P-256 multicodec prefix
ecdsa-hash-algorithm = %x12 ; SHA2-256
```

| Segment                | Hexadecimal    | Unsigned Varint | Comment                                          |
|------------------------|----------------|-----------------|--------------------------------------------------|
| `ecdsa-prefix`         | `0xEC`         | `0xEC01`        | ECDSA                                            |
| `ecdsa-curve`         | `0x1200`       | `0x8024`        | P-256 [multicodec] prefix                        |
| `ecdsa-hash-algorithm` | `0x12`         | `0x12`          | SHA2-256 [multicodec] prefix                     |

### Example: ES256K

``` abnf
es256k-varsig = ecdsa-prefix ecdsa-curve ecdsa-hash-algorithm
ecdsa-prefix = %xEC ; ECDSA
ecdsa-curve = %xe7 ; secp256k1 multicodec prefix
ecdsa-hash-algorithm = %x12 ; SHA2-256
```

| Segment                 | Hexadecimal    | Unsigned Varint | Comment                                          |
|-------------------------|----------------|-----------------|--------------------------------------------------|
| `ecdsa-prefix`          | `0xEC`         | `0xEC01`        | ECDSA                                            |
| `ecdsa-curve`  | `0xE7`         | `0xE701`        | secp256k1 [multicodec] prefix                    |
| `ecdsa-hash-algorithm` | `0x12`         | `0x12`          | SHA2-256 [multicodec] prefix                     |

### Example: ES512

``` abnf
es512-varsig = ecdsa-prefix ecdsa-curve ecdsa-hash-algorithm
ecdsa-prefix = %xEC ; ECDSA
ecdsa-curve = %x1202 ; P-521 multicodec prefix
ecdsa-hash-algorithm = %x13 ; SHA2-512
```

| Segment                | Hexadecimal | Unsigned Varint | Comment                      | 
|------------------------|-------------|-----------------|------------------------------|
| `ecdsa-prefix`         | `0xEC`      | `0xEC01`        | ECDSA                        |
| `ecdsa-curve`          | `0x1202`    | `0x8224`        | P-521 [multicodec] prefix    |
| `ecdsa-hash-algorithm` | `0x13`      | `0x13`          | SHA2-512 [multicodec] prefix |

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
[RS256]: https://datatracker.ietf.org/doc/html/rfc7518
[RSASSA-PKCS #1 v1.5]: https://www.rfc-editor.org/rfc/rfc2313
[Taxonomy of Attacks]: https://www.blackhat.com/presentations/bh-usa-07/Hill/Whitepaper/bh-usa-07-hill-WP.pdf
[`secp256k1`]: https://en.bitcoin.it/wiki/Secp256k1
[base64]: https://en.wikipedia.org/wiki/Base64
[canonicalization attacks]: https://soatok.blog/2021/07/30/canonicalization-attacks-against-macs-and-signatures/
[multibase]: https://github.com/multiformats/multibase
[multicodec]: https://github.com/multiformats/multicodec
[raw binary multicodec]: https://github.com/multiformats/multicodec/blob/master/table.csv#L40
[unsigned varint]: https://github.com/multiformats/unsigned-varint
