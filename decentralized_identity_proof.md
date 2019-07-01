# Decentralized Identity Proof

## Using DNS to verify the identity of a OA document issuer

DNS records can be used to proof decentrally that an entity is the owner of the a document store. To establish the proof, the issuer needs to:

1. Create a TXT record on their DNS under the root domain or sub domain
2. Add the DNS domain into the issuer object

### DNS Record on openattestation.com

```
TXT     0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d
```

### Identity proof on the OA Document

```json
"issuers": [
    {
        "network": "ETHEREUM",
        "documentStore": "0x9178F546D3FF57D7A6352bD61B80cCCD46199C2d",
        "identityProof": {
            "type": "DNS-TXT",
            "domain": "openattestation.com"
        }
    }
]
```

### Verifying identity

To proof the identity of the owner of the documentStore, we can query the TXT records of the domain to check if the `documentStore` address exists. 

```sh
curl https://dns.google/resolve?name=openattestation.io&type=TXT
```

At this point, if the `documentStore` address is one of the TXT records, we can say that the document is issued from the domain. 

### Messaging

An appropriate message could be

```
Issued by openattestation.com
```

### Issues with using DNS

#### Non-repudiation

As DNS records can be altered by the controlling entity, the entity can remove the DNS record after issuing a document and there is no way to proof that they have issued the documents in the first place.

We found this issue to be trivial since the controlling entity should be in control of what constitute a valid or invalid document. 


## Can other identity proofs be used?

```json
"issuers": [
    {
        "identityProof": {
            "type": "ENS-TXT",
            "domain": "opencerts.ens"
        }
    }
]
```

```json
"issuers": [
    {
        "identityProof": {
            "type": "UPORT-SIGNED",
            "domain": "<SIGNED-DOCUMENT-STORE-ADDRESS-BY-PRIVATE-KEY>"
        }
    }
]
```

## Why is a centralised registry not included

If a centralised registry is used, it would have been hardcoded in the application using the centralised registry. It shouldn't be in the document created as it will not be useful to someone else that does not subscribe to the centralised registry. If they are subscribe to it, they will hardcode the registry as well. 

## Why tokenStore does not require identity proof

The `documentStore` alone will proof that issuer intends to create the document. It does not need to be conveyed again through a separate proof for `tokenStore`. 

In addition, this will allow an anonymity cloak to their tokenStores. Issuers can create multiple token stores, even to the extent of one store per token, to hide transaction histories of a document.
