# Burning TradeTrustERC721 Token

## Status

Draft

## Goal

ERC721 Burnable Token can be irreversibly burned (destroyed). The ERC721 implemented by OpenZeppelin allows this by transfering burn tokens to address(0) as shown below:

```sol
function _burn(address owner, uint256 tokenId) internal {
    require(ownerOf(tokenId) == owner, "ERC721: burn of token that is not own");

    _clearApproval(tokenId);

    _ownedTokensCount[owner].decrement();
    _tokenOwner[tokenId] = address(0);

    emit Transfer(owner, address(0), tokenId);
}
```

However, TradeTrust is still required to render and track burnt token. Since unminted tokens and destoryed token have the same owner address in the Erc721 Token Registry, we needed to find a way to differentiate a unminted and burnt token. This means that TradeTrust would have to modify the ERC721 implementation or modify its verifier method.

## Options

After some discussion, 2 strategies stood out:

#### Continue using 0x0 as burn address

This strategy involve burning token normally. However, when verifying the token, the verifier would check if the document had previously been emitted a `Minted` event.

Pros:

- semantically correct because token is not owned by anyone anymore
- no need to inform consumers of a burn address

Cons:

- difficult to implement
- event emission might be lost in future
- state is not immediately available to be queried (as both unminted token and burnt tokens had `0x0` address, thus we need to query events)

#### using 0xknown as burn address (like [0x000000000000000000000000000000000000dead](https://etherscan.io/address/0x000000000000000000000000000000000000dead))

This method instead would see that the token would be burn to `0xdead` address, a known burn address which no one has the keys for. Subsequently, in order to track the token, we will need to check the `ownerOf` token from the token registry.

Pros:

- easy to implement
- known dead address are available and labeled
- state is immediately available to be queried

Cons:

- burnt address a convention that needs to be propagated
- it's not possible to prove to another party that this address is uncontrolled

## Implementation

In the end, a 0xknown burn address, `0x000000000000000000000000000000000000dead` was choosen. Since the logic of tracking token after burning is an app layer logic, there is no need to bloat the `TradeTrustERC721` smart contract with more methods. Also, scalability wise, if there were more intermediate state, other known burn address can be used to represent different states as opposed to more complex app layer logic on the verifier.

## Review on Current State

For token-registry, a specificed function called `destroyToken` will be used to denote this function with the following implementaion.

```sol
function destroyToken(uint256 _tokenId) public onlyMinter {
    require(ownerOf(_tokenId) == address(this), "Cannot destroy token: Token not owned by token registry");
    _safeTransferFrom(ownerOf(_tokenId), 0x000000000000000000000000000000000000dEaD, _tokenId, "");
    emit TokenBurnt(_tokenId);
}
```
