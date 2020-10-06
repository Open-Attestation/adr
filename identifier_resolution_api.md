# 3rd Party Identifier Resolver API

This document specify the specifications for the `3rd Party Identifier Resolver API` documented in the [Identifier Resolution Framework](./identifier_resolution_framework.md)

## Status

Draft

## Goal

Different entities may choose to use different pseudonyms (in our case Ethereum addresses), some of these identifiers are reused and some are not. For those entities who chose to reuse a pseudonym, they may want wish for these resources to be identified. Examples of such resources could be a shipping line wallet, multi-sig wallet or eBL token registry.

As such there is a reason to believe that there are likely multiple source of identifier registry, some examples are:

- A company's internal address book shared amongst all it's employee
- A KYC company supporting multiple companies in identifying known resource pseudonyms
- A country's providing digital infrastructure to bind companies to their online pseudonyms
- An infrastructure accompanying a bilateral agreement between two countries for their businesses to identify each other

The purpose of the POC is to experiment with a spec which supports the above use cases where an Ethereum address could be resolved to an entity name. It is not within the scope of the POC to determine:

- If the resolution is correct
- How the resolution happens
- How the external identity framework is setup

The overarching principle to the decisions above is to allow end user to choose who to trust and leverage on that entity's knowledge. Should a trustless infrastructure be available, consider extending the general verification method instead.

## Implementation

The POC will provide a API spec to be included in the identifier resolution framework and a sample implementation which conforms to the spec proposed.

## What should the spec specify

1. When sent an identifier, return a resolved entity name if it resolved or return an error when it is not found
2. Optionally, provide a reason or remark on how the resolution happens
3. Optionally, provide basic authentication mechanism to allow only authorized parties to query

## What should the spec not specify

1. Specify how the resolution happens
2. How entires can be inserted or deleted
3. Authorisation
4. Business model of the API owner

---

## API Specification

### GET /\<identifier\>

This is the endpoint that will be used to resolve an identifier. For instance to resolve the identity of `0x0103e04ecaa67c4e5a8c6dc1ddda35340e2c6bc8`, we will call the endpoint `https://demo-resolver.tradetrust.io/identifier/0x0103e04ecaa67c4e5a8c6dc1ddda35340e2c6bc8` where `https://demo-resolver.tradetrust.io/identifier/` is the base url.

#### Successful Resolution

For successful resolution, the API must return a `200` response in `application/json` format. The message must contain an `identity` object with the `identifier` and `name` key. Additional keys may be added to the `identity` object.

```json
{
  "identity": {
    "identifier": "0x0103e04ecaa67c4e5a8c6dc1ddda35340e2c6bc8",
    "name": "ABC Pte Ltd",
    "remarks": "Added by Raymond"
  }
}
```

#### Unsuccessful Resolution

For unsuccessful resolution, te API must return a `404` response in `application/json` format. The message may contain a `message` key to explain the error. Additional keys are also permitted.

```json
{
  "requestId": "db98db33-5c58-4f3e-a740-b855e4232230",
  "message": "No profile found for 0x0103e04ecaa67c4e5a8c6dc1ddda35340e2c6b88"
}
```

## Reference Implementation

A reference implementation of this service can be found at https://github.com/Open-Attestation/demo-identifier-resolver.

The hosted endpoint is available at https://demo-resolver.tradetrust.io/identifier.

Successful Resolution:
https://demo-resolver.tradetrust.io/identifier/0x0103e04ecaa67c4e5a8c6dc1ddda35340e2c6bc8

Failed Resolution:
https://demo-resolver.tradetrust.io/identifier/0x0103e04ecaa67c4e5a8c6dc1ddda35340e2c6b88

## TBD

Include reference implementation and specs for protected endpoint with API keys.