# Selective Disclosure

## Status

Accepted

## Goal

Selective disclosure allows for holder of OA documents present subsets of the document to be verified. It does so by allowing users to obfuscated data fields without compromising the integrity of the document.

To achieve this, we compute a hash over individually hashed and salted key-value pairs. During obfuscation, we will hash the salted key-value pair and store the hash in another location of the document.

During integrity verification, we will recompute the hash of the document by:

1. Nomalising the document into individual key-value pairs
2. Hashing each of the individual key-value pairs
3. Compute a hash (merkle root) of all the hashes

In this document, we will describe the steps to:

1. Prepare an OA document
2. Obfuscate a value on an issued OA document
3. Check the integrity of an issued OA document

## Preparing the document

### Salting the data

To generate a OA document, through a process knwon as `document wrapping`, each individual value is converted from its primitive type to a string with a uuid, the data type and the original value.

The uuid prevents rainbow table attacks on the value after it has been obfuscated and the type will help disambiguate the original data type.

Sample raw data input:

```json
{
  "reference": "document identifier",
  "validFrom": "2010-01-01T19:23:24Z",
  "name": "document owner name",
  "template": {
    "name": "any",
    "type": "EMBEDDED_RENDERER",
    "url": "http://some.example.com"
  },
  "issuer": {
    "id": "http://some.example.com",
    "name": "DEMO STORE",
    "identityProof": { "type": "DNS-TXT", "location": "tradetrust.io" }
  },
  "proof": {
    "type": "OpenAttestationSignature2018",
    "value": "0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d",
    "method": "TOKEN_REGISTRY"
  },
  "key1": "value1",
  "key2": "value2",
  "key3": ["value3.1", "value3.2"],
  "key4": { "key41": "value4.1", "key42": "value4.2" }
}
```

Output after salt:

```json
{
  "reference": "2cc91073-3863-4db6-9f1a-b68665088caa:string:document identifier",
  "validFrom": "ca425b5c-d25d-4a36-b0f7-e368fbbaf75d:string:2010-01-01T19:23:24Z",
  "name": "c6864a9b-0367-4b8b-8cc5-f59cb6331f3d:string:document owner name",
  "template": {
    "name": "36a95964-fbe9-4c6d-ab6a-a5247c07f105:string:any",
    "type": "45c266d0-3efe-4841-893c-96723cd7ddb5:string:EMBEDDED_RENDERER",
    "url": "bbec168c-b9c3-4fb3-aa6f-3d1a1017f1e9:string:http://some.example.com"
  },
  "issuer": {
    "id": "42b6991b-e4b5-444b-aba7-7ce97bbe8441:string:http://some.example.com",
    "name": "da3fa13d-b0a9-4357-bd4c-4f3941304225:string:DEMO STORE",
    "identityProof": {
      "type": "e2039c5b-6cab-4679-9150-7266e34371a4:string:DNS-TXT",
      "location": "684bb30d-a59e-4cc3-a8ed-44e6b72579d9:string:tradetrust.io"
    }
  },
  "proof": {
    "type": "bc64159e-90f3-4f0c-99bb-43818ff402e5:string:OpenAttestationSignature2018",
    "value": "edd73480-3043-4cdc-aefd-f4ee21fc6a7f:string:0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d",
    "method": "6ae5877f-be97-47df-99f7-7464a5ca1cde:string:TOKEN_REGISTRY"
  },
  "key1": "c1ba53a6-68b8-421e-ac80-7d4b71ca8e52:string:value1",
  "key2": "0356d74e-10cd-4cc1-b1ba-eab23f8bea3d:string:value2",
  "key3": [
    "44faf2a4-d16f-4945-8784-d2b8a088edc5:string:value3.1",
    "18eaf71f-3b4a-4064-b8f5-79092c2e04aa:string:value3.2"
  ],
  "key4": {
    "key41": "7e3b72f8-e521-4892-a26c-b56d3e2ab5cf:string:value4.1",
    "key42": "c4f96a54-dda6-4fe2-9f49-bc77e8e19d02:string:value4.2"
  }
}
```

### Computing individual hashes

After the individual values has been salted, the entire data object is flattened, while preserving the hierarchial structure of the data within the individual keys.

