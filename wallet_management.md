# Wallet Management Solutions
Author: RJ

Status: Draft

Published: 18/01/2021

## Existing State of TradeTrust (and OpenCerts)
We currently support a mixture of browser storage (Metamask), consumer-grade hardware wallets and encrypted keystore files across our products. Any of our website that uses Metamask will automatically support the hardware wallets manufactured by Ledger and Trezor because Metamask allows the user to use these wallets through Metamask. 


### creator.tradetrust.io 
We only allow users to issue using a pre-setup configuration file which contains the Ethereum wallet using the [encrypted keystore format](https://kb.myetherwallet.com/en/security-and-privacy/what-is-a-keystore-file/). This wallet is password protected using a password that is entered after the user provides the configuration file to the web application.

### tradetrust.io
On tradetrust.io we support the Metamask browser extension, which allows the user to use either browser storage or a [hardware wallet](https://metamask.zendesk.com/hc/en-us/articles/360020394612-How-to-connect-a-Trezor-or-Ledger-Hardware-Wallet) (Ledger or Trezor) to perform Ethereum transactions. This is only required for transfer of asset owner/holdership.

### opencerts.io
Ethereum wallet is not required on this website as verification can take place without one.

### admin.opencerts.io
The user interface on this web application allows the user to select between Metamask or Ledger Nano. This application has a large amount of legacy code, and it is uncertain if the "Ledger" menu option still works - we have received some reports that it does not work, but upon investigation by our team it appears to work with a very poor UX on Windows. Using the Metamask option is feasible, and allows the user to use both browser stored wallets and consumer grade hardware wallets.


## Survey of key management solutions 

### Proven 1st Party solutions
- Encrypted Ethereum keystore JSON wallet format
- Browser Extension: Metamask
- Consumer grade hardware wallet: Ledger + Metamask

### Incomplete or WIP solutions 
- AWS KMS / CloudHSM - AWS KMS is a mature solution, AWS KMS + Ethereum might have some hickups but should not be anything catastrophic. Will require some work to support with our existing software. Uncertain about CloudHSM since it is an expensive solution and you really only need it if you are running very high stakes stuff like crypto exchanges or certificate authorities etc
- Azure Key Vault - Same as AWS CloudHSM
- Hashicorp Vault - Looking at the Github issues it does not look very ready for production use, and also having to run a whole Hashicorp Vault installation will require Ops personnel
- Thales SafeNet Luna HSM - [Luna HSM](https://cpl.thalesgroup.com/sites/default/files/content/integration_guides/field_document/2020-06/007-000189-001_EthereumBlockchain_SafeNetLunaHSM_IntegrationGuide_RevA.pdf) - On-premise hardware solution. Will require custom development to integrate with this, not feasible for IMDA/Govtech to support this effort - if there is a requirement for this level of security they should do their own development and security audits on top of that

### 3rd Party Solutions
[Torus Labs - Flexible, Universal Key Management](https://tor.us/)

[Onboard.js — Easily support the top wallets](https://www.blocknative.com/onboard)

These providers require some work to integrate onto our site but will require payment and is a form of centralisation. Usage also means that TradeTrust/IMDA/GovTech implicitly endorses them, which could be problematic. The upside is that they support numerous types of wallet storage with a good user experience - especially for instance of lost keys.



## Possible avenues of improvement

### AWS KMS 
Writing a shim to use Ethers.js with AWS KMS => just need to implement the signing methods [ethers.js/ledger.ts at master · ethers-io/ethers.js · GitHub](https://github.com/ethers-io/ethers.js/blob/master/packages/hardware-wallets/src.ts/ledger.ts) , [legwork on using aws kms with Ethereum](https://luhenning.medium.com/the-dark-side-of-the-elliptic-curve-signing-ethereum-transactions-with-aws-kms-in-javascript-83610d9a6f81))
This piece of work would allow users to use an Ethereum wallet stored on AWS.

Further work on this could be to write a managed proxy to AWS KMS that only allows transactions that pass certain criteria such as pre-defined transaction parameters (only allowed to issue documents, not send Eth) or contract address

### Signing proxy for creator.tradetrust.io
A Ethereum node connection details could be specified in the configuration file, allowing it to connect to a node that has the ability to do signing. Could be something like [EthSigner Transaction Signer](https://docs.ethsigner.consensys.net/en/stable/). Perhaps secured with oauth login or AWS api key signed requests.


## References

Metamask supports:
  Ledger [Hardware Wallet - State-of-the-art security for crypto assets \| Ledger](https://www.ledger.com/)
  Trezor [Trezor Hardware Wallet (Official) \| The original and most secure hardware wallet.](https://trezor.io/)
  
  
Ethers.JS supports:
  Metamask
  Ledger [ethers.js/packages/hardware-wallets at master · ethers-io/ethers.js · GitHub](https://github.com/ethers-io/ethers.js/tree/master/packages/hardware-wallets)
 
 
 Azure Key Vault HSM
 [Simple Ethereum Wallets Management with Azure Key Vault \| by Itay Podhajcer | Microsoft Azure | Dec, 2020 | Medium](https://medium.com/microsoftazure/simple-ethereum-wallets-management-with-azure-key-vault-2b701bc0505)
 
 Hashicorp Vault
 [Using Vault to Build an Ethereum Wallet](https://www.hashicorp.com/blog/using-vault-to-build-an-ethereum-wallet)
 
 AWS key management service (KMS) as ethereum wallet
 [The Dark Side of the Elliptic Curve - Signing Ethereum Transactions with AWS KMS in JavaScript \| by Lucas Henning | Medium](https://luhenning.medium.com/the-dark-side-of-the-elliptic-curve-signing-ethereum-transactions-with-aws-kms-in-javascript-83610d9a6f81)
 
 AWS CloudHSM
 [AWS CloudHSM - Knowledge Center](https://docs.kaleido.io/kaleido-services/cloudhsm/aws-cloudhsm/)
 

[Overview - EthSigner](https://docs.ethsigner.consensys.net/en/stable/Concepts/Overview/) : This project seems to implement a proxy for signing transactions with a configurable wallet store, but not sure how mature it is or how supported it is.