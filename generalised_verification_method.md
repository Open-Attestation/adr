# Generalised Verification Method

## Status

Draft

## Rationale

Currently OpenAttestation (OA) uses it's own set of verification method to query smart contracts on Ethereum based on the issuer's key `certificateStore`(legacy), `documentStore`, or `tokenRegistry`. This means that new verification method will change the core OA schema to include the keys into the `issuer` object. Rather than that, we can reference how W3C VC approach this problem using a [registry of methods](https://w3c-ccg.github.io/vc-extension-registry/). This allows us to be more interoperable with W3C VC if we upstream the change as a proof method to them and also allow us to extend the methods in the future.

Below is the proposed data format to allow multiple issuers to use a new `proof` object to say how to verify the document.

```json
{
  "data": {
    "id": "SALT:SN-123",
    "foo": "SALT:bar",
    "issuers": [
      {
        "name": "SALT:Issuer",
        "identityProof": {
          "type": "SALT:DNS-TXT",
          "location": "SALT:example.com",
          "subject": "SALT:proof.value"
        }
      }
    ],
    "proof": {
      "type": "SALT:OpenAttestationSignature2018",
      "method": "SALT:TOKEN_REGISTRY",
      "value": "SALT:0xABCD"
    }
  }
}
```

```json
{
  "data": {
    "id": "SALT:SN-123",
    "foo": "SALT:bar",
    "issuers": [
      {
        "name": "SALT:Issuer 1",
        "identityProof": {
          "type": "SALT:DNS-TXT",
          "location": "SALT:example1.com",
          "subject": "SALT:proof.value[0]"
        }
      },
      {
        "name": "SALT:Issuer 2",
        "identityProof": {
          "type": "SALT:DNS-TXT",
          "location": "SALT:example2.com",
          "subject": "SALT:proof.value[1]"
        }
      }
    ],
    "proof": {
      "type": "SALT:OpenAttestationSignature2018",
      "method": "SALT:DOCUMENT_STORE",
      "value": ["SALT:0xABCD", "SALT:0x1234"]
    }
  }
}
```

## Alternative Data Model:

```json
{
  "data": {
    "id": "SALT:SN-123",
    "foo": "SALT:bar",
    "issuers": [
      {
        "name": "SALT:Issuer 1"
      },
      {
        "name": "SALT:Issuer 2"
      }
    ],
    "proof": [
      {
        "type": "SALT:OpenAttestationSignature2018",
        "method": "SALT:DOCUMENT_STORE",
        "value": "SALT:0xABCD",
        "identityProof": {
          "type": "SALT:DNS-TXT",
          "location": "SALT:example1.com",
          "subject": "SALT:proof.value[0]"
        }
      },
      {
        "type": "SALT:OpenAttestationSignature2018",
        "method": "SALT:DOCUMENT_STORE",
        "value": "SALT:0xABCD",
        "identityProof": {
          "type": "SALT:DNS-TXT",
          "location": "SALT:example2.com",
          "subject": "SALT:proof.value[1]"
        }
      }
    ]
  }
}
```
