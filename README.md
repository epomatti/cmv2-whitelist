# Metaplex Candy Machine Whitelist

###### tags: `metaplex` `candy-machine` `gumdrop` `spl-token` `whitelist`

Launch your whitelist using Candy Machine V2.


## Environment Setup

Install the required tools as described in the [official guide](https://docs.metaplex.com/candy-machine-v2/getting-started).

Or, follow my guide below :point_down:

```bash
$ git version
git version 2.34.1

$ node --version
v16.13.1

$ yarn --version
1.22.17

$ ts-node --version
v10.4.0

$ solana-cli --version
solana-cli 1.8.11 (src:423a4d65; feat:2385070269)

$ spl-token --version
spl-token-cli 2.0.15
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


## Install the dependencies

### Environment Variables

```bash
export ENV='devnet'
export URL='https://api.devnet.solana.com'
export KEYPAIR="$HOME/.config/solana/devnet.json"
export CMV2='./js/packages/cli/src/candy-machine-v2-cli.ts'
export CONFIG='/extras/launch/config/devnet-config.json'
export ASSETS='/extras/metadata-config-utility/output/assets'
export NUMBER=10
```

:::info
:information_source: I'm skipping 
:::





## 2 - Candy Machine

## Create the SPL Token

