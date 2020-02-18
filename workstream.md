# Workstream Summary

## Developer Support

### Documentation for different developer personas

#### Managers

"I want to blockchain this document"

Examples of such persona:

- CEO of a company/agency
- Manager of an company/agency tasked to blockchain something
- Manager of an company/agency tasked to explore digital or signed documents solutions

Some specific problems from users in this group:

- What is blockchain?
- Why is blockchain better?
- Show me examples of OpenAttesttation documents?
- What is OpenAttestation?
- What are the key features of OpenAttestation documents?
- What's the difference between OpenAttestation with XYZ?
- What kind of infrastructure do I need?
- What resources are available for my developers?

#### Integrator

"I want to create OpenAttestation documents or transferable records"

Examples of such persona:

- Founder or CTO of blockchain startup
- System integrator of a company/agency
- Solutons architect of a company/agency
- Developers from the company's IT department

Some specific problems from users in this group:

- How do I issue my university degree?
- How do I issue a certificate of non-manipulation?
- How do I create a certificate with a QR code that loads the certificate in OpenCerts' website when scanned?
- How do I create a link on my web page to open my e-license on the mobile app?

#### Developer

"I want to build platform or services on top of OpenAttestation"

Examples of such persona:

- Founder/CTO/Developer of blockchain startup
- Developers from platform provider's IT department (ie Job Site/SAAS Solutions)
- Developers wanting to build extensions/modules to existing stacks

Some specific problems from users in this group:

- How do I verify the authenticity of a document with API?
- How do I let my other applications read the content of the education credentials?
- How can I view all OpenAttestation document on my website?
- How do I automatically issue document from my server?
- How do I build a custom verifier module for my mobile app?
- How do I build a custom identity association module for my KYC company?

#### Contributor

"I want to contribute to OpenAttestation"

Examples of such persona:

- Developers interested in identity related problems

Some specific problem form users in this group:

- How do I make OpenAttestation support JWT?
- How do I build CL-Signatures into OpenAttestation documents?
- How do I build an adapter from my solution to be compatible to OpenAttestation?
- I want to contribute to the roadmap and architecture of the project!

## Tools

### OpenAttestation CLI

- Generate sample document
- Deploy document store and token registry (with proxy option)
- Issue a document / create a token
- W3C VC commands

### Transferable Records Creator

- Generate document preview from a config & data file
- Create a transferable record token from the data
- Download the transferable record file

### Document Creator

- Generate document(s) preview from a config & data file
- Create a batch of document from the data
- Download the document files

## W3C Verifiable Claims Compatibility

- Join [W3C Credentials Community Group](https://www.w3.org/community/credentials/) [[Guide]](https://github.com/w3c-ccg/w3c-ccg.github.io/blob/master/joining.md)
- JWT backend support
- Create CLI in conformance to [W3C test suite](https://w3c.github.io/vc-test-suite/)
- Submit OA proof method in [VC Extension Registry](https://w3c-ccg.github.io/vc-extension-registry/)
- Submit as implementation to [VC Test Suite Implementation](https://w3c.github.io/vc-test-suite/implementations/)
