# Abstract:
This document describes a different approach to the current verifying mechanisms in verifiable credentials.

# Introduction:
Currently there are three primary variants of verifiable credentials, all of them have at least one implementation in different stages of production.
They are:
1. JSON-LD family with LD-signatures or with BBS+ Signatures that enable Zero knowledge proofs (ZKPs)
2. JSON with JSON web signatures, in the form of a JSON Web Token (JWT)
3. ZKP with Camenisch-Lysyanskaya Signatures (ZKP-CL)

Each of these implementation have various pros and cons and this paper will not go into the details of them. Instead, we would refer to the readers to a paper [3] written by Kaliya young, which goes into the specific details of these various implementations.

Although contrary to what the above paper hopes to achieves, what we propose is another flavour of a verifiable credential that is expressed in JSON-LD using linked data proofs that leverage on Merkle hash trees that are less complex ZKP implementations and more pragmatic in practice [5][6]. 

# Design of the proof:
Before we get into the design of the proof, we need to establish a few definitions around our solution. In the current implementation usage for our solution in `OpenAttestation`, the **set of claims** within **our credential** is mostly pre-defined and are mostly tied to a single **subject**. [2]

We will first start with a minimal Verifiable credential, then talk about how we protect the integrity of this document, while discussing this, we will also touch upon selective disclosure as a consequence of our implementation and its limitations. Lastly, we will extend our solution to allow a more efficient way of verifying credentials in a batch using merkle trees.

We assume that the reader is familiar with basic cryptographic primitives and would not go into a deep discussion about them.

First of all, let's start with a very minimal credential that adheres to the VC data model specification.

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1"
  ],
  "id": "http://example.edu/credentials/58473",
  "type": [
    "VerifiableCredential",
    "AlumniCredential",
    "OpenAttestationCredential"
  ],
  "issuer": "https://example.edu/issuers/14",
  "issuanceDate": "2010-01-01T19:23:24Z",
  "credentialSubject": {
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
    "alumniOf": "Example University"
  },
  "openAttestationMetadata": {
    "template": {
      "name": "EXAMPLE_RENDERER",
      "type": "EMBEDDED_RENDERER",
      "url": "https://renderer.openattestation.com/"
    },
    "proof": {
      "type": "OpenAttestationProofMethod",
      "method": "DOCUMENT_STORE",
      "value": "0xED2E50434Ac3623bAD763a35213DAD79b43208E4"
    },
    "identityProof": { "identifier": "some.example", "type": "DNS-TXT" }
  }
}

