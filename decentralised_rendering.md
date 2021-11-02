# Decentralised Document Rendering

## Status

Accepted

## Goal

![Overview](./assets/decentralised_renderer/overview.png)

The purpose of the decentralised document renderer is to allow OpenAttestation (OA) document issuers to style their documents without code change to the different implementations of the document viewer. It does so by embedding the website specified by the document issuers as an iframe or webview (for mobile apps) and sending the content of the OA document into the iframe.

The document viewer and the document renderer will rely on [postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) for the communication, and more specifically OpenAttestation rely on [penpal](https://github.com/Aaronius/penpal), a promise-based library for securely communicating with iframes via postMessage.

## Connection
`Penpal` is responsible for establishing the connection between the document viewer and the document renderer. To initiate the connection, the document viewer will load the document renderer into an iframe, using the `$template.url` from the OA certificate.

The message sent by penpal to establish the connection is out-of-scope of this document. However to establish the connection correctly, few things must be mentionned:
- The document viewer and the document renderer must both provide one method only, called `dispatch`. This method is the only used for the communication of any `Actions` (see below for a description of the different actions)
- The document viewer may have to support multiplt version of penpal. Indeed, there were a breaking change between penpal v4 and penpal v5, making [the version incompatible](https://github.com/Aaronius/penpal/issues/52). You can also decide to support a specific version of penpal. It's up to the implementer to decide.
- The document renderer just need to conform to the version used by the document viewer. We advise to use penpal >= 5.



## Communication

![API](./assets/decentralised_renderer/api.png)

Since the decentralised renderer is rendering a document dynamically, `Actions` are used to two-way communication between the parent frame (document-renderer.com) and the child frame (renderer.example.com).

All actions follow the same structure and are composed of type and payload

- `type` indicates the kind of action executed, for instance, RENDER_DOCUMENT to render a document. The type of action is mandatory
- `payload` indicates optional data associated with the type, for instance, the content of the document to render.

Examples of actions used:

```js
const renderDocumentAction = {
  type: "RENDER_DOCUMENT",
  payload: {
    document: documentToRender
  }
};

const printAction = {
  type: "PRINT"
};
```

### From host to frame actions (document viewer to document renderer)

The following list of actions are made for the host to communicate to the iframe (and thus must be handled by application embed in the iframe):

#### RENDER_DOCUMENT

This action sends the document data to the child frame to be rendered. This is usually the first action that is called after the connection has been established.

The payload is an object with 2 properties:

- document: (mandatory) document data as returned by getData method from @govtechsg/open-attestation
- rawDocument: (optional) Open Attestation document

Example:

```js
const action = {
  type: "RENDER_DOCUMENT",
  payload: {
    document: getData(document),
    rawDocument: document
  }
};
```

#### SELECT_TEMPLATE

This action selects a template amongst the one provided by the decentralized renderer (A renderer may provide 1 to many different templates to display a document).

The payload is the tab id to display.

Example:

```js
const action = {
  type: "SELECT_TEMPLATE",
  payload: "CUSTOM_TEMPLATE"
};
```

#### PRINT

This action request for the template to process the print action by the browser. It gives full control to the renderer to build the look and feel of the document when printed. With this, we ensure no interaction with the document viewer.

Example:

```js
const action = {
type: "PRINT"
const action = {
  type: "PRINT"
};
```

#### GET_TEMPLATES

This action returns the list of template tabs available on the document.

The payload is the document data as returned by `getData` method from @govtechsg/open-attestation

Example:

```js
const action = {
  type: "GET_TEMPLATES",
  payload: getData(document)
};
```

## From frame to host actions (document renderer to document viewer)

The following list of actions are made for iframe to communicate to the host (and thus must be handled by application embedding the iframe):

### UPDATE_HEIGHT

This action provides the full content height of the iframe so that the host can adapt the automatically the size of the embedded iframe. It's worht to note that we hope to get rid of ths action one day. It was created only because we had difficulties to size correctly the iframe from the document viewer. IF you know a way to get rid of this, please get in touch with us, we will be happy to improve our architecture.

The payload is the full content height of the child iframe.

Example:

```js
const action = {
  type: "UPDATE_HEIGHT",
  payload: 150
};
```

### OBFUSCATE

This action provides the name of a field on the document to obfuscate. The value must follow path property.

The payload is the path to the field to be obfuscated.

Example:

```js
const action = {
  type: "OBFUSCATE",
  payload: "a[0].b.c"
};
```

### UPDATE_TEMPLATES

This action provides the list of templates that can be used to render a document. This is usually the response to the `GET_TEMPLATES` action.

Example:

```js
const action = {
  type: "UPDATE_TEMPLATES",
  payload: [
    {
      id: "certificate",
      label: "Certificate"
    },
    {
      id: "transcript",
      label: "Transcript"
    }
  ]
};
```

## Implementations

[Decentralised Renderer Template](https://github.com/Open-Attestation/decentralized-renderer-react-template)

[Decentralised Renderer React Component](https://github.com/Open-Attestation/decentralized-renderer-react-components)
