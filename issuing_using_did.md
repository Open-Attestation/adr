###### Authors:

Raymond Yeh

# Identify Issuer & Issuing Document using DID

## Goal

As a new issuer, I would like to issue verifiable documents without using ethers to reduce cost, but yet being able to be identified via DNS or DID.
As an existing issuer of verifiable document and transferable records, I would like to have my identity bound to a DID instead of DNS only.

The problems describe above have two facet:

1. How might we identify issuers of document via DID, especially if they are using issuing transferable records?
2. How might an issuer who is identified with DID be able to issue verifiable document directly by signing on it?
3. How might we revoke a document issued directly with DID?

## Approach

This document forms an understanding of the guiding principles which leads to implementation detail at the end.

## Guiding Principles

1. The solution should be guided by W3C VC standards as far as possible
2. The solution should allow a less expensive way to issue verifiable document
3. The solution should allow revocation of said document
4. The solution should extend to DID
5. The solution should leverage on the scale of DID for maximum reach

## Overview

![Overview](./assets/issuing_using_did/overview.png)

The proposed solution is to allow DID to be used a terminal identifier in OA viewers.

![Extending terminal identifiers](./assets/issuing_using_did/terminal_identifiers.png)

The approach will extend the current approach of using only DNS as the terminal identifier and also allow for DID's. While certain identifiers such as ethereum addresses may not be human-friendly, they are still useful for machines which are able to process them. This allows anyone with DID to be able to associate it with their issued documents.

Optionally, one may also associate their DID with a domain name to allow both the DID and the domain name to be recognized as the issuer.

## Technical Details

For simplicity sake, we will use a DID document from the universal resolver as shown in the implementation details discussed:

```json
{
  "@context": "https://w3id.org/did/v1",
  "id": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6",
  "authentication": [
    {
      "type": "Secp256k1SignatureAuthentication2018",
      "publicKey": ["did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6#owner"]
    }
  ],
  "publicKey": [
    {
      "id": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6#owner",
      "type": "Secp256k1VerificationKey2018",
      "ethereumAddress": "0xe6fe788d8ca214a080b0f6ac7f48480b2aefa9a6",
      "owner": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6"
    },
    {
      "id": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6#delegate-1",
      "type": "Secp256k1VerificationKey2018",
      "owner": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6",
      "publicKeyHex": "0295dda1dca7f80e308ef60155ddeac00e46b797fd40ef407f422e88d2467a27eb"
    }
  ]
}
```

### For documents that are signed directly

Assuming that an issuer with DID has the private key to one of the public key in the DID document, we can simply signal the intent to issue a given document by signing the `merkleRoot` of the document itself.

