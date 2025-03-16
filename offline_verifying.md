# Offline Verifier

## Problem

Currently, after a document has been issued in the Document Creator, the user has to download the file before clicking the Verify Documents tab and uploading the document for verification which may be cumbersome. Created documents can be easily accessed through their `links.self.href` attribute only because the Document creator uses this function to upload the document to a server:

```js
export const uploadToStorage = async (
  doc: WrappedDocument,
  documentStorage: DocumentStorage
): Promise<AxiosResponse> => {
  const qrCodeObj = decodeQrCode(doc.rawDocument.links.self.href);
  const uri = qrCodeObj.payload.uri;

  return axios({
    method: "post",
    url: uri,
    headers: getHeaders(documentStorage),
    data: {
      document: doc.wrappedDocument,
    },
  });
};
```
However, as the user may not have consented to an upload, this poses a problem in our current workflow. The current implementation of easy verification of a document can be found in the [gallery](https://gallery.openattestation.com/).

## Goal

To explore the option of opening a document without uploading it to a server. This will save the step of downloading, opening another window and uploading the document. Possible usecases include: Allowing a user to view a .tt document with .tt files as attachments easily in another tab. 

## Proposed Solution

We can use the postMessage method to conduct parent and child communication. To detect the message sent, an eventListener can be added in the DropZone. For example:

```js
  window.addEventListener(
    "message",
    (event) => {
      if (event.data.document) {
        updateCertificate(event.data.document);
      }
    },
    false
  );

```

In order for the parent to communicate with its child window, a reference can be saved to the return value of `window.open()`. We will
also pass in the index of the file as the name of the child window to ensure that the correct inner .tt document is sent to the child window. For example:

```js
  const childWin = window.open(url, index);
```

Once the reference is made, we will establish a 2-way handshake to ensure that messages can be sent from parent to child and child to parent. This is to 
ensure that the parent will send over the document only when the child window is ready to receive it.

When the child window is ready, it will send a READY message to the parent through:

```js
window.opener.postMessage({ type: "READY" });
```

When the parent receives the READY message, he will send out a LOAD_DOCUMENT message to its child which references the inner .tt document in 
the attachments attribute of the parent .tt file:

```js
if (event.data.type == "LOAD_DOCUMENT") {
  childWin.postMessage({ type: "LOAD_DOCUMENT",payload: MY_JSON_FILE.data.attachments[childWin.name].data });
}
```

When the child receives the LOAD_DOCUMENT message, he will retrieve the document and process it to before uploading it to the dropzone for it to be verified:

```js
if (event.data.type == "LOAD_DOCUMENT") {
  const doc = window.atob(event.data.payload.split(":")[2]);
  updateCertificate(JSON.parse(doc));
}
```

## Alternative Solution

Other methods we considered were:

1. LocalStorage method

We could store the document data on the user's local storage and upon clicking the Verify Document tab, load the data into the verifier before clearing it. However, this method is not recommended as there is no data protection, creating a security risk. There is also a ~5MB data limit which may be too low to store document data. Objects cannot be stored as well and must be stringified.

2. Broadcast Channel API

We could create a channel to allow a child window to subscribe to, enabling it to receive messages from its parent. More information about the implementation can be found [here](https://dev.to/dcodeyt/send-data-between-tabs-with-javascript-2oa). This method is better than listening to changes on the local storage as it is more secure and full objects can be posted. However, as of November 2020, this method is not supported by Safari, IE and Edge.