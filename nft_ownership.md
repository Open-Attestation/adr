###### Authors:

Raymond Yeh

# Single Owner to NFT for Transferable Records

Using a single owner for NFT allows additional business logic to be implemented in the smart contract level without deviating from the ERC721 standards.

Pros:
- Accessibility to tools for ERC721
- Availability of reference implementation (OpenZepplin)
- Better design in terms of abstraction
- Blockchain-blockchain interopability ([Hashed Time-lock Contract](https://medium.com/coinmonks/interoperability-the-holy-grail-of-blockchain-eb078e1a29cc)) 
- Atomic swaps (ie. USD <-> eBL)

Cons:
- Complexity in implementation. Ie. Different interaction between wallet and smart contract owners
- Complexity in user interaction. Ie. Can user still use the platform to perform all types of interaction?

# Problem/Solutions

## How do people tell the owner of the token?

Since smart contract addresses and wallet addresses are indistinguishable from another, users cannot tell if the "owner" will be able to directly interact with the token. For instance, if the token is owned by a smart contract the owner of a private key that controls the smart contract holding the token will not be able to:

- sign a message with the address
- transact with the standard ERC721 contract directly (ie. transferFrom)

In this case, the platform will have to distinguish if the owner is a smart contract or a wallet (via presence of code) and display different UI for each case. 

In the case of a wallet, the UI can have the functionalities:
- Transfer token
- Request for ownership proof
- Sign ownership proof
- Verify ownership proof

In the case of a smart contract, the UI can have the functionalities:
- Show that a smart contract controls this asset
- Dapp address (to bring user out to a separate platform to interact. Ie. https://wallet.gnosis.pm)

Other considerations:
[ERC-1654 Dapp-wallet authentication process with contract wallets support](https://github.com/ethereum/EIPs/issues/1654)