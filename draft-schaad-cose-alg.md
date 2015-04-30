---
stand_alone: true
ipr: trust200902
docname: draft-schaad-cose-alg-latest
cat: info
pi:
  toc: 'yes'
  symrefs: 'yes'
  sortrefs: 'yes'
  comments: 'yes'
title: CBORE Algorithm Details
area: Security
author:
- ins: J. Schaad
  name: Jim Schaad
  org: August Cellars
  email: ietf@augustcellars.com
normative:
  I-D.greevenbosch-appsawg-cbor-cddl:
  I-D.schaad-cose:
  RFC5869:
informative:
  RFC2119:
  RFC7159:
  RFC7049:
  RFC7253:
  JWA:
   title: 'JWA'
   author:
   - ins: M. Jones
     org: Microsoft
   date: ''
  NIST-800-56A:
    title: 'NIST 800.56Ar2'
    author: 
    - ins: NIST
      org: U.S. National Institute of Standards and Technology
    date: ''

--- abstract

Make some general comments on how algorithms can work. 
Some of these things are just to make changes from the way JOSE works to be more CBOR like.

--- middle

# Introduction

The JOSE working group produced a set of documents that defined how to perform 
encryption, signatures and message authenticiation (MAC) operations for
JavaScript Object Notation (JSON) documents and then to encde the results using
the JSON format {{RFC7159}}.
The COSE working group has adapted this work to be more integrated with the Concise Binary Object Representation (CBOR) {{RFC7049}} document format.
The message formats have been documented in {{I-D.schaad-cose}} while this document contains the changes to the the algorithms to work better in the CBOR world.

This document additionally defines some new algorithms that are commonly used in the constrained device world.

# Key Derivation Function Primitives

The JOSE specifications all use a Key Derivation Fucntion (KDF) based on the Concat functionality from {{NIST-800-56A}}.  A new KDF primative is defined in this document to take its place.

## HKDF Primative Definition

The IETF is generally moving toward the use of the HMAC-based Extract-and-Expand Key Derivation Function (HKDF) defined in {{RFC5869}}.
The inputs to the HKDF function consist of a salt, the input keying material (or shared-secret), a hash function, an output length and context information.

The salt is optional to be provided.  It MUST be provided if the input keying material is used multiple times (for example with static-static ECDH key agreement).  The salt is placed into an attribute with the name 'salt' and SHOULD be of the same length as the hash function output.

The context information is built using a structure similar to that defined in Section 5.8 of {{NIST-800-56A}}.  The context information is the OtherInfo structure defined below, this strucutre is encoded using CBOR.  The elements in the OtherInfo structure map to those used in both the NIST document and in Section 4.6.2 of {{JWA}}.

~~~~ CDDL

OtherInfo = [
    algorithmId: (tstr / int),
    partyUInfo: bstr,
    partyVInfo: bstr,
    SuppPubInfo: int,
    ? SuppPrivInfo: bstr
]

~~~~

The fields contain the following:

algorithmId
: contains the identifier of the algorithm for which the key is being derived.  This contains the same value that would be in the 'alg' parameter field.  This may be either a string or an integer.

partyUInfo
: contains information supplied by the sending party.  This information is transported in the 'apu' parameter field.  If the field is absent, then the field may be defined by the application or coded to a zero length binary value.

partyVInfo
: contains information supplied by the receiving party.  This information is transported in the 'apv' parameter field.  If the field is absent, then the field may be defined by the application or coded to a zero length binary value.  This field will normally not be set.

SuppPubInfo
: contains the key length of the value being derived.

SuppPrivInfo
: will usually not be present.  If this exists then it is defined by the application protocol and not transported as part of the message.

## IANA Considerations

This change has no IANA considerations to deal with.

## Security Considerations

HKDF can be used with algorithms other than HMAC as the expand step, but it is not recommended for the extract step.  As such it would be possible to use an algorithm such as AES-CBC for the expand state, but it does not have the necessary characteristics for the extract step.

The extract step can be omitted if the input keying material is uniformlly randon.  This is not generally the case for the result of a key agreement opertion or a human generated shared secret, but can be for a machine generated secret.

# Key Managment Algorithms
## Elliptical Curve Key Agreement 

### Ephemeral-Static ECDH

{{JWA}} defines a four different ECHD algorithms.  When used with the CBOR specificiation, these algorithms are unchanged except for the fact that HKDF as defined in Section 3 is used for the key derivation step.

### Static-Static ECHD

Static-Static ECHD (ECDH-SS) can be used as a form of origination validation.  When used in a mode where only one ECDH-SS is used as a recipient for a message, the sender can make an assumption that the message was created by the owner of the sender half of the static key agreement.  (This assumes that the sender knows that it did not send the message.)  This type of origination validation cannot be demonstrated to a third part as can be done with asymmetric signature algorithms.

The recipient's key is identified using the same header parameters defined in {{JWA}}.  The sender's key identification is placed in the new header parameter 'spk'.  

