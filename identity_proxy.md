# Identity Proxy

## Goal

There are often times that the user of OA products are required to make multiple transactions on the blockchain in one business transaction. This leads to a wastage of gas as well as time when making multiple transactions. How might we allow transactions to be combined into one without writing all the permutations of business transactions in the core contract and with minimal changes to the front end code to enable this?

## Study into ds-proxy

[dapp.tools](https://dapp.tools/) has an implementation of an identity proxy. Using the proxy, one is able to send transactions to the identity proxy to execute code in the context of the identity proxy. This means that all transaction to external contracts will seemed like they originated from the identity proxy and state changes are at the proxy's state.

A demo implementation of how we might call the ds-proxy is documented [here](https://github.com/yehjxraymond/dsproxy-experiment/blob/master/src/index.ts).

ds-proxy automatically uses ds-auth to determine who is allowed to execute transactions on the contract. The default behavior is to only allow the contract deployer (or more accurately builder) to execute the contract. One can make use of the ds-auth contract to set custom access controls to the proxy contract. See [Auth.sol](https://github.com/yehjxraymond/dsproxy-experiment/blob/master/contracts/Auth.sol).

From the experiment we can see that if we implement ds-proxy, we will be able to distribute commonly used business transaction sequences, such as creation of a title escrow and then executing the transfer of a token to the escrow, for users. Users themselves can also implement their own custom business transaction contracts as well as access controls easily.

One problem remains. That is there will be a need for a major change on the frontend of OpenAttestation websites, and contract calls from the different tools such as CLI. How might we reduce the changes needed for these components while having the benefits of the ds-proxy?

## Forwarded Calls

In ds-proxy's implementation, it is doing nothing in the fallback transaction. We can leverage on the fallback transaction of the ds-proxy to forward (not delegate) calls to a target contract. It will call the target contract with the same call data from the proxy contract address itself. This also means that the ds-proxy will need to store a state `target`to point to a contract whenever a call to the proxy contract is made (and is not one of the defined functions in ds-proxy).

By doing this, we can set the target of the ds-proxy to the document store or token registry and transfer the ownership of the token registry or document store to the proxy. From the web interface or other tools, we can then simply replace the original document store or token registry address with that of the ds-proxy and the tools will continue to work.

For calls that require multiple transactions it will check if the address is an implementation of ds-proxy and if so, executes the `execute` transaction instead of having multiple calls to the document store or token registry.

Also, it should implement ds-auth to allow for custom access controls on the functions.

## Implementation

We will create a contract that inherits from ds-auth & ds-note:

Pseudocode:

```sol
address target;

function() external auth note payable {
    // get call data of current transaction
    // call target contract
    // (bool success, bytes memory data) = target.call("f")
    // revert if unsuccessful
}

function setTarget(address _target) external auth note {
    // sets target contract
}
```

From here, we can create a contract `ForwardingDsProxy` that inherits both ds-proxy and above contract. Be sure to have the above contract overwrites ds-proxy for the fallback function. 

## See also

https://github.com/Amxx/KitsuneWallet-ERC1836