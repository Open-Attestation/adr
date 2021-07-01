OA doesn't fit in a QR code so don't even go there

BUT: OA can fit in a self-contained file with the only external data necessary being the DNS record, which can be done away with if you write a OA verifier site that has a list of trusted DIDs or use some form of hierarchical PKI like x509-style

```
{
  "recipient": {
    "name": "John Doe"
  },
  "$template": {
    "name": "main",
    "type": "SELF_CONTAINED",
     "visualRepresentation": `<svg>...</svg>` // fully self contained render of the credential
  },
  "issuers": [
    {
      "id": "did:ethr:0xaCc51f664D647C9928196c4e33D46fd98FDaA91D",
      "name": "Demo Issuer",
      "revocation": {
        "type": "NONE"
      },
      "identityProof": {
        "type": "DNS-DID",
        "location": "intermediate-sapphire-catfish.sandbox.openattestation.com",
        "key": "did:ethr:0xaCc51f664D647C9928196c4e33D46fd98FDaA91D#controller"
      }
    }
  ]
}
```

Now, the renderer on the custom OA verifier just needs to render the SVG.
Alternatively HTML with embedded CSS can be used, but it is probably easier to deal with rendering an SVG.

