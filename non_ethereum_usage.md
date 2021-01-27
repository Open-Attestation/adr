# Using OpenAttestation with non-Ethereum ledgers

## Quorum

[Quorum](https://consensys.net/quorum/) is a fork of Ethereum, with enterprise functionality and permissioning added on top of it. It also differs from Ethereum by using non-proof of work consensus algorithms - between a choice of [IBFT](https://docs.goquorum.consensys.net/en/stable/Concepts/Consensus/IBFT/) or [Raft](https://docs.goquorum.consensys.net/en/stable/Concepts/Consensus/Raft/). It thus follows that it is not a public, permissionless network like Ethereum, but a permissioned network.

Nevertheless, Quorum has taken care to maintain compatibility with Ethereum smart contracts and this means that we can use not only the OpenAttestation file format, we can also use the Document Store and Token Registry smart contracts without any modification.

In our brief experiments, we set up a private Quorum network using [Chainstack](https://chainstack.com/) as a platform, and found no problems deploying and issuing using the Document Store smart contract.

This was facilitated using the [open-attestation-cli](https://github.com/Open-Attestation/open-attestation-cli) with a [simplistic proxy](https://gist.github.com/rjchow/5b95f9ce9ad15c9e1f71640dafe72c83) for handling authentication. Simply run the proxy locally on port 8545, and then use open-attestation-cli with the flag --local.

For verifying of the issued documents, you can simply use [oa-verify](https://github.com/Open-Attestation/oa-verify) with a custom provider that points to the Quorum node if it is accessible publicly or using the same local proxy method as above. Augmenting oa-verify with your custom provider is documented [here](https://github.com/Open-Attestation/oa-verify#switching-network).