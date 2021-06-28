# Provider Solution

## The Problem

As there are many repositories that require to be connected to the Ethereum node, we have multiple ways of connecting, and thus, lead to un manageable and inconsistent code through out all the repositories.

## The Solution

To create a universal function to be used in all the repositories, to ensure consistency throughout all the repositories.

#### The function behaviour

The function should have the following behaviour:

1. Able to use:
   - etherscan provider
   - Jsonrpc provider
   - infura provider
   - alchemy provider
2. Able to connect to the specific network that the environment requires.
3. Should use these default values if no values are specified:
   - network: _mainnet_
   - provider: _infura_
   - apiKey: _infura apiKey_
4. Should use process.env value (if there are any)
5. Should allow user to provide the specific network, provider and apiKey values. (these values should override the process.env values)

```
// to use the default values in the function
generateProvider()

// to use the function with the process.env value
// in .env file
NETWORK = "ropsten"
// in the file where the provider is used
generateProvider()

// to use the function with a set of specific value
generateProvider({ network: "ropsten, provider: "infura", apiKey: "fjdkwcjwk123" })
```
