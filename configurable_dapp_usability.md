###### Authors:

Raymond Yeh

# Configurable Dapp for Better Usability

## Problem Statement

The barrier to entry for DApps is currently very high for users who are not familiar with how blockchain and DApp works. The extra cognitive workload results in poor user experience for the majority of people.

Take for instance the DApp for the [OpenCerts Admin Portal](https://admin.opencerts.io), users are required to know the difference between their own wallet address, the document store address and what document merkle root to issue. A typical user would have expected a username & password as login and not have to to configure those egnimatic paramters.

How might we make DApp more accessible to the general audience while not compromising on the benefits of decentralisation?

## Configuration Files for Users

To lower the cognitive workload of the administrative personnel, we could hide the configurations behind a custom config file. This allows a standard DApp to be created for different types of organisation and functions. The configuration file can be used for various purposes such as:

- Access control
- Default values
- Dynamic form
- Hooks/callbacks
- Address books
- Key storage

### Access Control by Hiding Actions

If a user does not need (or do not have permission) to access certain functionalities or modules in a Dapp, we
may want to hide some of the functionalities.

Take for instance a clerk who is tasked to create TradeTrust document, because he does not need to see the revocation module (while his key may have permission to do so), we only show him the "Create Document" feature.

This method cannot be used as access control in traditional sense as it does not prevent the user from interacting with the smart contract directly. Should actual access control is needed, it should be represented as code in a smart contract.

Examples of access control config:

```
modules: [
    "create"
]
```

### Default Values

During the creation of documents, we often need to populate default values such as the `documentStore` address. Since the administrative staff is expected to always enter the same value for all the documents issued, it make sense to configure them as defaults.

Example of default value config:

```
default: [
    issuers[0].name: "DEMO STORE",
    issuers[0].documentStore: "0x2f60375e8144e16Adf1979936301D8341D58C36C",
    issuers[0].identityProof.type: "DNS-TXT",
    issuers[0].identityProof.location: "example.openattestation.com",
    certificateSignature: "<BASE64-IMG>"
]
```

### Dynamic Forms

If a clerk is expected to perform data entry (as oppose to dropping documents as attachments), a form can be created dynamically from a configuration file to:

1. Render a form
2. Transform inputs from a form into structured data

This presents a familiar interface for the clerk to perform data entry on a form.

Some dynamic form generator: https://react.rocks/tag/FormGenerator

Example of dynamic form (using react-jsonschema):

```
forms: [{
    id: "invoice",
    name: "Invoice"
    schema: <JSON-SCHEMA>,
    ui: <UI-SCHEMA>
},{
    id: "bill-lading",
    name: "Bill of lading"
    schema: <JSON-SCHEMA>,
    ui: <UI-SCHEMA>
}]
```

### Hooks/Callbacks

Hooks can be added to represent workflow management. The Dapp can be configured to fire certain endpoints upon the success of different actions. For instance, upon issuing a trade document to a company, it instantly calls the endpoint of the company to send them the data of the issued document. The counterparty can then choose how it intend to process the document. For instance, if it's a PASS app, it can automatically place the document in a document repository or even automatically process the document.

Example of hooks:

```
hooks: {
    invoice: {
        issued: "https://tradetrust.abcbank.com/trader-123/<API-KEY>/issued"
    }
}
```

### Address Books

Often a business has an address book of common values to be used for different business units they interact with. The address book can have values like:

1. Smart contract address of counter party
2. Default values of counter party (ie. Company names, address, etc)
3. Endpoints of counter party (ie callback/hooks after a transaction is completed)

This can be implemented with a dropdown selector or autocomplete searchbox.

Example of address book:

```
addressBook: [{
    name: "ABC Bank",
    wallet: "0x12...34",
    default: [
        certificate.toName: "ABC Bank",
        certificate.toAddress: "ABC Bank Addr"
    ],
    hooks: {
        invoice: {
            issued: "https://tradetrust.abcbank.com/<my-identifier>/<API-KEY>/issued"
        }
    }
}]
```

### Key Storage

While storing key in a config file (or key file) in this case is may not be sound, it is important to take note that the key should not be an all-access key in the first place. If there are plans in place to protect against lost keys, we could look at providing a more familiar interface to the user where they will enter a password to "login" to the "account".

```
wallet: {
    address: "0xab..cd",
    encryptedKey: <enc-private-key>
}
```

## Additional Note

Additionally, the config file with all the above values could be stored in the browser's local storage to avoid having the user configure the application every time he uses it. 