```

The anatomy of the credentials is as above, but we will focus on the `credentialSubject` object as its values are the **set of claims** about the **subject**.

For every property, nested or otherwise, we will first generate a secure random string (also known as a salt). This is to prevent a rainbow table attack [7]. While traversing the key value pairs, we would also keep a reference to the path of the original structure of the credential subject.  After all the processing, we would flatten the values in an array of objects with the key value pairs of `value`, indicating the secure random string, and `path`, indicating the reference of the path of the original document.

The code sample below shows the salted credential subject.

```typescript
const saltedCredentialSubject = [
  {
    value: "d775b5339c6b0adef3a2b8aaf33886b01759880ccad4e1f6cd4e44c18efadaca",
    path: "credentialSubject.id",
  },
  {
    value: "678465d280b40c984013af128913d30a4da311d5cfa75d4621efaa7921e1761a",
    path: "credentialSubject.alumniOf",
  },
];
```

To further compact this output, we will first stringify the output from the salts, and then encode the stringified JSON in base64. This would be then inserted into the proof object as specified by the VC data model with the key of `salts`.

This would be the encoded salts value of the entire credential:

```
W3sidmFsdWUiOiI1MGMxM2U1MmQ0N2FmOWYxMjQ5N2U3ZWUxYmFhM2UyYmEyMWU0ODAxYTExYTEwOGU2OTFiYTI4NWNhOWU2ZWUzIiwicGF0aCI6IkBjb250ZXh0WzBdIn0seyJ2YWx1ZSI6ImM3NDRhOWY4MTRhMjg0NmUxZTU2MDQ3YjM1NTM4NTU3MTYwMjg3OTM3YzQ2ZTM4MzE3ZTg4MDFlZDAyMDcyYjQiLCJwYXRoIjoiQGNvbnRleHRbMV0ifSx7InZhbHVlIjoiZmFmMWZjMDEzZDkxODYzM2ZkMzI5ZTA1ZDM4NmExZjU1MGE2ZmQ2NDQ4ZDQyMTI0N2VlMzJjMWFlOTFkY2IxNyIsInBhdGgiOiJpZCJ9LHsidmFsdWUiOiIwNTk5N2FhMDJmNTE3MjA0YTA2YzAyY2VmNWIzMTFiMzViMjljNDBhYmU1YTM4YTIwNTIyMzlkZmI2MjM2ZDFjIiwicGF0aCI6InR5cGVbMF0ifSx7InZhbHVlIjoiZTQ3NDVkMzZmMWI0ZTdjZjAwNmU5MWVmMjc0OTlhNzM1MDI1MmQ2ZGUwMGVmMzU5YzFiNWViZTIxMDY5YmMwYyIsInBhdGgiOiJ0eXBlWzFdIn0seyJ2YWx1ZSI6IjUwNjViZmQwY2UxZWQyZmRhYmM4ZTMxNTY4NWI3MWYzYjk2ZjFiYTI4NDlmMDk5NDQ0Yjk3ZWNjZjUzMjVmYjgiLCJwYXRoIjoiaXNzdWVyIn0seyJ2YWx1ZSI6ImNhMTYzNTRlZTZjODQ5NTI1YTE5Y2RhMzdiNjA0ODdjZTJlNGEwMTk3NTA5NGQxNDMyMjg2MjRlNjc3N2NmZGYiLCJwYXRoIjoiaXNzdWFuY2VEYXRlIn0seyJ2YWx1ZSI6Ijc4ODczNzJjMGU2ZWE4YjVmNGFkZmZhZTIzY2ZlNmYwMzQwZTYxOTk2NjlhNzI2OGZlZjdhM2IzZTI3NjE3MDYiLCJwYXRoIjoiY3JlZGVudGlhbFN1YmplY3QuaWQifSx7InZhbHVlIjoiYzhmNTU1MDYwNWJjOGI1ZGIyOTA5YWU3ZDhjMDcwN2U3Zjk2YzY0OWMzYTlhZTAwY2JkM2E4NTFmY2Q4NTViNCIsInBhdGgiOiJjcmVkZW50aWFsU3ViamVjdC5hbHVtbmlPZiJ9LHsidmFsdWUiOiIyZmNlMmY0MTQ3OTJlZDNjYmQ2NGJkZTkxNTA0MTIwNmNlMWUyOTllZjlhOWIyYzAwMGIyNjZmZjAzMzlhMTRlIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnRlbXBsYXRlLm5hbWUifSx7InZhbHVlIjoiODU2YjVkMDg2ODE4NjUzMWZkMTFhNzRjY2U3NDBiNjU2NTEzYjM3NzdjODk0NTY3NDhiODlmN2ZjNTQ4NTE4OCIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS50ZW1wbGF0ZS50eXBlIn0seyJ2YWx1ZSI6ImQ5MzE3YmIxYjVlMzk5ZDQyMTljYjc2YmFmZGVjOWNlMjBkOTU2ZDdjMGQ5YzJlZDQ3Y2E1ZmE5ZmI5NDJkNTgiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEudGVtcGxhdGUudXJsIn0seyJ2YWx1ZSI6ImNjMjE2MDAyNDVlMGQ4MzUwMDAzNjM1NjljMzhhY2I4YzRlZjFhNTg3NDc2YmZmMzM0ZmQyNWE2M2MzMWU2NTkiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEucHJvb2YudHlwZSJ9LHsidmFsdWUiOiIzOTRlNGI5ZTM4ODc4YjdjZmExZThlOWFjYWQyMjgzYzA4MjFhZWM0MTAyZDc4MzlhYjg5ZWRjY2Q4MzZmYjA5IiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnByb29mLm1ldGhvZCJ9LHsidmFsdWUiOiIyNzg3OTVlNmU3NjcxZmVkYWIwOTJhMzE2YjgyZjBhNDUwOTA3MWQ2MDk2NDc0MzE1ZjIwNjhhZTEzNDFiMmMxIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnByb29mLnZhbHVlIn0seyJ2YWx1ZSI6IjYyOTc2ZjJjNGJiNmJhOTRiNTQ0MDI0NzgwNTg3MDk3YjcxODdkOWY1MmM0ZjMwNTgyYjAwODE3NjdiODNiMDgiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEuaWRlbnRpdHlQcm9vZi5pZGVudGlmaWVyIn0seyJ2YWx1ZSI6ImNkMjhhNjU1MjgxZDMxYTNmN2EyZDNlNzYwZTZiZWQ3ZGY0MDJiM2M5YmQzYjJhNjFhMTViYzg1YjNmMzFhMTQiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEuaWRlbnRpdHlQcm9vZi50eXBlIn0seyJ2YWx1ZSI6ImU4OGZiYmM4MGMxODJmMDI2OTFhNmUzYmQ0YzkxYjEwMWIzN2E1YWQ0OGUyMmZkMmU1ZjBjYmU2ZmYxN2FjOTYiLCJwYXRoIjoiZ3JhZGVzWzBdIn0seyJ2YWx1ZSI6IjY5MWExODJmNGEzMTcwMTQwNWQ2NDExYzQ5YzA2Njc4MTQ5ZTdhYzRlZTAxMDk3MzcxY2Q0ZDU1YjYwMjcxYjYiLCJwYXRoIjoiZ3JhZGVzWzFdIn0seyJ2YWx1ZSI6Ijg5OTVlMjkyNTc2NDRhNjZiNGU3OGJlYTgzYzA0MzQ3YmNjYzUwNzNhNDA0ZDIzZTcwZGRkZjUzY2YyZTIwYzUiLCJwYXRoIjoiZ3JhZGVzWzJdIn0seyJ2YWx1ZSI6IjBjYmYzM2Y4Y2E3NjVlOGMxZTk1YTVhZWMwNGRlNWVkYTg5MzRiMGQ3NzMzMWQwMzhkZGQ3MmRlN2UxNTYwMWEiLCJwYXRoIjoiZ3JhZGVzWzNdIn0seyJ2YWx1ZSI6ImM1ZDIxMzQxNGVhMDVkNmMwOTNhZDM0MjMxZWU2ZGQ0M2JhNjg2ZjVhM2FlYTRmOTM3MTAwZTQ1NTRlZWQ1ZjUiLCJwYXRoIjoiZ3JhZGVzWzRdIn1d
```

From the salts generated, we will utilise a function to combine both the credential, the salts, and the obfuscated data (look at the selective disclosure section for more details) to produce the digest / or the proof signature of the credential.

`3511ac4badb521948b729ae935cefa1645d56582c8faf6a7f2509f0b4bcf2d99`

How does this function work internally? It first prepares an array of hashes from the data the holder wants to reveal. With reference to the path provided by the array of salts, it would take each individual value of the document, stringifies it, and finally hash it with a `Keccak SHA3` hash function.

```typescript
const hashedValues = [
  "a04a92ef2b8d62d40a4ce98d13e35b0e5c4eb68dca748b4c888c25b801b31f37",
  "5590b555d1b075894ec357bbcd542c3773b69962716aebd245dfe54a852b2c3e",
  "eb62c90f10070c5de1f36e603da4b705eeef6036665434ffb61f7df30a33fba0",
  "733611053fc6536b9dab1da7557e20f88f61b7a33519a14bf26787b1fa6c1d2a",
  "22196062d12b00a85cdfd369e8a6b394d012f5bd6b2a907bc53f2d729792a651",
  "b0e4419f5f117d01d26d91271998bcdcc68a4f7fe2bd13563c6833104d82f720",
  "45f02801352253d7eac8ebcba2b51ac8965a510102e2df8edbbf160c98b1e63b",
  "f67937efbecfb066ac0fc7973d9e49dbd92443603923b2794259b72f45167194",
  "004176499ade7be2ffa6d64edfbc373fe5a70bca90aad1bb8d52826a1b21e26a",
  "b744d7f8a09833f70aadccb82273d7568964209eb4313d8d591b92b87da7938c",
  "5d17a584954ebcfe0e3419e3dc81c71be258b6a37160cff008fcc90e619ba74c",
  "9534477eadfb0a5c5835e5a41b788158c734315da0f1f13a0b956f166a4cc98f",
  "0828304b0077227fd6e693db513e42b3e3c327489d6603abf710d82eb327614e",
  "3a980e75a52748bfa9ba842997d7e1f294df1dca8c4fe12db6117a644d8a03fb",
  "a5329021d8a0c4ac235370f50421bad71f30bc04da205e18972245f80698870c",
  "2e80f0be181a8d2b5ae813db6f3f5304795b8f300aac59aece566464a8a494dc",
  "7b6edf40d811952f4d1d66a12419d1e91f72170b104339e5152c550f697d0857",
];
```

If there are any obfuscated values, we will concatenate these hash values and sort them numerically (each hash corresponds to a unique number expressed in hexadecimal). We sort them so as to ensure determinism.

```typescript
const sortedHashValues = [
  "20f1714550a68f94ad55f6c553bcf6196a840c70d3c24a49ed11299d241c2dad",
  "2b99ab683570fb6e95a05f0310900e994afaeabb067511500b07a2be9d74a1c6",
  "2ebe6e64ecab156d706435fd42b9a5c046a3b8f6f6fdbcb3b6df32af2888639a",
  "38f9c60988f2bb52aa62b0a3c516776a942024023ae8aba308a2ae0237a3c12c",
  "3a482bd48775c90f39c378e11001ce04b61d98b92f3de67a78e71c117e59d044",
  "425c9b3800c204f4cdc1677fc6a3f993f3e0d866d93036a8c24ad94dfbf9bffa",
  "5a8efc4718dcb96e80cd1153ff5e0c17a08afbca41825377eaa3f6b1cef7c1cf",
  "64e59d5c456dc830b2cd546f37d420014df34dcaa8d970dea39d378220fa141e",
  "6c03cb29414c1af6312ccd00343134f3a6dd1fc0a6a0d9242f40daa454dcdf44",
  "6f1ffffcb536495a2b52aa33235a92037cd9982b3998014150d76270224c0823",
  "88488157146d5925c7d7c5bb93d92c973f6eed9709fe2479e30f332516d5ec7c",
  "9307347da695f7a8fe3229b68be2b4af0b1bf46d43e2521471927caae95a013b",
  "a909316f01c45b3e20d8d91d72f947d376a3cc3acb253553b6279e92d9be02b5",
  "bd7200d671bae68e0729810c66581f8e74b1f67cf7deb4b32bbc79a3ae70b695",
  "c614b6c0295688ea211d83d0e07ece10dd56d8b169e4380d3d2f7ed0104c7ed2",
  "d02bd32a3b46e62e06cb1cca103da472d38c915ca4af886e49768630751b1017",
  "e091e21aa75fec642b8422efc813149d25b15e07074c4f464b0a8160f6be47d5",
];
```

Finally we apply the `SHA3` hash function once more on the stringified sorted array of hashes.
Once the digest of the credential is obtained, we will then also append this digest in the proof object with the key of `targetHash`.

To actually ensure credential integrity, during the verification process, the same exact steps as listed above will be performed on the credential again to generate the digest of the credential. As long as none of the key value pairs have been modified, the final `targetHash` (signature) will be exactly the same from the original. If any one value changes, the `targetHash` would be radically different from the original.

For completeness, lets display the credential and what it looks like after processing it.

```json
{
  "version": "https://schema.openattestation.com/3.0/schema.json",
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1",
    "https://schemata.openattestation.com/com/openattestation/1.0/OpenAttestation.v3.json"
  ],
  "id": "http://example.edu/credentials/58473",
  "type": [
    "VerifiableCredential",
    "AlumniCredential",
    "OpenAttestationCredential"
  ],
  "issuer": "https://example.edu/issuers/14",
  "issuanceDate": "2010-01-01T19:23:24Z",
  "credentialSubject": {
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
    "alumniOf": "Example University"
  },
  "openAttestationMetadata": {
    "template": {
      "name": "EXAMPLE_RENDERER",
      "type": "EMBEDDED_RENDERER",
      "url": "https://renderer.openattestation.com/"
    },
    "proof": {
      "type": "OpenAttestationProofMethod",
      "method": "DOCUMENT_STORE",
      "value": "0xED2E50434Ac3623bAD763a35213DAD79b43208E4"
    },
    "identityProof": {
      "identifier": "some.example",
      "type": "DNS-TXT"
    }
  },
  "proof": {
    "type": "OpenAttestationMerkleProofSignature2018",
    "proofPurpose": "assertionMethod",
    "targetHash": "cd6569ac4927a28103f07dde4092b160a6c3613d4d35918faaee087029ad7eea",
    "proofs": [],
    "merkleRoot": "cd6569ac4927a28103f07dde4092b160a6c3613d4d35918faaee087029ad7eea",
    "salts": "W3sidmFsdWUiOiJiNjAzM2YyN2UwZmQ3ZWQ0MWQ5MjllOTk0MDdlNDVjODYwMTY4N2U4NDc3NzQ4NTU4MzEyZTM2MzE4YjI4YzBlIiwicGF0aCI6InZlcnNpb24ifSx7InZhbHVlIjoiY2Q1NTcwNGU4ZjExNDc3NTM1MTkzMmJlMGQxMGY0YWI1ZDZiOTFhYjc4ZWQxODcwN2MwNjE3ODJiMDE1ZDNkMyIsInBhdGgiOiJAY29udGV4dFswXSJ9LHsidmFsdWUiOiJiMjRmYjIyZDUzMWNhMjRkYmY0NGZmYTc0MTJiNDk5ZDljNjUzNTBmNzA3NTA2ZjBiOGMwOTdiMmFjMTU5M2U1IiwicGF0aCI6IkBjb250ZXh0WzFdIn0seyJ2YWx1ZSI6ImJhYTU0NjFkNDBlMTM4Y2RhYmU3ZDA1NTc1M2ZhNjUxZDIwMWYzNzA5Nzg2YmRiZjA3NTU5OWEzZjY2OGZjZmIiLCJwYXRoIjoiQGNvbnRleHRbMl0ifSx7InZhbHVlIjoiZjRkMDA2YWYwYzgzZjRjM2RiZjVkYWY2NTNlYTMxZDQ4YjIyMDYyM2VmY2FjNzkxMjQwNTUyNDdmNWQ5NDk1ZiIsInBhdGgiOiJpZCJ9LHsidmFsdWUiOiI3NWQ1MDgyNGM2ZGUzODEwN2Y0Y2NhMWM4YjYyODU0YmU5YzdlYWEwNjc3NDk4YmQyNzkwM2U0MjM4NWVjNTJlIiwicGF0aCI6InR5cGVbMF0ifSx7InZhbHVlIjoiZjRlNTI1NmU1MjRhYTdjNjkyOWYzZjZkMWVmMTM2ZGQ3NDVhZmQ0MWI5NDc4ZmNiYzJiYzJiOWI4OGU0ZTQzMCIsInBhdGgiOiJ0eXBlWzFdIn0seyJ2YWx1ZSI6IjA3OTZlOGRlZmVhZjcwZDMyNjYxZWZhNmMxYzk2MjFjNjg0YWQ0MDNlZmY5MDQxYTI5NDExOTQ0ZTU2ZDZhMDQiLCJwYXRoIjoidHlwZVsyXSJ9LHsidmFsdWUiOiI0MzEwNTRjODBlZGYyNWUyZTU5NzliZTM4ZTdmMWZkMTk5YWY1N2I2OWRmNmVlNzE0MGYzMTZiNTEyYWQ5MmRkIiwicGF0aCI6Imlzc3VlciJ9LHsidmFsdWUiOiJmOTE1ZmFkOWMxNDgxOWQ3NzQxZjNmMWEyMDg2YWJiMjY2M2JiYjMzNmI3N2FjMDUyMDY5MDI2YjgyNzE3NzIzIiwicGF0aCI6Imlzc3VhbmNlRGF0ZSJ9LHsidmFsdWUiOiI4M2MwYjVkOGYwZGNmNWIzNDAyNjQ1YTkyYzUzNTA2OTM1ZGVjNDBmOTA1MTBhMjQ2Nzk3Y2RkZmFmYTY3OTdlIiwicGF0aCI6ImNyZWRlbnRpYWxTdWJqZWN0LmlkIn0seyJ2YWx1ZSI6IjlmODI5NGY4Y2U0ZGNiYmE3NDQ5YjAwOGY4MGY3MjlmZTI3Yjk0OTUyNmRlNGE3MjhjOWNiMjNhNzRjODA1MDYiLCJwYXRoIjoiY3JlZGVudGlhbFN1YmplY3QuYWx1bW5pT2YifSx7InZhbHVlIjoiMjU0M2RlMTQ2MGQ5ODUxYmMyY2EzOGFlMDhlY2EzYTQ1ODM4MGQxNjkwYWM1Njg4NDc1NzdjMWRiM2VkZTE2YyIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS50ZW1wbGF0ZS5uYW1lIn0seyJ2YWx1ZSI6ImE4NWZlMWNjZjExZTI1YzZkYzQ1ZmQ4N2VlMDZiZDAxMDA5YjQwNGQwZmQxYzViMzk5ZmE4MTU4OTdkNTdiMTkiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEudGVtcGxhdGUudHlwZSJ9LHsidmFsdWUiOiIxMGUzY2YwMGZmNWRmOTBiZDMwNDZmZTZkNWE3ZmE0OTY5ODI1YjFkM2NkNjhiMDI2NzQwZjQzYjczZGUwNjI0IiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnRlbXBsYXRlLnVybCJ9LHsidmFsdWUiOiIzNzAzNzQwNDRjMDNkNTI5NGQ4MGU1NGFmNzNkNzY1NjQ2MTkwNmViNDk1MmNlYzkxOGRhNWE5YTFlZDEyNGM2IiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnByb29mLnR5cGUifSx7InZhbHVlIjoiMzExYjI4M2M5YTZlYjU1ZjI2N2MwNDIyZDlmZTk1ZjUzZGVjOTQ5ZjE1YzBlOTUwNGVmMTVhZThhY2VhMjllOSIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5wcm9vZi5tZXRob2QifSx7InZhbHVlIjoiNzg1MjU4OGU0N2YwZGNkMjYyZGU2MTVjNGM3MjllNjI0YmE0NmZiM2YyOTQyZDY2Zjk4ZDBjNTMzNDI0YjZhMiIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5wcm9vZi52YWx1ZSJ9LHsidmFsdWUiOiI3OTdjYmRkYTc5ZDA5ZDY2Y2NkODhjZDg2OTM4ZWUyODg3OTJkNDVjMTAxMWM4NjVhZmRkMjkzNjBhOTY5MzAyIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLmlkZW50aXR5UHJvb2YuaWRlbnRpZmllciJ9LHsidmFsdWUiOiIyOTI2MjdkMWI5MDM2M2VmYWM4MTZmNmZlNjE1Mzg1ZGY0NWRlNzA1ZTlhNDI0MGRmZWQ5M2E1ODU0ODdkYTc2IiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLmlkZW50aXR5UHJvb2YudHlwZSJ9XQ==",
    "privacy": {
      "obfuscated": []
    }
  }
}
```

## Selective disclosure:
Due to the way we compute our digest of the credential, this allows us to selectively obfuscate data that we do want others to know about. If we recall our sample credential, lets say if the holder decides to redact or obfuscate the field `credentialSubject.alumniOf`, after the stage of generating the salts and hash of individual key values, we would then remove that particular key value pair in the original document. We will append that particular hash of that field into the proof object `privacy` key, this would contain the array of hashes that we are redacting or obfuscating. 

Once processed, the credential will look like this.

```json
{
  "version": "https://schema.openattestation.com/3.0/schema.json",
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1",
    "https://schemata.openattestation.com/com/openattestation/1.0/OpenAttestation.v3.json"
  ],
  "id": "http://example.edu/credentials/58473",
  "type": [
    "VerifiableCredential",
    "AlumniCredential",
    "OpenAttestationCredential"
  ],
  "issuer": "https://example.edu/issuers/14",
  "issuanceDate": "2010-01-01T19:23:24Z",
  "credentialSubject": {
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21"
  },
  "openAttestationMetadata": {
    "template": {
      "name": "EXAMPLE_RENDERER",
      "type": "EMBEDDED_RENDERER",
      "url": "https://renderer.openattestation.com/"
    },
    "proof": {
      "type": "OpenAttestationProofMethod",
      "method": "DOCUMENT_STORE",
      "value": "0xED2E50434Ac3623bAD763a35213DAD79b43208E4"
    },
    "identityProof": {
      "identifier": "some.example",
      "type": "DNS-TXT"
    }
  },
  "proof": {
    "type": "OpenAttestationMerkleProofSignature2018",
    "proofPurpose": "assertionMethod",
    "targetHash": "704510b6b097aab7a0e3e76b2112174571c20439bcbc1373f1020d0221763405",
    "proofs": [],
    "merkleRoot": "704510b6b097aab7a0e3e76b2112174571c20439bcbc1373f1020d0221763405",
    "salts": "W3sidmFsdWUiOiJiZmFkMTYzNTAwMzdkZTcxM2U5NTc3YzhmMjkzZTY1NGZiMTI5ODczYWE1OTM2NmM0YzViYzc1ODJiNjc1YWQ4IiwicGF0aCI6InZlcnNpb24ifSx7InZhbHVlIjoiNWExNjY5Mzg5NGY3YzYxODU2M2MwZjljM2YxOTZiODk3YjFiNDg0NzRiZTE0NzM2MTQyYzUzNmJkOTI3MzNkMCIsInBhdGgiOiJAY29udGV4dFswXSJ9LHsidmFsdWUiOiI3MjM5MzRmMDA3YmE5NWRhMWM0Y2MxN2U4MTM2NTI3YjliNGZmZGMxNDhmMTFkZDlkODRjNzdkN2E4MWFkMDI2IiwicGF0aCI6IkBjb250ZXh0WzFdIn0seyJ2YWx1ZSI6ImRlNWU5MTg3NDY4NjA3YTM0OWE1N2QzOGQ4ODAzOGY5NWE4MDY3OTM1OTcwNjk5MDY5MzI0MTljMDViN2EyN2YiLCJwYXRoIjoiQGNvbnRleHRbMl0ifSx7InZhbHVlIjoiMDI0ZjExNWQ1M2NkNDZjZjVhZmZkYmJhZmUzMjhhNDNiMGZjODJhYTc2ZTIyYWMzNjcxMzVlMzkzMGFmODE2YSIsInBhdGgiOiJpZCJ9LHsidmFsdWUiOiIwNWU1YjE2NTc5NDFkZjdiZDA0YjQ5ZmJhNjZlNDYyM2FkOGY1MjEyMmY2Yjc1MjVjY2ZjNWMxNDQwNDM1NjYzIiwicGF0aCI6InR5cGVbMF0ifSx7InZhbHVlIjoiYzAxYmQ5ODhiMDRiZDgxMzhmYzRjMTE0MDI0ZmUwNjQ2ZGI3ZThiNmExNTc4NGY3YmMzYTNiMmRjYjU2MTU1ZCIsInBhdGgiOiJ0eXBlWzFdIn0seyJ2YWx1ZSI6IjQwYjI4MTEyYjI1MGU1ZWE3MDgwMjNiMzY4MDE0OGVkNWMyYjY3NDIxNzJiYjg4Y2RjNzdkNzkxZWJhN2JmNmMiLCJwYXRoIjoidHlwZVsyXSJ9LHsidmFsdWUiOiIwNGY4OTU5YjA0MDc1OTBlNDM0MmZkZjZkYzFlYThiM2Y0YThhODQ4NTA3ZDcxYWYwOTM1MmUzYzU5NTRhMzIyIiwicGF0aCI6Imlzc3VlciJ9LHsidmFsdWUiOiI0NDY3MTYzOTQ4OGM0MGU2YThjNDc3N2MyZGViNWUxMWJhNjFhZGE2ZmI5ZWY0MmVkN2Q5NTEyYjE5MzJiZGYwIiwicGF0aCI6Imlzc3VhbmNlRGF0ZSJ9LHsidmFsdWUiOiIwZGU3ZDU2YzFkZDVkMWNmZTZjZGFjYTQ5Njg1NWZlZGI0MzQ3N2Q2NDliMTU4MDJlNzc4Yjg0NzAyZTVmZWNhIiwicGF0aCI6ImNyZWRlbnRpYWxTdWJqZWN0LmlkIn0seyJ2YWx1ZSI6IjA1YjhhMmNhYjdlMDg5MzIzYTVkMmY3ZjEwZDhjNTkxNGQ3MzJiNzQ1MWNlYzEyYmRkYzE5NTM2MGJjMDUwMDciLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEudGVtcGxhdGUubmFtZSJ9LHsidmFsdWUiOiIyZTRmYWY5OGVjYjViYTQ5M2U1ZWZlOTkzZDk3ZTU4MDUwMzI5YzE1NDJiZWQwYjY0YmEyNjdkMDA3NTgzMWRmIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnRlbXBsYXRlLnR5cGUifSx7InZhbHVlIjoiYjU2ZTg5N2Q4MDkxNGEwNDEzM2NmNWE2ODY5ZTY0NzNmMDJkOTgyYTExODU0MGU4NDk2YzNhOTI0ZjJhMDJlOSIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS50ZW1wbGF0ZS51cmwifSx7InZhbHVlIjoiZjM0YWVmYTE0MjRhMzkzZGM5M2MwZTExOTU0ZmYxMDUxZDRiZjYzZjNlMTVlOGU4NjlkYjlhMmI2OTU4ODRhZiIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5wcm9vZi50eXBlIn0seyJ2YWx1ZSI6ImRlYjM3MzFkZjVhYjA0YjY1MGRiYzUwNmYxMTM0NzcxYjU0ZDA3NTM1Yjc1ZGZiMzA2YWFmYzliOGJlZDgxZmIiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEucHJvb2YubWV0aG9kIn0seyJ2YWx1ZSI6IjNlODJkYTQwNDk1YzMwMjgyMGZlY2E3NDM5ZGRlZWJkNjZhYWQ1MmVkNmY5NDhhMTMzMjJhMjIyNWU4NDgzMmQiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEucHJvb2YudmFsdWUifSx7InZhbHVlIjoiYWI2MDI5OTI3ZjQ1N2E4Njk3NDlkNjAwNDc0NjNjMWQyOTk0MDZjN2ZmODc5NDkyYjJkMzg5OTE4NzMyOTkzMCIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5pZGVudGl0eVByb29mLmlkZW50aWZpZXIifSx7InZhbHVlIjoiM2NkMGJiYjg0ZmFiZDA3NGYwOWI4YmMwYWIzODk3YzZjYzNkODdmMTkxNzcyMDM3OWNmMWRjMGJlN2FlMDBmNiIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5pZGVudGl0eVByb29mLnR5cGUifV0=",
    "privacy": {
      "obfuscated": [
        "06b0b34eb201e67f81e24f89ce1cdf2d280e1c1b11203b076a395a2d44e1dc27"
      ]
    }
  }
}

