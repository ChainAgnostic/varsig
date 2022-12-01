# Varsig Specification v0.1.0

## Editors

* [Irakli Gozalishvili](https://github.com/Gozala), [DAG House](https://dag.house/)
* [Brooklyn Zelenka](https://github.com/expede/), [Fission](https://fission.codes/)

## Authors

* [Irakli Gozalishvili](https://github.com/Gozala), [DAG House](https://dag.house/)
* [Joel Thorstensson](https://github.com/oed), [3Box Labs](https://3boxlabs.com/)
* [Brooklyn Zelenka](https://github.com/expede/), [Fission](https://fission.codes/)

# 0 Abstract

Varsig is a [multiformat](https://multiformats.io) for describing signatures over IPLD data and other canonicalized formats.

## Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

# 1 Introduction



# 2 Format

```
<varint sig alg><varint multihash integrity hash><enc varint><integ varint><rawSig varint>
```

``` ipldsch
type Varsig struct {
  header        "0xd0"
  SigComponents Bytes
} representation stringjoin

type SigComponents struct {
  sigAlg        SigAlg     -- Signature algorithm bytesprefix
  enc           Multicodec -- Payload encoding
  hash nullable Multihash  -- Integrity Hash varint
  rawSig        Bytes      -- Raw signature bytes
} representation stringjoin

type SigAlg union {
  -- Algorithms here are expected to be valid "varsig" multiformat codes.
  | NonStandard "0x00"
  | ES256K      "0xe7" -- secp256k1
  | BLS12381G1  "0xea" -- NB: G1 is a signature for BLS-12381 G2
  | BLS12381G2  "0xeb" -- NB: G2 is a signature for BLS-12381 G1
  | EdDSA       "0xed"
  | ES256       "0x1200"
  | ES384       "0x1201"
  | ES512       "0x1202"
  | RS256       "0x1205"
} representation bytesprefix

type Canonicalization union {
  | Raw    "0x00"
  | DAG_JSON
  | DAG_CBOR
  | DAG_PB
  | EIP191 "0xE191"
  | EIP712 "0xE712"
}

type EIP191v1 struct {
  "EIP"

}

type EIP191v2


0x19 <0x00> <intended validator address> <data to sign>
0x19 <0x45 (E)> <thereum Signed Message:\n" + len(message)> <data to sign>

type EIP712 struct {

}
```

# 7 Prior Art

# 8 Acknowledgements
