# Building the Document Creator

## Goal

To define the steps to take to create a configurable document creator web application that one can use to create a valid OpenAttestation document (both transferable records and verifiable document).

We assume a configuration file generated prior to the form creation stage is available to this user. Technical details of the configuration file can be found [here](./configurable_dapp_usability.md)

The approach is an iterative agile development where the stories are organized by the most amount of value to be delivered first.

In the following stories the `user` refers to an administrative clerk with no programming experience and but is familiar with working with SASS products.

## 1. Configuration file dropzone

As a user, I want to be able to load my company's forms into the web application. I do this by dropping the config file my IT department gave me into the dropzone and entering my password to unlock the configuration.

Acceptance Criteria:

- dropzone to drop my config file
- upon dropping the config file, it should prompt me for my password
- with a wrong password, it should prompt me to try again with the wrong password error
- with a correct password, it should show me a raw display to show that the config file has been sucessfully read

The raw display should have:

- types of forms available (array of string)
- my wallet address

## 2. Document selector

As a user, after dropping my config file, I want to be able to select the types of documents that I can create. I do this by clicking on the button with document type. When I do that, it will show me the fields available on the form.

Acceptance Criteria:

- display buttons with the types of documents that can be created by user
- upon clicking the button, it should show me a raw display to show that the correct form has been selected

The raw display should have:

- name of forms
- fields of forms (array of string)

## 3. Dynamic Form

As a user, after selecting the type of form for creating documents, I want to be able to fill in the forms where the fields are defined by my IT department. After I do that and click next, it will show me a preview of what I've created.

Acceptance Criteria:

- dynamically generated form based on the config file and the selected form
- upon clicking on next, it should show me the a raw display of the document to be created

The raw display will simply be the raw OpenAttestation document that has yet to be batched.

## 4. Document Creation Status

As a user, after filling in the form, I want the OpenAttestation document to be created and ready to be sent out using other medium like an email. Upon clicking on next, the application will show the creation status and upon successful creation, it will allow me to download the document created.

Acceptance Criteria:

- upon clicking on next on the dynamic form, it will show a loading animation
- once the document has been created, it should allow me to download the document
- the document should be able to be verified on dev.tradetrust.io or tradetrust.io (if issued on main net)

---

At this point, the minimal viable product is created and has delivered most of the value. The stories below build upon this foundation to create a more fully featured application.

---

## Form data file

TBD

## Document Preview

TBD

## Attachments

TBD

## Batch Document Creation

TBD
