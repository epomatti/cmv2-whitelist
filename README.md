# How to: Candy Machine v2 Whitelist

###### tags: `metaplex` `whitelist`

A comprehensive, step-by-step tutorial, for you to launch your whitelist using Candy Machine V2 with SPL Tokens + Gumdrop.

_Check out how to maintain Metaplex as a [private repository](https://hackmd.io/@epomatti/HyCOmamhY)_.

:::info
:information_source: You should always read the [official docs](https://docs.metaplex.com/) and keep up with the community updates.
:::

### Environment Setup

Install the required tools as described in the [official guide](https://docs.metaplex.com/candy-machine-v2/getting-started).


:::spoiler _Versions & Downloads_

You'll need all of these installed (don't need to be exact versions):

```bash
$ git version
git version 2.34.1

$ node --version
v16.13.1

$ yarn --version
1.22.17

$ ts-node --version
v10.4.0

$ solana --version
solana-cli 1.8.11 (src:423a4d65; feat:2385070269)
```

Download links:

- Git - https://git-scm.com/downloads
- Node.js - https://nodejs.org/en/download/
- Yarn - https://classic.yarnpkg.com/lang/en/docs/install
  _(Use `yarn v1.x` - later versions need extra config)_
- ts-node - https://www.npmjs.com/package/ts-node#Installation
- Solana Tool Suite - https://docs.solana.com/cli/install-solana-cli-tools

Only need Solana CLI? Run this bad boy:

```bash
sh -c "$(curl -sSfL https://release.solana.com/stable/install)"
```

I'm running on `WSL2` Ubuntu:

```
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.3 LTS
Release:        20.04
Codename:       focal
```
:::

### Environment Variables

Setup the variables we'll need for this example:

```bash
# Environment
export ENV='devnet'
export URL='https://api.devnet.solana.com'
export KEYPAIR="$HOME/.config/solana/devnet.json"
export RPC='https://api.devnet.solana.com'

# Candy Machine
export EXTRAS='./extras'
export CONFIG="$EXTRAS/config.json"
export WHITELIST="$EXTRAS/whitelist.json"

# Collection
export ASSETS="$EXTRAS/assets"
```

:::spoiler _Setup your free custom RPC provider (Optional for this tutorial)_

This tutorial uses an RPC provider on all commands. Create a free one on Figment:

1. Sign up to Figment here: https://datahub.figment.io/sign_up?service=solana
3. Find and copy your `API Key` and the base URL.
3. Set this URL as your RPC variable:
   ```bash
   export RPC='https://solana--devnet.datahub.figment.io/apikey/<YOUR API KEY>'
   ```
:::

### Create your Treasury Wallet

Create and fund your wallet:

```bash
solana config set --url $URL
solana-keygen new --outfile $KEYPAIR
solana airdrop 2 --keypair $KEYPAIR
```

:bulb: Copy the public key - you'll need it later.

For the sake of simplicity we'll set this keypair as default:

```bash
solana config set --keypair $KEYPAIR
```

You can always get the public key later:

```bash
solana-keygen pubkey
```

### Create the SPL Token

Whitelisted users will be allowed to mint using SPL Tokens which will serve as an exchange currency. Create the token:

```bash
spl-token create-token --decimals 0
```

:bulb: Copy the token - you'll need it later.

Store the output value and set the token address variable like this:

```sh
export SPL_TOKEN="<YOUR NEW TOKEN JUST CREATED>"
```

Mint the tokens:

```bash
spl-token create-account $SPL_TOKEN
spl-token mint $SPL_TOKEN 1000

# Consider disabling the mint when you're done
spl-token authorize $SPL_TOKEN mint --disable
```

For `mainnet` you should [register the token](https://github.com/solana-labs/token-list).

### Candy Machine Configuration

Get the Metaplex code, then install the dependencies:

```bash
git clone 'git@github.com:metaplex-foundation/metaplex.git'

cd metaplex
yarn install --cwd ./js/
```

Create the `extras` directory and the `config.json`:

```bash
mkdir extras
touch ./extras/config.json
```

Copy/paste the following JSON content into `config.json`:



```json
{
  "price": 0.5,
  "number": 10,
  "gatekeeper": null,
  "solTreasuryAccount": "<YOUR WALLET ADDRESS>",
  "splTokenAccount": null,
  "splToken": null,
  "goLiveDate": "30 Jan 2022 00:00:00 UTC",
  "endSettings": null,
  "whitelistMintSettings": {
    "mode": {
      "burnEveryTime": true
    },
    "mint": "<SPL TOKEN>",
    "presale": true,
    "discountPrice": 0.33
  },
  "hiddenSettings": null,
  "storage": "arweave",
  "ipfsInfuraProjectId": null,
  "ipfsInfuraSecret": null,
  "awsS3Bucket": null,
  "noRetainAuthority": false,
  "noMutable": false
}
```

:arrow_forward: Update `config.json` with your custom configuration:

- **`solTreasuryAccount`** - The public key of the keypair of the Treasury Account.
- **`goLiveDate`** - Set a date in the future to allow for the whitelist behavior.
- **`whitelistMintSettings.mint`** - This is the SPL Token created earlier.

Notice that the `discountPrice` is 0.33. It will override the `price` 0.5 only for the whitelist.

Check out the [official docs](https://docs.metaplex.com/candy-machine-v2/configuration) to learn more about these configurations.

### Prepare the Assets

Download the sample assets:

```bash
wget -O 'assets.tar.gz' https://github.com/epomatti/cmv2-whitelist/blob/main/assets.tar.gz?raw=true
     
tar -xzf assets.tar.gz --directory $EXTRAS
```

:arrow_forward: Update all JSON files under `./extras/assets`

For this example, change **`properties.creators.address`** value to your Treasury Account public key.


### Create the Candy Machine :candy:

This command will create the candy machine and upload all assets.

```bash
ts-node ./js/packages/cli/src/candy-machine-v2-cli.ts upload \
  --env $ENV \
  --keypair $KEYPAIR \
  --config-path $CONFIG \
  --rpc-url $RPC \
  $ASSETS
```

:bulb: Wait for all assets to be uploaded, then copy the candy machine ID - you'll need it later

For more info on the metadata check the [official docs](https://docs.metaplex.com/nft-standard).

:::info
:information_source: The `number` of NFTs cannot be changed later, so make sure to get it right on `mainnet`
:::

Verify the uploads:

```bash
ts-node ./js/packages/cli/src/candy-machine-v2-cli.ts verify_upload \
  --env $ENV \
  --keypair $KEYPAIR \
  --rpc-url $RPC
```
You need an output like this before going forward:

```
Key size 10
uploaded (10) out of (10)
ready to deploy!
```

:::spoiler _Click here if you need to Upload the candy machine_
```bash
ts-node ./js/packages/cli/src/candy-machine-v2-cli.ts update_candy_machine \
  --env $ENV \
  --keypair $KEYPAIR \
  --config-path $CONFIG \
  --rpc-url $RPC
```
::: 

### Create the Gumdrop

On this tutorial we're using `gumdrop` but you may use other means for token distribution.

Create the `whitelist.json` file which will hold the list of whitelisted users:

```bash
touch ./extras/whitelist.json
```

Copy and paste the following content into the `whitelist.json`:

```json
[
  {
    "handle": "<USER WALLET>",
    "amount": 1
  }
]
```

:arrow_forward: Now change the `handle` property value to the user wallet.

The `handle` here is the public wallet address of the user that will be whitelisted. You can create a new one in your Phantom wallet connected to the `devnet` network.

:arrow_forward: Add funds to the new user wallets: https://solfaucet.com/

:::spoiler _Demo: Create a Phantom user wallet on `devnet`_
![](https://i.imgur.com/aauLh6q.gif)
:::

<br/>

:::info
:information_source: Read the [official docs](https://docs.metaplex.com/airdrops/create-gumdrop) to learn more about other distribution methods (email, SMS, ...)
:::

Here we'll use the [Token Airdrop](https://docs.metaplex.com/airdrops/create-gumdrop#token-airdrop) drop type supported by `CMv2`.

Create the `gumdrop`:

```bash
ts-node ./js/packages/cli/src/gumdrop-cli.ts create \
  --env $ENV \
  --keypair $KEYPAIR \
  --rpc-url $RPC \
  --claim-integration "transfer" \
  --transfer-mint $SPL_TOKEN \
  --distribution-method "wallets" \
  --otp-auth "disable" \
  --distribution-list $WHITELIST
```

:bulb: Copy the `gumdrop` ID - you'll need it later.

:::info
:information_source: Real launches should consider a custom `--host` deployment. Check the gumdrop docs.
:::

### URL Claim the SPL Token

Under the **`./logs`** folder the URLs have been generated in the `devnet-urls.json` file.

![](https://i.imgur.com/yJnIetz.png)

:arrow_forward: Open the URL and claim your token

Users will claim their own tokens using this URL and use their tokens as proof for whitelist privileges while minting their NFTs.

:::spoiler _Demo: Claiming the token with a user wallet_
![Uploading file..._edf0uudoq]()

:::


### Setup the Minting Frontend

:::success

:rocket:  Updated to use the newly released [Candy Machine UI](https://docs.metaplex.com/candy-machine-v2/mint-frontend) module.
:::

The frontend is available at `./js/packages/candy-machine-ui`.

```bash
export CANDYUI='./js/packages/candy-machine-ui'

cp "$CANDYUI/.env.example" "$CANDYUI/.env"
```


:arrow_forward: Edit the `.env` file with your own configuration:

```
REACT_APP_CANDY_MACHINE_ID=<YOUR CANDY MACHINE PROGRAM ID>
REACT_APP_SOLANA_NETWORK=devnet
REACT_APP_SOLANA_RPC_HOST=https://api.devnet.solana.com
```

If you're using a custom RPC server, change that too.

Now install & start your minting frontend:

```bash
yarn --cwd $CANDYUI install
yarn --cwd $CANDYUI start
```

:::spoiler _Click here if you lost your Candy Machine ID_

Run this to see your Candy Machine configuration:

```bash
ts-node ./js/packages/cli/src/candy-machine-v2-cli.ts show \
  --env $ENV \
  --keypair $KEYPAIR \
  --rpc-url $RPC
 ```

Copy the ID:

![](https://i.imgur.com/s5KW4UC.png)
::: 


### Mint an NFT providing your SPL Token

Whitelisted users will then connect to the minting site and mint the NFT.

They will exchange the token as part of the whitelist + the SOL price configured in the `config.json` discount property.

:arrow_forward: Site should be running here: **http://localhost:3000**

![](https://i.imgur.com/ugMU789.gif)

As expected, the whitelist price is 0.33 SOL and the SPL Token is removed from the user wallet.

:::info
:information_source: Opened [this issue](https://github.com/metaplex-foundation/metaplex/issues/1465) to fix the displayed price, as it should be aware of the discount.
:::


And we reached the end of our tutorial.

### Close the Gumdrop

Finally, once the whitelist launch is over, you should close the `gumdrop`:

```sh
ts-node ./js/packages/cli/src/gumdrop-cli.ts close \
  --env $ENV \
  --keypair $KEYPAIR \
  --rpc-url $RPC \
  --claim-integration "transfer" \
  --transfer-mint $SPL_TOKEN \
  --base '.log/devnet-XXXXXXXXXXXXX.json'
```

Check the [official docs](https://docs.metaplex.com/airdrops/create-gumdrop#closing-a-gumdrop) for this closing behavior.

---

I hope you find this article useful.

Let me know of any fixes in the comments, or reach out on Twitter: [@EvandroPomatti](https://twitter.com/EvandroPomatti)