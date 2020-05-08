# Goal

Following discussion from [previous discussion on identifer_resolver_integration](./identifier_resolver_integration.md), we want to explore other methods which 3rd parties **with traditional PKI** can partake in either providing identity proofs or identifier resolution.

## (recap) Distinction between identity proof and identifier resolution

Identity proofs are proofs that can be independently verified by the end users.

Identifier resolutions are simple identifier (in most case Ethereum addresses) to another identifier that need not require the trustless property.

## Different workstreams

### 1. Identifier Resolution

The 3rd party can contribute to identifier resolution where it resolves a specific Ethereum address to another identifier, for instance domain name, DN or BIC. Using this method, OA does not get involved with how the identifier is being resolved or it's correctness.

### 2. Identity Projection from PKI

In this workstream we explore how we can anchor a wallet or smart contract's off-chain identity to traditional PKI certificates. With this, it means anyone can independently verify the identity of the smart contract owner.

There are multiple approach under this workstream which will be discussed in detail below.

### 3. Certificate signing as OpenAttestation backend

Previous implementations of OpenAttestation make use of the Ethereum backend where either a document store or token registry is used to store the document status. Using this approach, we can directly sign on the document rather than relying on the Ethereum backend

## Analysis of different workstream

### Identifier Resolution

Described in [Identifier Resolution Framework](./identifier_resolution_framework.md)

Pros

- Simple for user to set up, endpoint + api key (optional)
- TradeTrust will have the feature soon
- API provider can customize access controls
- Each provider can choose how they like to implement their resolvers

Cons

- Missing non-repudiation property (endpoint can be non-deterministic)

Info

- User will trust the endpoint provider completely

### Direct Certificate Signing

Using this method, we will sign directly on the merkle root of the document and append the proof in the document itself.

Pros

- Issuers with certificates like SSL or those from private CAs can directly use it without Ethereum
- Faster and cheaper than using Ethereum

Cons

- Substantial work on OA framework
- Issued documents cannot be revoked without revocation list
- Signing key will be used frequently
- User will have to setup root CA certificates

Info

- SSL does not provide a stronger identity guarantee compared to DNS-TXT

### Identity Projection using Standard Identity Smart Contract

This method will use ideas from the [previous discussion](./identifier_resolver_integration.md)

Pros

- Standard way for users who have certificates from CA to use them as identity

Cons

- High setup cost on document issuer (smart contract)
- Maintenance and custodian to the identity smart contract and it's standard
- Standards war
- Does not reach audiences beyond those using private PKI (SSL users won't use this as it does not provide stronger guarantee)

### Identity Projection using DID

This method works by extending the existing DID implementation to use standard PKI certificates

Option 1 - Develop proprietary resolver method (ie did:swift:<BIC/URI_TO_CERT>)
Option 2 - Develop standard resolver for all x.509 certificates (ie did:x509:<URI_TO_CERT>)

Pros

- Decoupling of identity & identifiers (DID) from document verification (OpenAttestation)
- OpenAttestation plugged into other DID methods as well when using universal resolver
- Identity provider plugged into other applications consuming the identifiers
- Independent development of modules against standard specification

Cons

- DID is not exactly meant for centralized identity infrastructures
- End users will still need to setup root cert

## Recommendation

## Short term - Identifier resolution

In the short term, we should allow any 3rd party who are interested and able to provide the resolver endpoint for us to establish a POC against our existing workflow for user using transferable records. The specification is almost ready with only the authentication missing. The TradeTrust website is also almost ready with the feature for custom endpoints. Once any third party is ready with a sample endpoint we can begin testing.

## Long term - DID integration

In the long term, we want to be able to integrate with multiple types of identifier. The ongoing effort at DIF for DID has shown to be promising and the specifications are also standardized at the W3C level. We should avoid creating or act as custodian of standards related to identity as much as possible. In this case, having interested parties be plugged into DID and having our applications utilize DID will enable both parties to be entirely decoupled from each other use cases. Substantial work has to be done on OpenAttestation level to enable that. Nonetheless it will be meaningful work that brings us closer to standard formats.
