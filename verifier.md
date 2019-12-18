# OpenAttestation Verifier

## Status

[Draft](https://github.com/Open-Attestation/oa-verify/pull/74)

## Goal

The purpose of the verifier is to provide a generic verification method to verify OpenAttestation(OA) documents. The verifier will provide default verification methods conforming to the standard verification process proposed in OpenAttestation yet providing opportunities for it to be extended.

## Architecture

![Architecture](./assets/verifier/architecture.png)

### Verification Methods

A verifier is made up of multiple `Verification Methods`. In the diagram above, `OpenAttestationDnsTxt`, `OpenAttestationEthereumDocumentStoreIssued` and `OpenAttestationHash` are examples of `Verification Methods` provided.

The role of a verification method is to `verify` the OA document if it is valid. Since there are many types and versions of OA document, not all test should run against all types of OA document. For that reason, a `test` method is also defined to test if a method should run against a document. If the method is incompatible with the document type, it should be `skip` the method.

As a result, a verification method should implement 3 abstract method: `verify`, `test` and `skip`.

#### test(document: Document): boolean

This function takes in an OA document and returns a boolean result that determines if this verification method is meant to be ran against the document.

In the case that the test passes, we should run the `verify` function. Otherwise, the `skip` function should be ran.

#### skip(): VerificationFragment

This function should be ran when the verification method is skipped for the given OA document. It will return the skipped `VerificationFragment` for the test.

An example of `VerificationFragment` that skips the DNS-TXT verification:

```json
{
  "name": "OpenAttestationDnsTxt",
  "type": "ISSUER_IDENTITY",
  "status": "SKIPPED",
  "message": "Document issuers doesn't have \"documentStore\" or \"token\" property"
}
```

#### verify(document: Document): VerificationFragment

This function should be ran when the test passes. Running this function will execute the necessary computation to determine if the document passes the verification method.

An example `VerificationFragment` of a passing test:

```json
{
  "name": "OpenAttestationEthereumDocumentStoreIssued",
  "type": "DOCUMENT_STATUS",
  "status": "VALID",
  "data": {
    "details": [
      {
        "address": "0x007d40224f6562461633ccfbaffd359ebb2fc9ba",
        "issued": true
      }
    ],
    "issuedOnAll": true
  }
}
```

An example `VerificationFragment` of a failed test:

```json
{
  "name": "OpenAttestationEthereumDocumentStoreIssued",
  "type": "DOCUMENT_STATUS",
  "status": "INVALID",
  "message": "Certificate has not been issued",
  "data": {
    "details": [
      {
        "address": "0x20bc9C354A18C8178A713B9BcCFFaC2152b53990",
        "error": "call exception (address=\"0x20bc9C354A18C8178A713B9BcCFFaC2152b53990\", method=\"isIssued(bytes32)\", args=[\"0x85df2b4e905a82cf10c317df8f4b659b5cf38cc12bd5fbaffba5fc901ef0011b\"], version=4.0.40)",
        "issued": false
      }
    ],
    "issuedOnAll": false
  }
}
```

### Verification Classes

A `Verification Class` is a class label to a verification method. It specifies what type of verification the method is performing. The diagram above shows the 3 default verification classes: `ISSUER_IDENTITY`, `DOCUMENT_INTEGRITY` and `DOCUMENT_STATUS`.

For the validity of the verification class to be true, the requirements must be met:

1. At least one method in that class should return `VALID` as the status.
1. All methods in the class should return either `VALID` or `SKIPPED` as the status.

### Verifier

The verifier is a function used to verify any OpenAttestation document. It returns a set of `VerificationFragment` from the different verification methods.

From the `VerificationFragments` we can then determine if the OA document is valid. The OA document is valid when all `Verification Classes` has valid statuses.

## Default Methods

### Document Integrity

#### OpenAttestationHash

### Document Status

#### OpenAttestationEthereumDocumentStoreIssued

#### OpenAttestationEthereumDocumentStoreRevoked

### Issuer Identity

#### OpenAttestationDnsTxt

## Usage

To use the default verifier:

```js
const document = "OA-Document";
const verificationFragments = verify(document);
const verified = isVerified(verificationFragments);
```

To extend the verifier with a custom name registry:

```js
const document = "OA-Document-With-Custom-Identity-Proof";
const customIdentityRegistry = "Custom-Verification-Method";
const verificationFragments = verificationBuilder(document, [
  ...defaultVerifiers,
  customIdentityRegistry
]);
const verified = isVerified(verificationFragments);
```
