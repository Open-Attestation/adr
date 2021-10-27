# Abstract:

This document discusses two methods of determining a certificate's revocation status. Both Certificate Revocation Lists (CRLs) and the Online Certificate Status Protocol (OCSP) methods are explored and discussed.

# Problem Statement:

There is currently no way to revoke a document signed with a Decentralised Identifier (DID) without a deployed document store. As one of the main reasons for using DIDs to sign documents (as opposed to issuing documents via a document store) is to reduce cost arising from transactions on the Ethereum network, deploying a document store to keep track of revoked DID-signed documents is counter-intuitive. As such, there needs to be a solution to keep tracked of DID-signed documents without deploying a document store.

# Possible Solutions

## Certificate Revocation List

A CRL is a time-stamped list identifying revoked certificates that is signed by a CA or CRL issuer and made freely available in a public repository. Each revoked certificate is identified in a CRL by its certificate serial number. When a certificate-using system uses a certificate (e.g., for verifying a remote user's digital signature), that system not only checks the certificate signature and validity but also acquires a suitably recent CRL and checks that the certificate serial number is not on that CRL. [1]

## Online Certificate Status Protocol

The Online Certificate Status Protocol (OCSP) enables applications to determine the (revocation) state of identified certificates. OCSP may be used to satisfy some of the operational requirements of providing more timely revocation information than is possible with CRLs and may also be used to obtain additional status information. An OCSP client issues a status request to an OCSP responder and suspends acceptance of the certificates in question until the responder provides a response. [2]

## Comparison

| CRL                                                  | OCSP                                     |
| ---------------------------------------------------- | ---------------------------------------- |
| Published periodically (statuses may not be updated) | Available in real time                   |
| Entire list is downloaded by requester               | Single response is provided to requester |
| Published by Certificate Authority                   | Managed by Certificate Authority         |

### Certificate Revocation List

#### Benefits

- works offline
- CRL distributors do not need to be trusted as the CRL itself is signed

#### Drawbacks

- statuses of certificates may not reflect actual statuses as the CRL is only published periodically
- a high bandwidth is required to download the entire CRL, making it susceptible to a denial-of-service (DOS) attack
- for the making of single or few requests periodically, using a CRL is slow as the whole CRL would need to be downloaded

## Online Certificate Status Protocol

#### Benefits

- statuses reflect actual certificate statuses
- doesn't require downloading of entire CRL to check a few certificates

#### Drawbacks

- slow for bulk requests, would require processing many HTTP requests
- doesn't work offline
- service needs to be trusted

### Conclusion

The appropriate method of determining a certificate's revocation status depends on the use-case. If ensuring that statuses are up-to-date whenever a request is made is of top priority, OCSP should be the method of choice. If being able to verify a certificate's status offline is of prime importance, however, CRLs should be used.

Taking for example the scenario of checking if work passes have been revoked by the ICA,

- If _checking is done occasionally on an individual worker's pass_, use OCSP to make a request with the workpass identifier and immediately receive a response containing the status of the pass, removing the need to download an entire list of passes unrelated to the request

- If _checking is done on all workers belonging to a specific company on a fixed day_, a CRL could be downloaded immediately after the most recent publish date and a check could be done on all the workers offline, removing the need to make multiple requests to an OCSP responder

# Proposed Implementation

## Certificate Revocation List

This section was written based on RFC5280.

Relevant sections:

- [3.3 Revocation](https://datatracker.ietf.org/doc/html/rfc5280#section-3.3)
- [5.1 CRL Fields](https://datatracker.ietf.org/doc/html/rfc5280#section-5.1)
- [6.3 CRL Validation](https://datatracker.ietf.org/doc/html/rfc5280#section-6.3)

### CRL Object

The CRL object, as adapted for the extension of OpenAttestation is as follows:

> **_NOTE:_** We use the [DID verifiable document](https://www.openattestation.com/docs/verifiable-document/did/create) approach to create a verifiable CRL

```json
{
  "version": "https://schema.openattestation.com/2.0/schema.json",
  "data": {
    "issuers": [
      {
        "id": "527d39aa-d2fa-4e98-b219-0991b2c12b68:string:did:ethr:0x296eCD182E9A0BD845120691880219D6b26ACF8F",
        "name": "fabeb58e-846d-48c4-8a3a-24f8997fca9d:string:Demo Issuer",
        "revocation": {
          "type": "b6af10ff-fa48-453c-8b6c-81ee24a05c88:string:NONE"
        },
        "identityProof": {
          "type": "628950fd-adc0-42ce-8db1-a8fd91f3a317:string:DNS-DID",
          "location": "f97b32e7-7881-46f9-8b44-c8932a050afc:string:qualified-apricot-spoonbill.sandbox.openattestation.com",
          "key": "fd878c77-166d-4944-a6a5-1ebd510fd593:string:did:ethr:0x296eCD182E9A0BD845120691880219D6b26ACF8F#controller"
        }
      }
    ],
    "tbsCertList": {
      "serialNumber": 123567890,
      "validity": {
        "notBefore": "2021-10-26T05:02:20.100Z",
        "notAfter": "2021-10-27T05:02:20.100Z"
      },
      "thisUpdate": "2021-10-26T05:02:20.100Z",
      "nextUpdate": "2021-10-27T05:02:20.100Z",
      "revokedCertificates": [
        {
          "userCertificate": 1,
          "revocationDate": "2021-10-24T07:09:10.320Z",
          "reasonCode": 1
        },
        {
          "userCertificate": 2,
          "revocationDate": "2021-10-24T07:09:10.320Z",
          "reasonCode": 4
        },
        {
          "userCertificate": 3,
          "revocationDate": "2021-10-24T07:09:10.320Z",
          "reasonCode": 9
        }
      ]
    }
  },
  "signature": {
    "type": "SHA3MerkleProof",
    "targetHash": "650e44f3854f803fccf83693dd6a9ee039462c5d21c3ec459182993a6d24b0e4",
    "proof": [],
    "merkleRoot": "650e44f3854f803fccf83693dd6a9ee039462c5d21c3ec459182993a6d24b0e4"
  },
  "proof": [
    {
      "type": "OpenAttestationSignature2018",
      "created": "2021-10-26T05:02:20.100Z",
      "proofPurpose": "assertionMethod",
      "verificationMethod": "did:ethr:0x296eCD182E9A0BD845120691880219D6b26ACF8F#controller",
      "signature": "0x9de62ad3e3c992268532ab29d8ec692fd473c97bb8b02b90d1a90cda63ba31b37151b39a2bbb70c4095e053b9811598360bf6825e73392f4f6500ce6608893c41c"
    }
  ]
}
```

- `tbsCertList` references the to-be-signed CRL, which is signed using the DID method.

- `serialNumber` is the identifier of the CRL in the `tbsCertList`

- `validity` refers to the validity of the CRL

  - `notBefore` specifies the date from which the document is valid i.e. the document is valid _on or after_ this date but not before
  - `notAfter` specifies the date after which the document becomes invalid i.e. the document is valid _on or before_ this date but not after

    <br/>

- `thisUpdate` specifies the date on which the document was published
- `nextUpdate` specifies the date on which the next document will be published
- `revokedCertificates` lists the certificates which have been revoked
  - `userCertificate` refers to the certificate identifier
  - `revocationDate` specifies the date on which the certificate was revoked
  - `reasonCode` identifies the reason for certificate revocation. See the list of reason codes [here](#reason-codes)

### Revocation Checking Process

Assuming that an entity has previously downloaded a CRL,

- Check if the downloaded CRL is valid by comparing the `valid.notBefore` and `valid.notAfter` values with the current time.

  - If the downloaded CRL is **valid**,

    1. Iterate through `revokedCertificates` and compare the identifier of the certificate to be checked with the `userCertificate` value.
    2. If iteration completes without a match, the certificate is not on the CRL

       > **_NOTE:_** This does not mean that the certificate has definitely been revoked! It is possible that a mistake was made when publishing the CRL and a revoked certificate was not added to the updated list

  - If the downloaded CRL is **invalid**,
    1. Download the most recent CRL from the CRL distribution point
    2. Verify that the CRL is valid by checking the values of `valid` and checked that it has been signed by a trusted authority
    3. Follow the steps for when a CRL is valid

If the entity is not in possession of a previously downloaded CRL, simply download the CRL from a distribution point and follow the steps listed above.

### Distributing the CRL

CRL distribution for our purposes can be done through a simple web server. The entity who wishes to obtain the latest CRL may make a HTTP GET request with an API key to obtain the most recently published CRL. The CA may wish to rate-limit the requests as downloading of a CRL require significant bandwidth.

## Online Certificate Status Protocol

This section was written based on RFC6960.

Relevant sections:

- [4.1 Request Syntax](https://datatracker.ietf.org/doc/html/rfc6960#section-4.1)
- [4.2 Response Syntax](https://datatracker.ietf.org/doc/html/rfc6960#section-4.2)

### API

POST /:certListSerialNumber

REQUEST

```json
{
  "requestList": [
    {
      "reqCert": "0123456789"
    },
    {
      "reqCert": "0123456790"
    }
  ]
}
```

RESPONSE

```json
{
  "producedAt": "2021-10-26T05:02:20.100Z",
  "responses": [
    {
      "certID": "0123456789",
      "certStatus": "good",
      "thisUpdate": "2021-10-26T05:02:20.100Z"
    },
    {
      "certID": "0123456790",
      "certStatus": "revoked",
      "thisUpdate": "2021-10-26T05:02:20.100Z"
    }
  ]
}
```

# Appendix

## Reason Codes

| Reason               | Code |
| -------------------- | ---- |
| unspecified          | 0    |
| keyCompromise        | 1    |
| caCompromise         | 2    |
| affiliationChanged   | 3    |
| superseded           | 4    |
| cessationOfOperation | 5    |
| certificateHold      | 6    |
| NOT USED             | 7    |
| removeFromCRL        | 8    |
| privilegeWithdrawn   | 9    |
| aACompromise         | 10   |

> **_NOTE:_** The code number 7 is not used

## Certificate Status Value

| Status  | Description                                             |
| ------- | ------------------------------------------------------- |
| good    | certificate is not currently revoked                    |
| revoked | certificate has been temporarily or permanently revoked |

# References:

1. D. Cooper, S. Santesson, S. Farrell, S. Boeyen, R. Housley, W. Polk (2008). Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile. RFC 5280. Retrieved from https://datatracker.ietf.org/doc/html/rfc5280

2. S. Santesson, M. Myers, R. Ankney, A. Malpani, S. Galperin, C. Adams (2013). X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP. RFC 6960. Retrieved from https://datatracker.ietf.org/doc/html/rfc6960
