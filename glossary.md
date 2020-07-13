# Glossary of Terms

OpenAttestation: The underlying technology/framework that we use to power verifiable credentials

OpenCerts: The project that is owned by SSG and is concerned with educational credentials

TradeTrust: The project that is owned by IMDA and is concerned with trade documents as well as transferable asset deeds

### OpenAttestation Framework
  
  Envelope: Wrapper around a document that uses OpenAttestation framework to protect its integrity. _(Sometimes called Signed JSON file)_
  
  Proof: Part of the `Envelope` metadata, a set of Keccak256 hashes that leads to the Merkle root that is notarised (i.e on the Ethereum blockchain)
  
  Document: The information inside the .opencert or .tt file that excludes the `Envelope` metadata. _(Sometimes called Unsigned JSON)_
  
  Schema: Refers to a specific schema that the contained `Document` has to adhere to.
  
  Template: Instructions on how to display the data in the `Document`, it is currently an URL pointing to a `Renderer`. For OpenCerts, a simple path is used for v1.4 & v1.5, prior to that, a key is used.  
  
  Renderer: A HTTP(s) endpoint that takes a `Document` and returns some browser-renderable contents (i.e HTML/CSS/JS)
  
  RendererFrame: A component on the OpenCerts/OpenAttestation site that takes a `Document` and sends it to the `Renderer` endpoint.

### OpenCerts

  SSG: SkillsFuture Singapore, the owner of OpenCerts accreditation registry
  
  SSG Registry: A list of accredited institutes, administered by SSG _(Sometimes called The Registry or just Registry)_
  
  Issuer: An entity that issues educational credentials, represented by one or more DocumentStores.
  
  Certificate: .OpenCert file that contains a verifiable educational credential.
  
  DocumentStore: An Ethereum smart contract that keeps track of OpenCerts issuances and revocations.
  
  DocumentStore Address: The specific Ethereum Address that a `DocumentStore` is located at.
  
  Collaborator: A company that is listed on our Collaborators page, they can help people set up their systems to issue OpenCerts.
  

### TradeTrust
  <TODO>