```
Notice that now the credential does not have the field `credentialSubject.alumniOf` present, but during verification, this hash would be added to the array of hashes, then sorted before it is used to calculate the digest of the credential to ensure document integrity (see the section on Design of Proof).

## Merkle Hash trees:
Although we prefaced our proposal with the term Merkle trees, we have yet to talk about anything related to merkle trees. For the uninformed, a merkle hash tree is constructed by first hashing the leaf nodes (usually some data), then hashing the hashes of the leaf nodes arranged in a binary tree structure [8]. The first node at the top of the binary tree is also known as the merkle root. This is an efficient way to compact multiple credentials into a hash representation so that an issuer could issue multiple credentials in only one transaction (either in a centralised server like a Certificate Authority, or something more decentralised like a blockchain ledger). 

From the current solution with the minimal credential, if the credential were to be processed in a batch of other credentials, we would take the `targetHash` of each individual credential and produce a Merkle Tree representation of the hashes. We would then take the merkle root of this tree and append it to the proof object section of the credential. Simultaneously, we would append the intermediate hashes used to construct the Merkle Tree into the `proofs` key value pair. Lastly, if we were to process only one credential, then both `merkleRoot` and `targetHash` would resolve to the same value.

At the time of verification, the verifier would not only check for the credential’s integrity, it would also check whether this batch of credentials (`merkleRoot`) is still valid at the time of the claim.

```json
{
  "version": "https://schema.openattestation.com/3.0/schema.json",
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://www.w3.org/2018/credentials/examples/v1",
    "https://schemata.openattestation.com/com/openattestation/1.0/OpenAttestation.v3.json"
  ],
  "id": "http://example.edu/credentials/58473",
  "type": [
    "VerifiableCredential",
    "AlumniCredential",
    "OpenAttestationCredential"
  ],
  "issuer": "https://example.edu/issuers/14",
  "issuanceDate": "2010-01-01T19:23:24Z",
  "credentialSubject": {
    "id": "did:example:ebfeb1f712ebc6f1c276e12ec21",
    "alumniOf": "Example University"
  },
  "openAttestationMetadata": {
    "template": {
      "name": "EXAMPLE_RENDERER",
      "type": "EMBEDDED_RENDERER",
      "url": "https://renderer.openattestation.com/"
    },
    "proof": {
      "type": "OpenAttestationProofMethod",
      "method": "DOCUMENT_STORE",
      "value": "0xED2E50434Ac3623bAD763a35213DAD79b43208E4"
    },
    "identityProof": {
      "identifier": "some.example",
      "type": "DNS-TXT"
    }
  },
  "proof": {
    "type": "OpenAttestationMerkleProofSignature2018",
    "proofPurpose": "assertionMethod",
    "targetHash": "d9336a9ca1ecbc3cd9c7a4245bda9d4994c86cd056a436ef120481eb14a3d101",
    "proofs": [
      "8b315ff6e14f40022b073d32aff47022d43e3f296b98de0055bd031709c3284b"
    ],
    "merkleRoot": "4f6339d030193e96c724b5f9520c716306ccb675ade5fa2d60ec1a8691687492",
    "salts": "W3sidmFsdWUiOiJjMzc2YzE2N2E2Yzk4MGMyMDcyZGFjNDZjY2RjZGZhNWUyMTc5YWVkYTM4ZmM0OWY0MTRiMTNjNGQyMzU3Mzk5IiwicGF0aCI6InZlcnNpb24ifSx7InZhbHVlIjoiYmY2NDUyOWJlMDQ4YjY1ZDVmMGNlY2ZmOTQwOWFjZjk4MmYwMjRmNTcxMjk1ZThmODc2NTdlZGUzZDAzMzlkZCIsInBhdGgiOiJAY29udGV4dFswXSJ9LHsidmFsdWUiOiI4OTAzZTFkYWM0YmMwZjI5MDYzNTk1Y2ZmODFmYjQ5NGVjZGVkYjllNmZmYjAxZTAzYTA2ZThiYWU0ZTc1NzlkIiwicGF0aCI6IkBjb250ZXh0WzFdIn0seyJ2YWx1ZSI6ImQ2YTMwMzIxNWIxNmRmMDA1MzI1OThmNDRkMTExN2QxNjJkMTY4NTMxYzE2OTdlMjU5ZTgxYjRhMWM0OThmMzMiLCJwYXRoIjoiQGNvbnRleHRbMl0ifSx7InZhbHVlIjoiM2Q4ZmFjYTkzMWE3ZTZjYTk2MjRmZjEyYzY2MDAxOWU5OGFjM2E2ZDU0ZmNjZmNhZGUxOTc2OGNiMjdlMTlmNyIsInBhdGgiOiJpZCJ9LHsidmFsdWUiOiJmZjk1ZTQ1NDIzYTRiYjhhNGEyMjg0MTRiMmJlMWMzNWQyYjdjNGFjODRkZGQwZWYzY2FkMDNmNjU3ZWEwODY1IiwicGF0aCI6InR5cGVbMF0ifSx7InZhbHVlIjoiNTEzYTdjMjY0N2ZkZmQ4M2VjNzc5ZjNkYzgyZGI3OWNmM2IyZjkyMzQ1NTQxZWE1OTQ5YThkYTRkMGJlODliOSIsInBhdGgiOiJ0eXBlWzFdIn0seyJ2YWx1ZSI6IjY2ZDgxM2UyMTAwMDU2MGI5NmZmZGMyNGU0MzJjNzRlM2RhNzYwNzY2ZGE2NjVjZWIzMmUyNjA1NjQ4NTRlNmIiLCJwYXRoIjoidHlwZVsyXSJ9LHsidmFsdWUiOiJiYTA1ZmM4OTIzMGI2NjExODhhMDczYTkyNWY2NjY0YTM3OTU1ZjdhNGZjYzA0YjQ1NjZlZGI0YjI1NThiYmFmIiwicGF0aCI6Imlzc3VlciJ9LHsidmFsdWUiOiI1ZTY2MTI0ZDBjNDk1ZWVlZDMyMzVhYmRmMGVkZDEwNmQ3NjQxN2M1ZjljZGRhMzA1MDM4ZDY5YTkzODc1ODRkIiwicGF0aCI6Imlzc3VhbmNlRGF0ZSJ9LHsidmFsdWUiOiJmNDMzMzE4YWU5YTZlZWY2OTc5NmIwZjhlZDRmMTYxYzk0YTM5MzJkN2M0MzEyNmMzZGVmYzNjN2Q0MDYyYzhlIiwicGF0aCI6ImNyZWRlbnRpYWxTdWJqZWN0LmlkIn0seyJ2YWx1ZSI6IjUzYjQwZjc4MjM3OTgzY2QyODg1N2NiMGYzN2EyZDg0YWQ1MzRjMjU5MTMwY2U2YzU0MTQ2ZDc2MTQ3MDhlYWMiLCJwYXRoIjoiY3JlZGVudGlhbFN1YmplY3QuYWx1bW5pT2YifSx7InZhbHVlIjoiNDBlZTQ2ZDBjN2NkYWUyZTE1ZTk2MjZlYzg4ZjZmY2QzZTAwMDNhYmQwMWQ4MDM1YjBhYTVmYTg0ZTY5MzljNSIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS50ZW1wbGF0ZS5uYW1lIn0seyJ2YWx1ZSI6ImY2OWUyMGI4YWUzNjg5MjY5NjhjNTY2ZjBjOTZmYjM0ZGQ3YzlmMTg3MzY0ZmIwMTg1YTU1Zjc1YmNiZTU4ZTEiLCJwYXRoIjoib3BlbkF0dGVzdGF0aW9uTWV0YWRhdGEudGVtcGxhdGUudHlwZSJ9LHsidmFsdWUiOiI0MGI1YWZjMjg2ODM0MDliMjZjMDBmNzcwYWFkZDBhNmM2MDUzN2Q1NGU2Y2Y2MjMxMTQwN2FmMzEwNDZkYmVkIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnRlbXBsYXRlLnVybCJ9LHsidmFsdWUiOiIwY2IwMTI0MGRlMzIyODBiOGMyOTkwMmFmZmQ0NmJmNzcyNzQ2ZWI3NTBlMjI0YjM3YmY2MzQzODgzZWVhZjNhIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLnByb29mLnR5cGUifSx7InZhbHVlIjoiNDE4NTdiMjhkZmM0MTJjMzc2NWM5NGFjYTIzOWRjZTE1MzgxMjVmNDgzNDc0MmU2YzZhODg5MDc5NWMyN2Q1OSIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5wcm9vZi5tZXRob2QifSx7InZhbHVlIjoiYTNiMjljMmMzOTkwY2Q5Mzc2MTFmZTU4ZmIyZjAxYmY3NTlhYmQxNjM2ZmJhZmZjOGNiZjRmYTIzOWZiYjlkNCIsInBhdGgiOiJvcGVuQXR0ZXN0YXRpb25NZXRhZGF0YS5wcm9vZi52YWx1ZSJ9LHsidmFsdWUiOiI4Y2I0ZjIzNDI0MzgzOTE1YWNmNjBhZTRlMGNjYTI5MTgzNGNkOWU3NTU0NGJhYjQ4ZDYyYzk3YWZhZjRlN2YxIiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLmlkZW50aXR5UHJvb2YuaWRlbnRpZmllciJ9LHsidmFsdWUiOiJkMWU2OWFhYWUwZWE0ZTJhNTg2YmYzMDk0MzExZTA1OTlkZjBmMTk0MTA4NDZkZjFmZTA3MjIwNGRmNjI1Njg2IiwicGF0aCI6Im9wZW5BdHRlc3RhdGlvbk1ldGFkYXRhLmlkZW50aXR5UHJvb2YudHlwZSJ9XQ==",
    "privacy": {
      "obfuscated": []
    }
  }
}
```

# Limitations of the Proof:
One limitation behind the selective disclosure method we used is that obfuscating all key-value pairs within an object would lead to an error, for example if an object named `foo` contains the keys `bar` and `baz`, then obfuscating both `bar` and `baz` would throw an error in our solution. Instead, what we recommend people to do is to actually obfuscate the entire object itself ie: `foo` in order to obfuscate all the values within.

Another limitation about the proof is that the encoded salts within the proof object is huge. This encoded salts would only grow linearly in size depending on the number of fields created within the credential, which might impact the portability of the credential, or hit the size constraints of a protocol such as maybe an embedded QR code.

# Related Work:
There are related works that have much different solutions from the proof proposed here. In [4], similar to us, use the leaf nodes to represent a claim, and they proposed methods to actually combine multiple subtrees together to get a combined credential consisting of different sets of claims. Also in [1], they applied the Merkle hash tree to construct redactable signatures, but did not use them in the context of a credential.

# Conclusion:
In conclusion, we hope that this paper gives an insight into the proof signature method we at `OpenAttesation` use, and hope that it would be adopted as it treads the middle ground between no selective disclosure (JWT) and standard cryptographic techniques (ZKP with BBS+ Signatures).

# References:
1. R. Johnson, D. Molnar, D. X. Song, and D. Wagner. Homomorphic signature schemes. In Topics in Cryptology – CTRSA 2002, volume 2271, pages 244–262. Springer-Verlag, 2002.
2. Manu Sporny; Grant Noble; Dave Longley; Daniel Burnett; Brent Zundel. W3C. 5 September 2019. W3C Proposed Recommendation. Verifiable Credentials Data Model 1.0. 
3. Kaliya Young “Identity Woman” COVID Credentials Initiative. Verifiable Credentials Flavors Explained
4. David Bauer, Douglas M. Blough, and David Cash. 2008. Minimal Information Disclosure with Efficiently Verifiable Credentials. In Proceedings of the 4th ACM Workshop on Digital Identity Management (Alexandria, Virginia, USA) (DIM ’08). Association for Computing Machinery, New York, NY, USA, 15–24.
5. Andreea A. Corci, Detleft Hühnlein, Tina Hühnlein, and Olaf Rode. Towards Interoperable Vaccination Certificate Services. ARES 2021, August 17–20, 2021, Vienna, Austria
6. Kenji Saito, Satoki Watanabe. Lightweight Selective Disclosure for Verifiable Documents on Blockchain. arXiv:2103.07655, March 13, 2021.
7. https://en.wikipedia.org/wiki/Rainbow_table
8. https://en.wikipedia.org/wiki/Merkle_tree