The 'spk' header parameter uses key structure defined in Section X {{I-D.schaad-cose}} contains a map.  The map may contain a key or a one or more of the key identification fields defined in {{JWA}} such as 'kid' or 'jku'.

When this algorithm is used, an 'apu' value MUST be supplied.  The value used MUST be unique for every pairing of sender and recipient keys.

ALTERNATE: We could potentially do this by using the salt field of the HKDF function instead.  This would allow for a kind of normal usage of apu, but would make a potential problem in the future if we switched to a SHA-3 direct KDF function.  It might still be worth while since one could use the HKDF function on a shared secret to do a key derivation step when doing a direct encryption or direct hash operation.

| text | int | KDF | Key Wrap |
| name | name |  | |
| ECDH-SS | - | HKDF + HMAC-256 | - |
| ECDH-SS-A128KW | - | HKDF + HMAC-SHA256 | AES-KW-128 |
| ECDH-SS-A256KW | - | HKDF + HMAC-SHA256 | AES-KW-256 |

## IANA Considerations

Put in the template for defining the header parameter 'spk'.

### Security Considerations

Reuse of the same two key pairs along with the same KDF parameters will lead to the same key being re-used.  This is generally considered to be bad practice.


## Additional Curves

Curve25519
Goldilocks

| text | int | KDF | KDF | Key Wrap |
| key | key | COSE | JOSE | |
| C25519-ES | - | HKDF | Concat | - |
| C25519-ES+A128KW | - | HKDF | Concat | AES KW 128-bit |
| C25519-ES+A256KW | - | HKDF | Concat | AES KW 256-bit |
| Goldi-ES | - | HDKF | Concat | - |
| Goldi-ES+A256KW | - | HKDF | Concat | AES KW 256-bit |


A new key type is defined:

kty = "EC1"
Mandatory elements:

x = Public key 


## Direct key with KDF

By combining a KDF function with a preshared secret, it is possible to extend the life of a shared secret as it does not ever get directly used.

This mode can be used by keeping a counter that is used for the salt value in the HKDF function.  Each entity can keep it's own counter if the protocol specifies fixed values of 'apu' and 'apv' to be used .  For example one may be designated as the 'client' and one as the 'server'.  This will ensure that they will generate te same keys.  The counter can either be transmitted as part of the message or could be implicitly transmitted and both sides keep track of the other sides view of the counter as well.

# Content Encryption Algorithms

## AES-CCM algorithm

The CORE people are currently use the AES-CCM algorithm as their prefered content encryption algorithm.  The reference for AES-CCM is {{RFC3610}}.  

| key | key | key | tag |
| name | id | length | length |
| AES-128-CCM-64 | - | 128 | 64 |


### IANA Considerations

### Security Considerations

There have been security analysis and they generally look good.  Most of the complaints are about efficiency not security.  (http://csrc.nist.gov/groups/ST/toolkit/BCM/documents/proposedmodes/ccm/ccm-ad1.pdf)  (http://csrc.nist.gov/groups/ST/toolkit/BCM/documents/comments/800-38_Series-Drafts/CCM/RW_CCM_comments.pdf)  OCB is not as nice since it requires both an encrypt and a decrypt engine.


## AES-OCB algorithm

This is the hot new content encryption algorithm {{RFC7253}}.

| key | key | key | tag |
| name | id | length | length |
| AES-128-OCB-64 | - | 128 | 64 |

Key lengths are same as for AES - 128, 196, 256.
Tag lengths are 64, 96, 128.
Length of nonce   0 <= |nonce| <= 120 bits

### Security Considerations

Nonce must be non-repeating.  Nonces are public and can be a counter.


## AES-CBC-MAC algorithm

### IANA Considerations

### Security Considerations

## ChaCha-Poly Algorithm

It is the new hot bulk encryption algorithm

algName = "ChaChaPoly"

# Message Authentication Algorithms

256-bit key
96-bit nonce
128-bit tag output


# Signature Algorithms

## Pintsov-Vanstone signature scheme with partial message recover (PVSSR)

| name | id | curve | size of n | size of added data | Hash | Enc |
| PVSRR-256-n-m | - | P-256 | n | m | SHA-256 | AES-CTR |


### Security Considerations

Pointer to security proof is http://grouper.ieee.org/groups/1363/Research/contributions/PVSigSec.pdf.  I have a problem understanding this as it talks about redundancy bytes in terms of adding addition information to the message part to be encrypted.  The security of the scheme is based on the amount of redundancy in the message.

Security is based on the redundancy in n.  m allows for added redundancy to be added.  
I don't believe that it is going to be easy for people to understand what they need to do to get the correct level of security. 
As an example, if the text ends in binary data which is really random, the redundancy is high.
If the text ends in a fixed type string (think "Launch ### missiles") then the redundancy could be as low as a couple of bits.
This type of analysis could easily be beyond the capabilities of most protocol designers.

Worry about the requirement that the random chosen must be unique - not stated in the proofs?  Can it be deterministic?  

# IANA Considerations

There are IANA considerations

# Security Considerations

There are security considerations.

---- back