In our implementation the library [flatley](https://www.npmjs.com/package/flatley) is used to flatten the object and [js-sha3](https://www.npmjs.com/package/js-sha3) is used to compute the `keccak256` hash.

Output after flattening:

```json
{
  "reference": "2cc91073-3863-4db6-9f1a-b68665088caa:string:document identifier",
  "validFrom": "ca425b5c-d25d-4a36-b0f7-e368fbbaf75d:string:2010-01-01T19:23:24Z",
  "name": "c6864a9b-0367-4b8b-8cc5-f59cb6331f3d:string:document owner name",
  "template.name": "36a95964-fbe9-4c6d-ab6a-a5247c07f105:string:any",
  "template.type": "45c266d0-3efe-4841-893c-96723cd7ddb5:string:EMBEDDED_RENDERER",
  "template.url": "bbec168c-b9c3-4fb3-aa6f-3d1a1017f1e9:string:http://some.example.com",
  "issuer.id": "42b6991b-e4b5-444b-aba7-7ce97bbe8441:string:http://some.example.com",
  "issuer.name": "da3fa13d-b0a9-4357-bd4c-4f3941304225:string:DEMO STORE",
  "issuer.identityProof.type": "e2039c5b-6cab-4679-9150-7266e34371a4:string:DNS-TXT",
  "issuer.identityProof.location": "684bb30d-a59e-4cc3-a8ed-44e6b72579d9:string:tradetrust.io",
  "proof.type": "bc64159e-90f3-4f0c-99bb-43818ff402e5:string:OpenAttestationSignature2018",
  "proof.value": "edd73480-3043-4cdc-aefd-f4ee21fc6a7f:string:0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d",
  "proof.method": "6ae5877f-be97-47df-99f7-7464a5ca1cde:string:TOKEN_REGISTRY",
  "key1": "c1ba53a6-68b8-421e-ac80-7d4b71ca8e52:string:value1",
  "key2": "0356d74e-10cd-4cc1-b1ba-eab23f8bea3d:string:value2",
  "key3.0": "44faf2a4-d16f-4945-8784-d2b8a088edc5:string:value3.1",
  "key3.1": "18eaf71f-3b4a-4064-b8f5-79092c2e04aa:string:value3.2",
  "key4.key41": "7e3b72f8-e521-4892-a26c-b56d3e2ab5cf:string:value4.1",
  "key4.key42": "c4f96a54-dda6-4fe2-9f49-bc77e8e19d02:string:value4.2"
}
```

Once the data object has been salted and flattened, each individual key-value pair will be hashed.

In the example above, we will apply the `keccak256` hash on the object `{"key4.key41":"7e3b72f8-e521-4892-a26c-b56d3e2ab5cf:string:value4.1"}` which will yield `c3edad333f0829b92a82cd3c09b7795b0f00f07dfbbfc5ff8779272d1eaba3a8`.

Once this function has been applied to all the keys, the output hashes will be sorted and stored in an array.

Output after all hashing individual keys (sorted):

```json
[
  "1b61eaed9b46a53c530ca7cbe1ed54621e9478b86efb445a932f3cb38d6a97da",
  "28e72c3815a932b137f3dc0d9d9fa101d6d7477793acf5adcb5f34f0901c3811",
  "2a73ae2d0e942748f632130c530e4d3eb0e91dcf8e560c52586ecf2cf84daefd",
  "32559f7f05f498d2ffe89345a379fa6619cd20466474386b26ac8b328b28e03e",
  "5464490fed778a23505a0da8cb8ffa8ebf47b85fa221f544d0757a96e5be315d",
  "5c51bab9dda34200e24a5bf46ce39a82561515239f869d444cd7b458bb986483",
  "6b19136bf4ff28c270b913786c329c6c1186857a5fb5b9897bc4bdf00f95041d",
  "6ec1924f5f25afe82b01ed108775ab27125b0e7277687a1c1acef27ce34cb189",
  "87da7c3a74118736144688664ffa621e7e39837c23719a0b4c88ee0fa0b08357",
  "88c680973e4c58095b222aabfe2d62ffe8bf3a9dacccb6cea112805abdb2d475",
  "91788b66ce30099e244d180bc18ce16b98de48598cd2ebf9f1cc7c508dc1b65e",
  "9e029d96b11e4f4e4a37cb3bf03bf3af854b8ea059686945072f02c215e0aba6",
  "a0b0a1e5992ad724ce101921001d9de2ff08a64c65d9e92b47142eb76fa03618",
  "c04833e08f6a0d17cb8388db75dec3b07c603be118a1303f68332ae7eef26db5",
  "c3edad333f0829b92a82cd3c09b7795b0f00f07dfbbfc5ff8779272d1eaba3a8",
  "c6daa8acbd7b7591d6ea0055a6de3827b953ea3e295c024fc31c167cac1f112d",
  "d8b26580983759e8cad2bcfd10dffdf375cd49b1e5d7b4424d7698216b683218",
  "e3494dd41622c560a2df8cf9051f75a27ef651c46826e3eef6382187825794d5",
  "f29d9f500d3843f547158fcd8b69b67406ad492621d9702a7903062f86ef97b0"
]
```

### Computing document's targetHash

Finally, with the array of sorted hashes, `keccak256` is applied again on the stringified array to obtain the `targetHash` of the document.

```js
keccak256(
  JSON.stringify([
    "1b61eaed9b46a53c530ca7cbe1ed54621e9478b86efb445a932f3cb38d6a97da",
    "28e72c3815a932b137f3dc0d9d9fa101d6d7477793acf5adcb5f34f0901c3811",
    "2a73ae2d0e942748f632130c530e4d3eb0e91dcf8e560c52586ecf2cf84daefd",
    "32559f7f05f498d2ffe89345a379fa6619cd20466474386b26ac8b328b28e03e",
    "5464490fed778a23505a0da8cb8ffa8ebf47b85fa221f544d0757a96e5be315d",
    "5c51bab9dda34200e24a5bf46ce39a82561515239f869d444cd7b458bb986483",
    "6b19136bf4ff28c270b913786c329c6c1186857a5fb5b9897bc4bdf00f95041d",
    "6ec1924f5f25afe82b01ed108775ab27125b0e7277687a1c1acef27ce34cb189",
    "87da7c3a74118736144688664ffa621e7e39837c23719a0b4c88ee0fa0b08357",
    "88c680973e4c58095b222aabfe2d62ffe8bf3a9dacccb6cea112805abdb2d475",
    "91788b66ce30099e244d180bc18ce16b98de48598cd2ebf9f1cc7c508dc1b65e",
    "9e029d96b11e4f4e4a37cb3bf03bf3af854b8ea059686945072f02c215e0aba6",
    "a0b0a1e5992ad724ce101921001d9de2ff08a64c65d9e92b47142eb76fa03618",
    "c04833e08f6a0d17cb8388db75dec3b07c603be118a1303f68332ae7eef26db5",
    "c3edad333f0829b92a82cd3c09b7795b0f00f07dfbbfc5ff8779272d1eaba3a8",
    "c6daa8acbd7b7591d6ea0055a6de3827b953ea3e295c024fc31c167cac1f112d",
    "d8b26580983759e8cad2bcfd10dffdf375cd49b1e5d7b4424d7698216b683218",
    "e3494dd41622c560a2df8cf9051f75a27ef651c46826e3eef6382187825794d5",
    "f29d9f500d3843f547158fcd8b69b67406ad492621d9702a7903062f86ef97b0"
  ])
);
```

Result:

```json
"51d6b872aae578d6a4b7decd4370f50b73b5729d8357f3057e240c10bae64ab2"
```

### Assembling the OA document

Once the `targetHash` of the document is calculated, it can be appended to the document under the `signature` object.

Example of wrapped OA Document:

```json
{
  "version": "open-attestation/3.0",
  "data": {
    "reference": "2cc91073-3863-4db6-9f1a-b68665088caa:string:document identifier",
    "validFrom": "ca425b5c-d25d-4a36-b0f7-e368fbbaf75d:string:2010-01-01T19:23:24Z",
    "name": "c6864a9b-0367-4b8b-8cc5-f59cb6331f3d:string:document owner name",
    "template": {
      "name": "36a95964-fbe9-4c6d-ab6a-a5247c07f105:string:any",
      "type": "45c266d0-3efe-4841-893c-96723cd7ddb5:string:EMBEDDED_RENDERER",
      "url": "bbec168c-b9c3-4fb3-aa6f-3d1a1017f1e9:string:http://some.example.com"
    },
    "issuer": {
      "id": "42b6991b-e4b5-444b-aba7-7ce97bbe8441:string:http://some.example.com",
      "name": "da3fa13d-b0a9-4357-bd4c-4f3941304225:string:DEMO STORE",
      "identityProof": {
        "type": "e2039c5b-6cab-4679-9150-7266e34371a4:string:DNS-TXT",
        "location": "684bb30d-a59e-4cc3-a8ed-44e6b72579d9:string:tradetrust.io"
      }
    },
    "proof": {
      "type": "bc64159e-90f3-4f0c-99bb-43818ff402e5:string:OpenAttestationSignature2018",
      "value": "edd73480-3043-4cdc-aefd-f4ee21fc6a7f:string:0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d",
      "method": "6ae5877f-be97-47df-99f7-7464a5ca1cde:string:TOKEN_REGISTRY"
    },
    "key1": "c1ba53a6-68b8-421e-ac80-7d4b71ca8e52:string:value1",
    "key2": "0356d74e-10cd-4cc1-b1ba-eab23f8bea3d:string:value2",
    "key3": [
      "44faf2a4-d16f-4945-8784-d2b8a088edc5:string:value3.1",
      "18eaf71f-3b4a-4064-b8f5-79092c2e04aa:string:value3.2"
    ],
    "key4": {
      "key41": "7e3b72f8-e521-4892-a26c-b56d3e2ab5cf:string:value4.1",
      "key42": "c4f96a54-dda6-4fe2-9f49-bc77e8e19d02:string:value4.2"
    }
  },
  "privacy": {
    "obfuscatedData": []
  },
  "signature": {
    "type": "SHA3MerkleProof",
    "targetHash": "51d6b872aae578d6a4b7decd4370f50b73b5729d8357f3057e240c10bae64ab2",
    "proof": [],
    "merkleRoot": "51d6b872aae578d6a4b7decd4370f50b73b5729d8357f3057e240c10bae64ab2"
  }
}
```

## Obfuscating a value

To obfuscate a value in the OA document, one simply hash the key-value pair with `keccak256` and store the resulting hash in `privacy.obfuscatedData`. This method can be used to obfuscate multiple key-value pairs in the document data file.

For example to obfuscated `key4.key41`:

```js
keccak256(
  JSON.stringify({
    "key4.key41": "7e3b72f8-e521-4892-a26c-b56d3e2ab5cf:string:value4.1"
  })
);
```

Resulting hash:

```sh
c3edad333f0829b92a82cd3c09b7795b0f00f07dfbbfc5ff8779272d1eaba3a8
```

This hash will then be appended to the original OA document and the object that has been obfuscated will be removed:

```json
{
  "version": "open-attestation/3.0",
  "data": {
    "reference": "2cc91073-3863-4db6-9f1a-b68665088caa:string:document identifier",
    "validFrom": "ca425b5c-d25d-4a36-b0f7-e368fbbaf75d:string:2010-01-01T19:23:24Z",
    "name": "c6864a9b-0367-4b8b-8cc5-f59cb6331f3d:string:document owner name",
    "template": {
      "name": "36a95964-fbe9-4c6d-ab6a-a5247c07f105:string:any",
      "type": "45c266d0-3efe-4841-893c-96723cd7ddb5:string:EMBEDDED_RENDERER",
      "url": "bbec168c-b9c3-4fb3-aa6f-3d1a1017f1e9:string:http://some.example.com"
    },
    "issuer": {
      "id": "42b6991b-e4b5-444b-aba7-7ce97bbe8441:string:http://some.example.com",
      "name": "da3fa13d-b0a9-4357-bd4c-4f3941304225:string:DEMO STORE",
      "identityProof": {
        "type": "e2039c5b-6cab-4679-9150-7266e34371a4:string:DNS-TXT",
        "location": "684bb30d-a59e-4cc3-a8ed-44e6b72579d9:string:tradetrust.io"
      }
    },
    "proof": {
      "type": "bc64159e-90f3-4f0c-99bb-43818ff402e5:string:OpenAttestationSignature2018",
      "value": "edd73480-3043-4cdc-aefd-f4ee21fc6a7f:string:0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d",
      "method": "6ae5877f-be97-47df-99f7-7464a5ca1cde:string:TOKEN_REGISTRY"
    },
    "key1": "c1ba53a6-68b8-421e-ac80-7d4b71ca8e52:string:value1",
    "key2": "0356d74e-10cd-4cc1-b1ba-eab23f8bea3d:string:value2",
    "key3": [
      "44faf2a4-d16f-4945-8784-d2b8a088edc5:string:value3.1",
      "18eaf71f-3b4a-4064-b8f5-79092c2e04aa:string:value3.2"
    ],
    "key4": { "key42": "c4f96a54-dda6-4fe2-9f49-bc77e8e19d02:string:value4.2" }
  },
  "privacy": {
    "obfuscatedData": [
      "c3edad333f0829b92a82cd3c09b7795b0f00f07dfbbfc5ff8779272d1eaba3a8"
    ]
  },
  "signature": {
    "type": "SHA3MerkleProof",
    "targetHash": "51d6b872aae578d6a4b7decd4370f50b73b5729d8357f3057e240c10bae64ab2",
    "proof": [],
    "merkleRoot": "51d6b872aae578d6a4b7decd4370f50b73b5729d8357f3057e240c10bae64ab2"
  }
}
```

## Check integrity of the document

To check the integrity of the document, we simply:

1. Flatten the data object
2. Hash individual key-value pairs
3. Append and sort the hash from (2) with the hashes from `privacy.obfuscatedData`
4. Compute the keccak256 hash for the results from (3)
5. Check that the resulting hash from (4) matches the `targetHash`

## Implementation

[OpenAttestation](https://github.com/Open-Attestation/open-attestation)