The issuer will simple append this proof into the root level of the document as such. The proof structure resembles the existing proof block in [w3c vc's structure](https://www.w3.org/TR/vc-data-model/#proofs-signatures).

Also, to signal if the document may or may not be revoked, a compulsory key `revocation.type` must always be present. It may take on multiple types of values such as:

- `NONE`
- `REVOCATION_STORE`
- `SIMPLE_REVOCATION_LIST`
- `REVOCATION_ENDPOINT`
- etc

A `revocation.type: "NONE"` must exist to ensure that the an end user did not obfuscate the revocation information to prevent his document from being revoked.

```json
{
  "schema": "tradetrust/v1.0",
  "data": {
    "issuers": [
      {
        "name": "TradeTrust Demo",
        "id": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6",
        "revocation": {
          "type": "REVOCATION_STORE",
          "location": "0xabcd...1234"
        },
        "identityProof": {
          "type": "DNS-DID",
          "location": "example.com",
          "key": "did:ethr:0x6813Eb9362372EEF6200f3b1dbC3f819671cBA69#controller"
        }
      }
    ]
  },
  "proof": [
    {
      "type": "OpenAttestationSignature2018",
      "created": "2020-10-05T09:05:35.171Z",
      "proofPurpose": "assertionMethod",
      "verificationMethod": "did:ethr:0x6813Eb9362372EEF6200f3b1dbC3f819671cBA69#controller",
      "signature": "SIGNED-MERKLE-ROOT"
    }
  ]
}
```

## New types of documents

New types of verifiable documents can now be created, using different verification method, depending on the top level identifier, issuance and revocation method.

Some examples are:

1. Top level identifier using DNS, document signed with key in DID
1. Top level identifier using DID, document issued & revoked with document store
1. Top level identifier using DID, document signed with key in DID
1. etc

[DNS-DID Sample](https://gist.github.com/yehjxraymond/a27ca5413673d0783c107c8d171cf1c9)

[DNS Sample](https://gist.github.com/yehjxraymond/952abe6e8d25a8263b804468785e0844)

## Examples

Below are some examples of issued documents. Note that the salt and types have been removed in the `data` field for legibility.

### Issuing Directly with DID signature, identified by DNS:

The `DNS-DID` type will be slightly different from the `DNS-TXT` method (which may be better named now to `DNS-CONTRACT`?). It does the job of:

1. Making the claim that the DID belong to a domain
2. Allowing the verifier to check against the domain txt record for validity of the claim
3. Allow the `proof` block to make a claim later that it is signed by this DID, and hence the domain

\_ Note that when the identityProof type is `DNS-DID`, we MUST check fo the existence of the revocation key

```json
{
  "schema": "tradetrust/v1.0",
  "data": {
    "issuers": [
      {
        "id": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6",
        "name": "TradeTrust Demo",
        "revocation": {
          "type": "REVOCATION_STORE",
          "location": "0xabcd...1234"
        },
        "identityProof": {
          "type": "DNS-DID",
          "key": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6#owner",
          "location": "demo-tradetrust.openattestation.com"
        }
      }
    ]
  },
  "proof": [
    {
      "type": "OpenAttestationSignature2018",
      "created": "2020-10-05T09:05:35.171Z",
      "proofPurpose": "assertionMethod",
      "verificationMethod": "did:ethr:0x6813Eb9362372EEF6200f3b1dbC3f819671cBA69#controller",
      "signature": "SIGNED-MERKLE-ROOT"
    }
  ]
}
```

### Issued via Document Store, identified via DID

The `DID-CONTRACT` type uses DID as the top level identifier. It does the job of:

1. Making a claim that a document store is associated with a DID, alongside which key it is
2. Allowing the verifier to check that the claim is legit by verifying the signature & that the public key belong to the DID

```json
{
  "schema": "tradetrust/v1.0",
  "data": {
    "issuers": [
      {
        "id": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6",
        "name": "TradeTrust Demo",
        "documentStore": "0x6d71da10Ae0e5B73d0565E2De46741231Eb247C7",
        "identityProof": {
          "type": "DID-CONTRACT",
          "signature": "<SIGNED-DOCUMENT-STORE>",
          "key": "did:ethr:0xE6Fe788d8ca214A080b0f6aC7F48480b2AEfa9a6#owner"
        }
      }
    ]
  },
  "privacy": {
    "obfuscatedData": []
  },
  "signature": {
    "type": "SHA3MerkleProof",
    "targetHash": "61dc9186345e05cc2ae53dc72af880a3b66e2fa7983feaa6254d1518540de50a",
    "proof": [],
    "merkleRoot": "61dc9186345e05cc2ae53dc72af880a3b66e2fa7983feaa6254d1518540de50a"
  }
}
```

### Issued via direct signing, identified by DID

The `DID` type will simply allow for:

1. The document to claim that it has been issued by a DID (and the specific key)
2. The verifier to check that is has been signed by the key and the key belongs to the DID

```json
{
  "version": "https://schema.openattestation.com/2.0/schema.json",
  "data": {
    "issuers": [
      {
        "id": "did:ethr:0x6813Eb9362372EEF6200f3b1dbC3f819671cBA69",
        "name": "DEMO STORE",
        "revocation": {
          "type": "NONE"
        },
        "identityProof": {
          "type": "DID",
          "key": "did:ethr:0x6813Eb9362372EEF6200f3b1dbC3f819671cBA69#controller"
        }
      }
    ]
  },
  "signature": {
    "type": "SHA3MerkleProof",
    "targetHash": "0ff2bee4bc5e0cde00dbf4524ca5173724538e3bf46241ff653e07ae51141c4d",
    "proof": [],
    "merkleRoot": "0ff2bee4bc5e0cde00dbf4524ca5173724538e3bf46241ff653e07ae51141c4d"
  },
  "proof": [
    {
      "type": "OpenAttestationSignature2018",
      "created": "2020-10-05T09:05:35.171Z",
      "proofPurpose": "assertionMethod",
      "verificationMethod": "did:ethr:0x6813Eb9362372EEF6200f3b1dbC3f819671cBA69#controller",
      "signature": "0x6d0ff5c64b8230cdc471f38267495002f2c762acf7a80250599809ee32b4255377f1adcb56fb712dee66bfeb21be6b5d802f299aea1f1edca129e88e4c1742ce1c"
    }
  ]
}
```

## Implementation Progress

### Done

#### Upgrade @govtechsg/open-attestation

[#127](https://github.com/Open-Attestation/open-attestation/pull/127)

Schema has been updated to allow `DID` and `DNS-DID` type for `identityProof`.
Released as v4.0.0.

#### Upgrade @govtechsg/oa-verify

[#114](https://github.com/Open-Attestation/oa-verify/pull/114)

Added `OpenAttestationDidSignedDocumentStatus` fragment to verify document signed directly.
Added `OpenAttestationDnsDid` fragment to verify document with top level DNS identifier, but signing with DID
Added `OpenAttestationDidSignedDidIdentityProof` fragment to verify document with top level DID identifier, and signing with DID
Added all except `OpenAttestationDidSignedDidIdentityProof` into the default set of verifiers.
Released as v5.1.0

#### Upgrade @govtechsg/open-attestation-cli

[#93](https://github.com/Open-Attestation/open-attestation-cli/pull/93)

Added command to sign wrapped documents with DID
Released as v1.29.0

### In Process

#### Upgrade TradeTrust website to support new verification

[#249](https://github.com/TradeTrust/tradetrust-website/pull/249)

Allow documents signed with `DNS-DID` to be verified
### Revocation method: Revocation Store

Implement the schema, oa-verify method to allow a document store to be used as a revocation store for documents which can be revoked, instead of just using `NONE` in the `revocation.type`.

As of now (180221), users can indicate `REVOCATION_STORE` in their documents for `revocation.type`. With a corresponding `revocation.location` documentStore address, this will allows users to check if their deployed `documentStore` has revoked any documents. 

### To Be Done

#### Individual document wrapping in CLI

Allow documents to be wrapped individually instead of batching it up. This prevent accidental issuance when two documents share the same merkle root and the proof is copied over from one document to another when one document has not been appended the proof.

#### IO Concurrency in CLI

Optimize IO concurrency in CLI by allow a limited number of threads to run concurrently. Current implementation does maximum parallelisation using `Promise.all`.

#### Counterfactually deploying document store as revocation store

This allow a user to "reserve" an address for the revocation store in the future but not perform the action now. The user will still be able to use the address during the document creation phase.

This could be done in the CLI to compute the address of the conterfactually deployed contract address.

#### DID-CONTRACT

Allow documents issued via document store or token registry to use DID as top level identifier. This should probably be done when DID becomes more widely accepted compared to DNS.
