###### Authors:
Adam Lemmon [<@adamjlemmon>](https://github.com/AdamJLemmon)

# W3C-DID Issuer Identity Proof
This document describes an identity proof method leveraging Decentralized Identifiers as formally specified
by the W3C: [Decentralized Identifiers (DIDs)](https://www.w3.org/TR/did-core/)

The primary objectives of this identity proof are: 
* Allow an issuer to be identified and verified with a DID
* Minimize changes to the existing issuer workflow
* Enable support for many [DID Methods](https://www.w3.org/TR/did-core/#did-methods)
* Enable extensibility for rich issuer identification and verification schemes

__W3C-DID Identity Proof Example:__
```json
	...
	identityProof {
		type: “W3C-DID”,
		location: “did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a”
	}
	...
```

One of the primary concerns of any document verifier is determining _who_ issued the given document.
This proof type allows document issuers to be identified and verified securely through the use of a DID.

A DID serves as a URI as defined by the IETF generic URI specification [RFC3986](https://tools.ietf.org/html/rfc3986). 
A DID may be resolved in order to obtain issuer information, cryptographic materials and other data to identify and verify the given document issuer.

## Using DIDs to Verify the Identity of an OpenAttestation Document Issuer
Within a given document the issuer's identity may be identifed with a DID. 
For example: `did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a`

The DID serves as a URI where information to both _identify_ and _verify_ the issuer may be obtained.

To be specific, each DID resolves to a DIDDocument, a JSON-LD document, as formally specified within the W3C DID Specification: [DID Documents](https://www.w3.org/TR/did-core/#did-documents)

For example the resulting DIDDocument for the above DID, `did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a`, is:
```json
"didDocument":{
	"@context" : "https://w3id.org/did/v1",
	"id" : "did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a",
		"authentication" : [{
		"type" : "Secp256k1SignatureAuthentication2018",
		"publicKey" : [ "did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner" ]
	}],
	"publicKey" : [ {
		"id" : "did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner",
		"type" : "Secp256k1VerificationKey2018",
		"ethereumAddress" : "0xb9c5714089478a327f09197987f16f9e5d936e8a",
		"owner" : "did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a"
	}]
}
```

To establish this proof, the issuer does not need to do anything new :)

This is made possible by leveraging the `did-ethr` DID Method which is specified here: [ETHR DID Method Specification](https://github.com/decentralized-identity/ethr-did-resolver/blob/develop/doc/did-method-spec.md)

In summary, the existing Ethereum externally owned account(EOA) that has been used to issue documents within the given issuer's `documentStore` is already a fully supported DID simply by prepending `did:ethr:` to the address.  By design DIDs are not required to be formally *registered* with any auhtority, registrar or ledger and will map to a DIDDocument as above by default.

### Example Verification Workflow
*Note this example assumes that a documentStore has been deployed on the Ethereum Mainnet and the document to be verified has been issued within that documentStore.*

Given an issued document containing the following (*truncated for simplicity*):
```json
{
	...
	proof: {
		type: "...:OpenAttestationSignature2018",
		method: "...:DOCUMENT_STORE",
		value: "...:0x8Fc57204c35fb9317D91285eF52D6b892EC08cD3"
	},
	issuer: {
		...
		identityProof: {
			type: "...:W3C-DID",
			location: "...:did:ethr:0xB26B4941941C51a4885E5B7D3A1B861E54405f90"
		}
	}
	...
}
```

1. **Resolve the DID to its DIDDocument**
	- **DID Resolution:** The process of resolving a given DID to its DIDDocument. DID Resolution is highly dependent on the specific [DID Method](https://www.w3.org/TR/did-core/#did-methods) being used. This example will make use of the `ethr` DID Method as identifed in the above DID: did:`ethr`:0xB26B4941941C51a4885E5B7D3A1B861E54405f90
	- In order to satisfy our core objectives and enable support for many DID Methods the [Decentralized Identity Foundation](https://identity.foundation/)'s [Universal Resolver](https://github.com/decentralized-identity/universal-resolver) was chosen as the core utility for DID Resolution.
	- An instance of the Universal Resolver is publicly hosted at [https://uniresolver.io/](https://uniresolver.io/) and a REST API for DID Resolution is exposed at [https://uniresolver.io/1.0/identifiers](https://uniresolver.io/1.0/identifiers)
	- DIDs may be resolved simply by appending the DID to the API path. 
	For Example:
		```sh
		curl -X GET https://uniresolver.io/1.0/identifiers/did:ethr:0xB26B4941941C51a4885E5B7D3A1B861E54405f90
		curl -X GET https://uniresolver.io/1.0/identifiers/did:sov:WRfXPg8dantKVubE3HX8pw
		```
	- The response from the Universal Resolver is the associated DIDDocument... and some metadata, but that has been omitted for simplicity:
	For Example:
    	```json
    	{
    		'@context': 'https://w3id.org/did/v1',
    		id: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a',
    		publicKey: [{
    			id: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner',
    			type: 'Secp256k1VerificationKey2018',
    			owner: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a',
    			ethereumAddress: '0xb9c5714089478a327f09197987f16f9e5d936e8a'
    		}],
    		authentication: [{
    			type: 'Secp256k1SignatureAuthentication2018',
    			publicKey: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner'
    		}]
    	}
    	```
2. **Issuer Verification**
	- Ownership of DIDs, or DID Auth, is not a simple task and is also highly dependent on the DID Method. A detailed write up on DID Auth may be found here: [Introduction to DID Auth](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/did-auth.md)
	- The retrieved DIDDocument effectively communicates information about how ownership of the DID can be proven as noted within the `authentication` array.
	For example:
    	```json
    	authentication: [{
    		type: 'Secp256k1SignatureAuthentication2018',
    		publicKey: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner'
    	}]
    	```
	- In this example the only authentication method is of type: `Secp256k1SignatureAuthentication2018` with `publicKey: did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner` and therefore we must define a way to verify the issuer with such.
	- The `publicKey` is in fact a URL itself that locates a property within the DIDDocument.  In this case locating the following:
    	```json
    	{
    		id: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a#owner',
    		type: 'Secp256k1VerificationKey2018',
    		owner: 'did:ethr:0xb9c5714089478a327f09197987f16f9e5d936e8a',
    		ethereumAddress: '0xb9c5714089478a327f09197987f16f9e5d936e8a'
    	}
    	```
	- This presents the opportunity to verify the issuer against the `ethereumAddress` associated with this `publicKey`.
	  From above: `ethereumAddress: 0xb9c5714089478a327f09197987f16f9e5d936e8a`
	- In order to verify the issuer we need some proof that the issuer actually controls this address.
	- If the `tx.origin` that sent the transaction to issue the document in question matches the `ethereumAddress` associated with the `publicKey` the issuer may be verified.
	- The `tx.origin` may be obtained by querying the blockchain event logs for the `DocumentIssued` event emitted from the `documentStore` filtered with the given `merkleRoot`.  The details of this query are outside the scope of this document but this will result in returning the Ethereum account address that signed the transaction.
	- If this address matches that of the `publicKey` above the issuer is verified.

### Benefits
At first glance integrating a DID as an identity proof in this manner may seem to be of little value as ownership of the given Ethereum Address is already proven through transaction authoring. However, the value is derived from being able to prove ownership over the DID and associated DIDDocument.

Cryptographically verifying ownership over a DID, and its associated DIDDocument, can potentially lead to several benefits such as:
* Rich identification and verification schemes... for example in high assurance situations where certain capabilities or credentials are required
	* Various attributes providing greater detail about the issuer's identity may be included in the DIDDocument
	* Verifiable credentials or other data / proofs may be be exchanged to correlate further data about the given issuer's identity
	* Granular capability verification
* Key rotation and support for extended authentication methods. For example even leveraging existing OpenAttestation identityProofs such DNS-TXT as an authentication method within the given DIDDocument.
* Delegation to other DIDs or key pairs
* Support for many other DID Methods, such as those defined within the [W3C DID Method Registry](https://w3c-ccg.github.io/did-method-registry/#the-registry)
* Integration of service endpoints for further discovery, authentication, authorization, or interaction
* DIDs / key pairs may be generated and utilized independently of any central authority. No dependence on certificate authorities, registrars or the like.
* General extensibility

### Further Considerations and Issues
* __ETHR DID Auth__
	* Enhanced `ethr` based verification.
		* Is the `tx.origin` of the document issuing transaction the best approach?
		* What happens in the case of a multi-sig?
		* Is the owner of the `documentStore` contract useful?
* __DID Auth__
	* Support for other DID Methods and authentication methods
	* Ideally non-interactive with issuers
	* Include a formal signature as part of the document. Signature over the `targetHash`, would then not be required to query the `documentStore` / blockchain at all to verify ownership of the DID.  This could also lead to further support for key types and verifcation levergaing the same workflow.
* __How to handle DIDDocument changes?__
	* If the key that issued the document is removed from Auth or removed altogether, what happens to the verification of the document?
	* If using the owner of the contract not the sender of the issuer transaction, this could perhaps enable key rotation without any effect to the validity of the document. For example update DIDDocument keys and then change the owner of the `documentStore`.
* __DID Resolution__
	* Do we want to host our own universal resolver instance?
	* Use the ethr-did-resolver lib and did-resolver lib directly… 
* __Further Verification Flows__
	* Presentation of a VC to prove registered issuer for example or require presentation of capabilities
	* Extend DIDs to recipient identity as well

### Resources
* [W3C Decentralized Identifiers (DIDs)](https://www.w3.org/TR/did-core/)
* [IETF generic URI specification RFC3986](https://tools.ietf.org/html/rfc3986)
* [DID Documents](https://www.w3.org/TR/did-core/#did-documents)
* [ETHR DID Method Specification](https://github.com/decentralized-identity/ethr-did-resolver/blob/develop/doc/did-method-spec.md)
* [DID Method](https://www.w3.org/TR/did-core/#did-methods)
* [Decentralized Identity Foundation](https://identity.foundation/)
* [Universal Resolver](https://github.com/decentralized-identity/universal-resolver)
* [uniresolver.io/](https://uniresolver.io/)
* [Universal Resolver API](https://uniresolver.io/1.0/identifiers)
* [Introduction to DID Auth](https://github.com/WebOfTrustInfo/rwot6-santabarbara/blob/master/final-documents/did-auth.md)
* [W3C DID Method Registry](https://w3c-ccg.github.io/did-method-registry/#the-registry)
