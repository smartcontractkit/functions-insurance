# Chainlink Functions <> Parametric Insurance Sample app

This use case showcases how Chainlink Functions can be used to trigger a parametric insurance to offer payouts to clients, with Chainlink Functions being used to fetch data from 3 different weather APIs for tempreture data.

[Parametric Insurance](https://en.wikipedia.org/wiki/Parametric_insurance) offers payouts to clients based upon a trigger event. In the sample, a smart contract will offers payouts based on the temperature in New York City. If the temperature falls below 60 degrees Fahrenheit (you can define the different number) for three consecutive days, the insurance will pay the client with balances. 

There is a smart contract called `ParametricInsurance` created for the use case, and clients can get payout if the predefined conditions in the samrt contract are met. 

In the `ParametricInsurance`, anyone can call function `executeRequest`(there is a limit that only one call per day) and send a request to Chainlink Functions. After the request event is detected by off-chain Chainlink node, the node will fetch the data and execute the computing logics defined in `Parametric-insurance-example.js`. 

After results are calculated, the returned data will be passed through [Chainlink Off-Chain Reporting mechanism(Chainlink OCR)](https://docs.chain.link/architecture-overview/off-chain-reporting/) to be aggregated. After the data aggregation, Chainlink functions will call function `fulfillRequest` and the client would be paid if predefined condition(three consecutive cold days in the use case) is met. 

## requirements
- node.js version 18

## Instructions to run this sample
1. Clone this repository to your local machine
2. Open this directory in your command line, then run `npm install` to install all dependencies.
3. Set environment variables in `.env` file
Prepend following environment variables in `.env` in the root directory of the repo.
```
OPEN_WEATHER_API_KEY="Open weather API key (Get a free one: https://openweathermap.org/)"
WORLD_WEATHER_ONLINE_API_KEY="World Weather API key (Get a free one: https://www.worldweatheronline.com/weather-api/)"
AMBEE_DATA_API_KEY="ambee data API key (Get a free one: https://api-dashboard.getambee.com/)"
CLIENT_ADDR="CLIENT ADDR"
```
Then get API keys for 3 different data source above
- OpenWeather API key: get a free key from [here](https://openweathermap.org/) (60 free calls per minute).
- WorldWeatherOnline API key: get a free key from [here](https://www.worldweatheronline.com/weather-api/)(500 calls per day).
- Ambeedata API key: get a free key from [here](https://api-dashboard.getambee.com/)(50,000 free calls).

`CLIENT_ADDR` is the address you want the parametric insurance to offer payouts.

4. Deploy and verify the RecordLabel contract to an actual blockchain network by running:
`npx hardhat functions-deploy-client --network network_name_here --verify true`

    Note: Make sure _ETHERSCAN_API_KEY_ or _POLYGONSCAN_API_KEY_ are set if using `--verify true`, depending on which network is used.

5. Create, fund & authorize a new Functions billing subscription by running:
`npx hardhat functions-sub-create --network network_name_here --amount LINK_funding_amount_here --contract 0xDeployed_client_contract_address_here`

    Note: Ensure your wallet has a sufficient LINK balance before running this command. Testnet LINK can be obtained at [faucets.chain.link](https://faucets.chain.link/).

6. Transfer some testnet native tokens to the contract with your wallet(metamask maybe).

7. Make an on-chain request by running:

    `npx hardhat functions-request --network network_name_here --contract 0xDeployed_client_contract_address_here --subid subscription_id_number_here`, replacing subscription_id_number_here with the subscription ID you received from the previous step

8. Resend on-chain requests(at least twice) to make sure the value of variable `consecutiveColdDays` reach 3 by running the same command in last step:

    `npx hardhat functions-request --network network_name_here --contract 0xDeployed_client_contract_address_here --subid subscription_id_number_here`, replacing subscription_id_number_here with the subscription ID you received from the previous step

9. Check if the client(client address is defined in `CLIENT_ADDR` in .env) receives the balance of insurance contract. 

## Tips
1. The default gaslimit for callback function is 100,000 and it may be insufficient. use `--gaslimit 300000` when send request like command below:
```
npx hardhat functions-request --network {network name} --contract {your contract addr} --subid {your subid} --gaslimit 300000
```
2. Want to check if the client paid by the parametric contract?

     Please transfer some native token on your network to the insurance contract before the number of consecutive days reach 3, and you can check if the token transferred to client address when the number of consucutive days reach 3.

3. Set a different temepreture consiered as cold and threshhold of cold days
    
    Update the state variable `coldTemp` and `consecutiveColdDays` in smart contract `ParametricInsurance.sol`. You can make the smart contract offer payouts if, for example, the temperature falls below 10 degrees Fahrenheit for 10 consecutive days.

# Command Glossary

Each of these commands can be executed in the following format.

`npx hardhat command_here --parameter1 parameter_1_here --parameter2 parameter_2_here`

Example: `npx hardhat functions-read --network mumbai --contract 0x787Fe00416140b37B026f3605c6C72d096110Bb8`

### Functions Commands

| Command                            | Description                                                                                                      | Parameters                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `compile`                          | Compiles all smart contracts                                                                                     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `functions-simulate`               | Simulates an end-to-end fulfillment locally for the FunctionsConsumer contract                                   | `gaslimit` (optional): Maximum amount of gas that can be used to call fulfillRequest in the client contract (defaults to 100,000 & must be less than 300,000)                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `functions-deploy-client`          | Deploys the FunctionsConsumer contract                                                                           | `network`: Name of blockchain network, `verify` (optional): Set to `true` to verify the FunctionsConsumer contract (defaults to `false`)                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `functions-request`                | Initiates a request from an FunctionsConsumer client contract from `Functions-request-config.js`                 | `network`: Name of blockchain network, `contract`: Address of the client contract to call, `subid`: Billing subscription ID used to pay for the request, `gaslimit` (optional): Maximum amount of gas that can be used to call fulfillRequest in the client contract (defaults to 100,000 & must be less than 300,000), `requestgas` (optional): Gas limit for calling the executeRequest function (defaults to 1,500,000), `simulate` (optional): Flag indicating if simulation should be run before making an on-chain request (defaults to true)                                |
| `functions-read`                   | Reads the latest response returned to a Functions client contract                                                | `network`: Name of blockchain network, `contract`: Address of the client contract to read                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `functions-read-error`             | Reads the latest error returned to a Functions client contract                                                   | `network`: Name of blockchain network, `contract`: Address of the client contract to read                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `functions-deploy-auto-client`     | Deploys the AutomatedFunctionsConsumer contract and sets Funcnction request from `Functions-request-config.js`   | `network`: Name of blockchain network, `subid`: Billing subscription ID used to pay for Functions requests, `gaslimit` (optional): Maximum amount of gas that can be used to call fulfillRequest in the client contract (defaults to 250000), `interval` (optional): Update interval in seconds for Automation to call performUpkeep (defaults to 300), `verify` (optional): Set to `true` to verify the FunctionsConsumer contract (defaults to `false`), `simulate` (optional): Flag indicating if simulation should be run before making an on-chain request (defaults to true) |
| `functions-check-upkeep`           | Checks if checkUpkeep returns true for an Automation compatible contract                                         | `network`: Name of blockchain network, `contract`: Address of the contract to check, `data` (optional): Hex string representing bytes that are passed to the checkUpkeep function (defaults to empty bytes)                                                                                                                                                                                                                                                                                                                                                                        |
| `functions-perform-upkeep`         | Manually call performUpkeep in an Automation compatible contract                                                 | `network`: Name of blockchain network, `contract`: Address of the contract to check, `data` (optional): Hex string representing bytes that are passed to the performUpkeep function (defaults to empty bytes)                                                                                                                                                                                                                                                                                                                                                                      |
| `functions-set-auto-request`       | Updates the Functions request in deployed AutomatedFunctionsConsumer contract from `Functions-request-config.js` | `network`: Name of blockchain network, `contract`: Address of the contract to update, `subid`: Billing subscription ID used to pay for Functions requests, `interval` (optional): Update interval in seconds for Automation to call performUpkeep (defaults to 300), `gaslimit` (optional): Maximum amount of gas that can be used to call fulfillRequest in the client contract (defaults to 250000)                                                                                                                                                                              |
| `functions-set-oracle-addr`        | Updates the oracle address for a client contract using the FunctionsOracle address from `network-config.js`      | `network`: Name of blockchain network, `contract`: Address of the client contract to update                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `functions-build-request`          | Creates a JSON file with Functions request parameters                                                            | `network`: Name of blockchain network, `output` (optional): Output file name (defaults to Functions-request.json), `simulate` (optional): Flag indicating if simulation should be run before building the request JSON file (defaults to true)                                                                                                                                                                                                                                                                                                                                     |
| `functions-build-offchain-secrets` | Builds an off-chain secrets object for one or many nodes that can be uploaded and referenced via URL             | `network`: Name of blockchain network, `output` (optional): Output file name (defaults to offchain-secrets.json)                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |

### Functions Subscription Management Commands

| Command                      | Description                                                                                                                        | Parameters                                                                                                                                                                                                                                             |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `functions-sub-create`       | Creates a new billing subscription for Functions client contracts                                                                  | `network`: Name of blockchain network, `amount` (optional): Initial amount used to fund the subscription in LINK (decimals are accepted), `contract` (optional): Address of the client contract address authorized to use the new billing subscription |
| `functions-sub-info`         | Gets the Functions billing subscription balance, owner, and list of authorized client contract addresses                           | `network`: Name of blockchain network, `subid`: Subscription ID                                                                                                                                                                                        |
| `functions-sub-fund`         | Funds a billing subscription with LINK                                                                                             | `network`: Name of blockchain network, `subid`: Subscription ID, `amount`: Amount to fund subscription in LINK (decimals are accepted)                                                                                                                 |
| `functions-sub-cancel`       | Cancels Functions billing subscription and refunds unused balance. Cancellation is only possible if there are no pending requests. | `network`: Name of blockchain network, `subid`: Subscription ID, `refundaddress` (optional): Address where the remaining subscription balance is sent (defaults to caller's address)                                                                   |
| `functions-sub-add`          | Adds a client contract to the Functions billing subscription                                                                       | `network`: Name of blockchain network, `subid`: Subscription ID, `contract`: Address of the client contract to authorize for billing                                                                                                                   |
| `functions-sub-remove`       | Removes a client contract from an Functions billing subscription                                                                   | `network`: Name of blockchain network, `subid`: Subscription ID, `contract`: Address of the client contract to remove from billing subscription                                                                                                        |
| `functions-sub-transfer`     | Request ownership of an Functions subscription be transferred to a new address                                                     | `network`: Name of blockchain network, `subid`: Subscription ID, `newowner`: Address of the new owner                                                                                                                                                  |
| `functions-sub-accept`       | Accepts ownership of an Functions subscription after a transfer is requested                                                       | `network`: Name of blockchain network, `subid`: Subscription ID                                                                                                                                                                                        |
| `functions-timeout-requests` | Times out expired requests                                                                                                         | `network`: Name of blockchain network, `requestids`: 1 or more request IDs to timeout separated by commas                                                                                                                                              |

# Request Configuration

Chainlink Functions requests can be configured by modifying values in the `requestConfig` object found in the `Functions-request-config.js` file located in the root of this repository.

| Setting Name             | Description                                                                                                                                                                                                                                                                                                                 |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `codeLocation`           | This specifies where the JavaScript code for a request is located. Currently, only the `Location.Inline` option is supported (represented by the value `0`). This means the JavaScript string is provided directly in the on-chain request instead of being referenced via a URL.                                           |
| `secretsLocation`        | This specifies where the encrypted secrets for a request are located. `Location.Inline` (represented by the value `0`) means encrypted secrets are provided directly in the on-chain, while `Location.Remote` (represented by `1`) means secrets are referenced via encrypted URLs.                                         |
| `codeLanguage`           | This specifies the language of the source code which is executed in a request. Currently, only `JavaScript` is supported (represented by the value `0`).                                                                                                                                                                    |
| `source`                 | This is a string containing the source code which is executed in a request. This must be valid JavaScript code that returns a Buffer. See the [JavaScript Code](#javascript-code) section for more details.                                                                                                                 |
| `secrets`                | This is a JavaScript object which contains secret values that are injected into the JavaScript source code and can be accessed using the name `secrets`. This object will be automatically encrypted by the tooling using the DON public key before making an on-chain request. This object can only contain string values. |
| `walletPrivateKey`       | This is the EVM private key. It is used to generate a signature for the encrypted secrets such that the secrets cannot be reused by an unauthorized 3rd party.                                                                                                                                                              |
| `args`                   | This is an array of strings which contains values that are injected into the JavaScript source code and can be accessed using the name `args`. This provides a convenient way to set modifiable parameters within a request.                                                                                                |
| `expectedReturnType`     | This specifies the expected return type of a request. It has no on-chain impact, but is used by the CLI to decode the response bytes into the specified type. The options are `uint256`, `int256`, `string`, or `Buffer`.                                                                                                   |
| `secretsURLs`            | This is an array of URLs where encrypted off-chain secrets can be fetched when a request is executed if `secretsLocation` == `Location.Remote`. This array is converted into a space-separated string, encrypted using the DON public key, and used as the `secrets` parameter.                                             |
| `perNodeOffchainSecrets` | This is an array of `secrets` objects that enables the optional ability to assign a separate set of secrets for each node in the DON if `secretsLocation` == `Location.Remote`. It is used by the `functions-build-offchain-secret` command. See the [Off-chain Secrets](#off-chain-secrets) section for more details.      |
| `globalOffchainSecrets`  | This is a default `secrets` object that any DON member can use to process a request. It is used by the `functions-build-offchain-secret` command. See the [Off-chain Secrets](#off-chain-secrets) section for more details.                                                                                                 |

## JavaScript Code

The JavaScript source code for an Functions request can use vanilla Node.js features, but cannot use any imported modules or `require` statements.
It must return a JavaScript Buffer which represents the response bytes that are sent back to the requesting contract.
Encoding functions are provided in the [Functions library](#functions-library).
Additionally, the script must return in **less than 10 seconds** or it will be terminated and send back an error to the requesting contract.

In order to make HTTP requests, the source code must use the `Functions.makeHttpRequest` function from the exposed [Functions library](#functions-library).
Asynchronous code with top-level `await` statements is supported, as shown in the file `API-request-example.js`.

### Functions Library

The `Functions` library is injected into the JavaScript source code and can be accessed using the name `Functions`.

In order to make HTTP requests, only the `Functions.makeHttpRequest(x)` function can be used. All other methods of accessing the Internet are restricted.
The function takes an object `x` with the following parameters.

```
{
  url: String with the URL to which the request is sent,
  method: (optional) String specifying the HTTP method to use which can be either 'GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD', or 'OPTIONS' (defaults to 'GET'),
  headers: (optional) Object with headers to use in the request,
  params: (optional) Object with URL query parameters,
  data: (optional) Object which represents the body sent with the request,
  timeout: (optional) Number with the maximum request duration in ms (defaults to 5000ms),
  responseType: (optional) String specifying the expected response type which can be either 'json', 'arraybuffer', 'document', 'text' or 'stream' (defaults to 'json'),
}
```

The function returns a promise that resolves to either a success response object or an error response object.

A success response object will have the following parameters.

```
{
  error: false,
  data: Response data sent by the server,
  status: Number representing the response status,
  statusText: String representing the response status,
  headers: Object with response headers sent by the server,
}
```

An error response object will have the following parameters.

```
{
  error: true,
  message: (may be undefined) String containing error message,
  code: (may be undefined) String containing an error code,
  response: (may be undefined) Object containing response from server,
}
```

This library also exposes functions for encoding JavaScript values into Buffers which represent the bytes that a returned on-chain.

- `Functions.encodeUint256(x)` takes a positive JavaScript integer number `x` and returns a 32 byte Buffer representing `x` as a `uint256` type in Solidity.
- `Functions.encodeInt256(x)` takes a JavaScript integer number `x` and returns a 32 byte Buffer representing `x` as a `int256` type in Solidity.
- `Functions.encodeString(x)` takes a JavaScript string `x` and returns a Buffer representing `x` as a `string` type in Solidity.

## Modifying Contracts

Client contracts which initiate a request and receive a fulfillment can be modified for specific use cases. The only requirements are that the client contract extends the `FunctionsClient` contract and the `fulfillRequest` callback function never uses more than 300,000 gas.

## Simulating Requests

An end-to-end request initiation and fulfillment can be simulated using the `npx hardhat functions-simulate` command. This command will report the total estimated cost of a request in LINK using the latest on-chain gas prices. Costs are based on the amount of gas used to validate the response and call the client contract's `fulfillRequest` function, plus a flat fee. Please note that actual request costs can vary based on gas prices when a request is initiated on-chain.

## Off-chain Secrets

Instead of using encrypted secrets stored directly on the blockchain, encrypted secrets can also be hosted off-chain and be fetched by DON nodes via HTTP when a request is initiated.

Off-chain secrets also enable a separate set of secrets to be assigned to each node in the DON. Each node will not be able to decrypt the set of secrets belonging to another node. Optionally, a set of default secrets encrypted with the DON public key can be used as a fallback by any DON member who does not have a set of secrets assigned to them. This handles the case where a new member is added to the DON, but the assigned secrets have not yet been updated.

To use per-node assigned secrets, enter a list of secrets objects into `perNodeOffchainSecrets` in `Functions-request-config.js` before running the `functions-build-offchain-secrets` command. The number of objects in the array must correspond to the number of nodes in the DON. Default secrets can be entered into the `globalOffchainSecrets` parameter of `Functions-request-config.js`. Each secrets object must have the same set of entries, but the values for each entry can be different (ie: `[ { apiKey: '123' }, { apiKey: '456' } ]`). If the per-node secrets feature is not desired, `perNodeOffchainSecrets` can be left empty and a single set of secrets can be entered for `globalOffchainSecrets`.

To generate the encrypted secrets JSON file, run the command `npx hardhat functions-build-offchain-secrets --network network_name_here`. This will output the file `offchain-secrets.json` which can be uploaded to S3, Github, or another hosting service that allows the JSON file to be fetched via URL.
Once the JSON file is uploaded, set `secretsLocation` to `Location.Remote` in `Functions-request-config.js` and enter the URL(s) where the JSON file is hosted into `secretsURLs`. Multiple URLs can be entered as a fallback in case any of the URLs are offline. Each URL should host the exact same JSON file. The tooling will automatically pack the secrets URL(s) into a space-separated string and encrypt the string using the DON public key so no 3rd party can view the URLs. Finally, this encrypted string of URLs is used in the `secrets` parameter when making an on-chain request.

URLs which host secrets must be available ever time a request is executed by DON nodes. For optimal security, it is recommended to expire the URLs when the off-chain secrets are no longer in use.

# Automation Integration

Chainlink Functions can be used with Chainlink Automation in order to automatically trigger a Functions request.

1. Create & fund a new Functions billing subscription by running:<br>`npx hardhat functions-sub-create --network network_name_here --amount LINK_funding_amount_here`<br>**Note**: Ensure your wallet has a sufficient LINK balance before running this command.<br><br>
2. Deploy the `AutomationFunctionsConsumer` client contract by running:<br>`npx hardhat functions-deploy-auto-client --network network_name_here --subid subscription_id_number_here --interval time_between_requests_here --verify true`<br>**Note**: Make sure `ETHERSCAN_API_KEY` or `POLYGONSCAN_API_KEY` environment variables are set. API keys for these services are freely available to anyone who creates an EtherScan or PolygonScan account.<br><br>
3. Register the contract for upkeep via the Chainlink Automation web app here: [https://automation.chain.link/](https://automation.chain.link/)
   - Find further documentation for working with Chainlink Automation here: [https://docs.chain.link/chainlink-automation/introduction](https://docs.chain.link/chainlink-automation/introduction)

Once the contract is registered for upkeep, check the latest response or error with the commands `npx hardhat functions-read --network network_name_here --contract contract_address_here`.

For debugging, use the command `npx hardhat functions-check-upkeep --network network_name_here --contract contract_address_here` to see if Automation needs to call `performUpkeep`.
To manually trigger a request, use the command `npx hardhat functions-perform-upkeep --network network_name_here --contract contract_address_here`.


# Sample Apps

> :warning: **Functions is in Beta**: Some of these use cases are solely to educate developers on functionality, current and proposed, in the rollout of Chainlink Functions. While several of these sample apps demonstrate posting data to external APIs, this functionality is under active development and is not yet recommended for production use. 

*Notes to Repo Developers*:
- Currently, we follow this [workflow for forks & syncing](https://www.atlassian.com/git/tutorials/git-forks-and-upstreams). These instructions use the same terminology from that article regarding "origin" and "upstream" repos. Please add the `upstream` repo as indicated in the article.
- Please make sure the `main` branch of this repo is synced to `main` of its [upstream repo](https://github.com/smartcontractkit/functions-hardhat-starter-kit) before pulling and adding any code.
- once you have synced to `main` of upstream, create  a new local branch on your machine with `git checkout -b <<YOUR DEV BRANCH NAME>>`
- As you develop locally, the `main` branch in the upstream may progress, so make sure you sync this fork via the Github UI and the `git pull` into your `main` branch. To get those changes patched into your dev branch, change to your dev branch and then run `git rebase main`.  This will apply all your changes "on top of" the latest syncs.
- Ideally do not push your dev branch to this repo until you're ready to submit a PR. Complete your development, and rebase your dev branch onto `main` before you push and request a PR   :warning: **If you have already pushed and have more changes your local branch and the remove will have a "has diverged" error.** You may need to follow the `git merge` workflow referred to in article mentioned above, to resolve conflicts.
- All sample app use case code goes into the `/samples`directory. Take a look at `/samples/twilio-spotify` to see how to write a sample app.
- Please add sample-specific instructions to a separate README inside your sample app directory.  For example `./samples/twilio-spotify/README.md`
- when submitting a PR for approval :warning: **make sure you set the base repo to this samples repo and NOT the upstream. Make sure you set the base branch as `main`**

### How to run a sample app

When running a sample app: 
- Take a look at the README file inside the sample app's directory in the `./samples/...` path.
- always make sure you comment out the existing `requestConfig` object in the `./Functions-request-config.js` file 
- Replace it with a correctly set up request configuration object that is specific to your app.  For example, for the twilio-spotify example, the config object for that use case is specified in `./samples/twilio-spotify/twilio-spotify-requestConfig.js`. That object is copied and pasted into `./Functions-request-config.js` to replace the default object.
- when running the CLI commands (which are Hardhat [tasks](https://hardhat.org/hardhat-runner/docs/guides/tasks-and-scripts)), be sure to find the script that implements the task in `/tasks` directory, and change the Contract name in the line that looks like this `const clientFactory = await ethers.getContractFactory("FunctionsConsumer")`. In the Twilio-spotify sample, the contract in this line will read as `const clientFactory = await ethers.getContractFactory("RecordLabel")`

# Parametric Insurance
Please check [here](https://github.com/smartcontractkit/functions-insurance/blob/main/samples/parametric-insurance/README.md) to see more details of Parameteric Inurance use case. 