# Decentralized Issuer's Identity Proof using DNS-TXT
The previous method of identifying issuer DocumentStore smart contract addresses was using a simple key-value mapping recorded in JSON, hosted and administered by the OpenCerts core development team. 

This method has the following issues:
  - It makes OpenCerts core development team effectively the authority on issuer identity
  - It is a manual system which can be prone to human error, and is a single point of failure/attack
  - The identity review is reliant on out-of-band methods such as email or face to face meetings
  - It's not scalable

## Using DNS to verify the identity of a OA document issuer

DNS is largely decentralised, not perfectly but it's a good tradeoff. DNS TXT records can only be created by the administrator of a domain, and hence can be used to show that a domain owner has endorsed a certain smart contract address. Ideally DNSSEC cryptographic proofs should be used, but at this point in time unfortunately DNSSEC adoption is still not widespread. We will revisit DNSSEC proofs at a later date, as it solves many of the issues listed below.

To establish the proof, the issuer needs to:

1. Create a TXT record on their DNS under the root domain or sub domain
2. Add the DNS domain into the issuer object

[Constraints](https://support.agari.com/hc/en-us/articles/202952749-How-long-can-my-SPF-record-be-):

    the limit to a TXT string is 255
    the limit to a UDP packet is 512
    the limit to total of TXT data for a given record is 65535


### Example DNS-TXT Record on openattestation.com, for a Ethereum-Ropsten document store
```
TXT     opencerts net=ethereum net-id=3 addr=0x0c9d5E6C766030cc6f0f49951D275Ad0701F81EC
```


### Example DNS-TXT Record on openattestation.com, for a Ethereum-Mainnet document store
```
TXT     opencerts net=ethereum net-id=1 addr=0x007d40224f6562461633ccfbaffd359ebb2fc9ba
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

At this point, if we find a valid TXT record labelled 'opencerts', we can say that the document is issued from the domain. 

### Messaging

An appropriate message could be

```
Issued by openattestation.com
```

### Issues with using DNS

#### Non-repudiation

As DNS records can be altered by the controlling entity, the entity can remove the DNS record after issuing a document and there is no way to proof that they have issued the documents in the first place.

We found this issue to be trivial since the controlling entity should be in control of what constitute a valid or invalid document. 

This issue is also resolved if DNSSEC is used as records are signed.

#### DNS Expiry / Squatting
This will unfortunately be an unresolved issue if we use DNS, as DNS has no archival functionality. If an organisation is concerned about this, perhaps they should consider the DNSSEC-ENS registry method which will persist with the same lifetime constraints as the document store smart contract.

#### DNS Spoofing
[DNS spoofing](https://en.wikipedia.org/wiki/DNS_spoofing) is partially mitigated by using a well known DNS resolver such as Google or Cloudflare to retrieve the TXT records. However using DNSSEC can fully resolve the issue.

## Can other identity proofs be used?

```json
"issuers": [
    {
        "identityProof": {
            "type": "ENS-TXT",
            "domain": "tech.gov.sg.opencerts.eth"
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

If a centralised registry is used, it would have been hardcoded in the application using the centralised registry. It shouldn't be in the document created as it will not be useful to someone else that does not subscribe to the centralised registry. If they are subscribed to it, they will hardcode the registry as well. 

## Why tokenStore does not require identity proof

The `documentStore` alone will prove that issuer intended to create the document. It does not need to be conveyed again through a separate proof for `tokenStore`. 

In addition, this will allow an anonymity cloak to their tokenStores. Issuers can create multiple token stores, even to the extent of one store per token, to hide transaction histories of a document.

## Why isn't the DNS TXT signed by the document store owner
This signature would serve to show that the document store owner has endorsed the DNS-TXT record - however this is already shown by the fact that the issuer object in the certificate points to this DNS-TXT record, and was the certificate was issued by the document store.
