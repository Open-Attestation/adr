# OpenAttestation Verifier

## Problem

The current method to assess if a document has been issued correctly by multiple issuers is a bad design. The inability for oa-verify to validate issuers when there are multiple issuers using different types of verification method is causing dependant projects to cobble together bad workarounds around a fundamental problem. The current design does not allow for different issuers to use different methods of identifying themselves safely and as a result, we get ridiculous code like the ones below at dependant code to get around this failure:

```js
// using a custom isValid because @govtechsg/oa-verify will NOT throw an error when there are 2 identities
// with one skipped and one valid.
// in the case of Tradetrust, we want to make sure all identities are valid
export const isValid = (
  verificationFragments: VerificationFragment[],
  types: VerificationFragmentType[] = [
    "DOCUMENT_STATUS",
    "DOCUMENT_INTEGRITY",
    "ISSUER_IDENTITY",
  ]
) => {
  if (types.includes("ISSUER_IDENTITY")) {
    const dnsTxtFragment = getFirstFragmentFor(
      verificationFragments,
      "OpenAttestationDnsTxt"
    );
    const dnsDidFragment = getFirstFragmentFor(
      verificationFragments,
      "OpenAttestationDnsDid"
    );
    const isUsingDnsTxt = dnsTxtFragment && dnsTxtFragment.status === "VALID";
    const isUsingDnsDis = dnsDidFragment && dnsDidFragment.status === "VALID";
    const isAllIdentityValid =
      (isUsingDnsTxt || isUsingDnsDis) &&
      (isUsingDnsTxt
        ? dnsTxtFragment?.data?.every(
            (issuer: VerificationFragment) => issuer.status === "VALID"
          )
        : true) &&
      (isUsingDnsDis
        ? dnsDidFragment?.data?.every(
            (issuer: VerificationFragment) => issuer.status === "VALID"
          )
        : true);
    return (
      isValidFromUpstream(verificationFragments, types) && isAllIdentityValid
    );
  } else {
    return isValidFromUpstream(verificationFragments, types);
  }
};
```

## Understanding the problem

The problem with the current approach is that individual identity verifier fragments do not know the context of other fragments.

Suppose we have document 1 with the following issuers:

1. DNS-TXT (valid: tech.gov.sg)
1. DNS-DID (valid: imda.gov.sg)
1. (Issuer without any corresponding verification mechanism)

and document 2 with the following issuers:

1. DNS-TXT (valid: tech.gov.sg)
1. DNS-DID (valid: imda.gov.sg)

The current default oa-verify will allow both document 1 & 2 to be valid, but in actual fact, document 1 should have been invalid since we are not able to verify ALL of the listed issuers. Instead, it is returning VALID for overall status because:

The dns-txt output will look like:

1. VALID
1. SKIPPED
1. SKIPPED

with overall status = VALID

The dns-did output will look like:

1. SKIPPED
1. VALID
1. SKIPPED

with overall status = VALID

## Proposed Solution

A better design that makes sense is have one single identity verifier fragment that is composable. A pseudocode for the fragment will look like:

```txt
for issuer in issuers in document:
    verifier = get_corresponding_verifier(issuer)
    result = verifier.verify(issuer, document)
    verification_data.push(result)

overall_status = every verification_data is valid
```

This will also provide a way for each verifiable method to recommend an identifier once it has been ran, instead of relying on a fixed `location` key.

The result will oa-verify exporting additional identityVerificationBuilder to allow dependant projects to customize that fragment. Alternatively, dependant projects can simply use the default verifier with the default identity verification fragment.

```js
const identityVerificationModule = identityVerificationBuilder([
  dnsTxtIdentityMethod,
  dnsDidIdentityMethod,
]);
const verify = verificationBuilder([...xxxxx, identityVerificationModule]);
```

## Alternative Solution

Since this problem will go away with V3 of OA, another solution is to maintain the current design of the identity verification fragments and have one stricter condition, which is that any document should have only at most one type of identity resolution (ie. it's using only DNS-TXT or DNS-DID).

Within each verification fragment, instead of skipping on unknown method, immediately return "INVALID".

This will result in document 2 described above to fail resulting in a false negative but it will not propagate the problem of the oa-verify's inability to verify to dependant projects.
