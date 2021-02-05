# Use of DS Proxy to bundle multiple transactions

## The Problem

As dapp developers and users, we often find ourselves in the need to execute multiple transactions in sequence or in parallel. With existing dapps, we can only execute one at a time. In TradeTrust example, when a EBL is being transferred, I'll need to do two things:

1. Create a title escrow with the new `owner` and `holder` and save the address (by calling the title escrow creator contract)
1. Transfer of owner from the existing one to the new title escrow contract address (by calling the ERC721 contract)

The current method is to add new functions to the title escrow to combine the actions:

```sol
function transferToNewEscrow(address newBeneficiary, address newHolder)
public
isHoldingToken
onlyHolder
allowTransferTitleEscrow(newBeneficiary, newHolder)
{
address newTitleEscrowAddress = titleEscrowFactory.deployNewTitleEscrow(
    address(tokenRegistry),
    newBeneficiary,
    newHolder
);
_transferTo(newTitleEscrowAddress);
}
```

The problem with this approach is that when new situation arises, such as the need to be able to "return" a surrendered document. The minter to the ERC721 contract will have to create the title escrow and then execute `sendToken` on the contract.

## The Solution

[DS-Proxy](https://dapp.tools/dappsys/ds-proxy.html) presents itself as one of the method to solve this for good and a previous experiment has been done to show how we can use ds-proxy (https://github.com/yehjxraymond/dsproxy-experiment/blob/master/src/index.ts).

The problem is that we will need to change the way we interact with smart contracts... and at the same time supporting people who are not using dsproxy...

### The Changes

#### Functions to call smart contracts

Instead of directly calling smart contracts like document store or token registry, all calls will be to the ds-proxy now.

Currently contract interactions are done through one of two methods:

Using contract function call directly

eg

```ts
const escrowDeploymentReceipt = await titleEscrowCreatorContract.deployNewTitleEscrow(
  contractInstance.address,
  previousBeneficiary,
  previousHolder
);

const escrowDeploymentTx = await escrowDeploymentReceipt.wait();
const deployedTitleEscrowArgs = escrowDeploymentTx.events?.find(
  (event) => event.event === "TitleEscrowDeployed"
)?.args;
```

where `titleEscrowCreatorContract` is an instance of `ethers.Contract`.

and

Using `contract-function-hook`

eg

```ts
const {
  send: changeHolder,
  state: changeHolderState,
  reset: resetChangeHolder,
} = useContractFunctionHook(titleEscrow, "changeHolder");
```

One likely way is to create `someAsyncMethod` for the direct calls, and then pass in the dsproxy commands into a wrapper hook to return the statuses:

```ts
const {
  send: restoreTokenSimple,
  state: restoreTokenSimpleState,
  reset: resetRestoreTokenSimple,
} = usePossibleProxyContractHook({
  direct: someAsyncMethod(),
  proxy: { method: "mintAndTransfer", args: ["xxx", "xxx", "xxx"] },
});
```

#### Tooling changes

Aside from website's changes, toolings around smart contract interactions, especially the CLI will have to add in the options to specify if a proxy is used, and if so, the address of that proxy...

### Another alternative

Another alternative to ds-proxy and writing permutations of code in existing smart contract will be to use a custom ds-proxy.

This method will add a fallback method on the ds-proxy to forward (not delegate) all calls (authorized by ds-auth) to a named contract. So the existence of the ds-proxy will be invisible to the tools and website code in most cases.

Only in cases where there is a possibility of combining transactions, it will check if the contract is a modified ds-proxy instance and then executes the combined transaction, otherwise it will fall back to sequential execution of transactions.

The following code need to be replaced/added in ds-proxy:

```sol
function() public payable {
  // load call data
  // check for auth
  // call the target contract with call data
}

function setTargetContract(address _target){
    target = _target;
}
```
