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

# 0 Abstract

Varsig is a [multiformat](https://multiformats.io) for describing signatures over IPLD data and other canonicalized formats.

# 1 Introduction



# 2 Format

```
0xd0<varint sigAlg multicodec_key_prefix><varint hashAlg multicodec_hash_prefix><varint contentEnc multicodec_prefix><varint rawSig bytes>
```

``` ipldsch
type TaggedVarsig union {
  | Varsig "0xd0"
} representation bytesprefix

type Varsig struct {
  sigAlg         Multicodec.Key       -- Public key type for signature 
  hashAlg        Multicodec.Multihash -- Hash function used by signature
  contentEnc     Multicodec           -- Payload encoding
  rawSig         Bytes                -- Raw signature bytes
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

# 8 Acknowledgements

Many thanks to [Quinn Wilton](https://github.com/QuinnWilton) for her many recommendations and stories of times she's used JSON signing exploits in [CTF](https://en.wikipedia.org/wiki/Capture_the_flag_(cybersecurity)) competitons.
