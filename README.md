# EtherEver Dedicated MetaMask Mobile Wallet

[![Code Validation](https://github.com/makewalletfirst/EtherEver-Metamask8/actions/workflows/ci.yml/badge.svg)](https://github.com/makewalletfirst/EtherEver-Metamask8/actions/workflows/ci.yml)
[![Ecosystem](https://img.shields.io/badge/Ecosystem-EtherEver-blue)](https://etherever.ever-chain.xyz)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)

Welcome to the official repository for the **EtherEver MetaMask Mobile Wallet**.
This project is a tailored, pre-configured distribution of the open-source
[MetaMask Mobile App](https://github.com/MetaMask/metamask-mobile) designed
specifically for **EtherEver (ETE)**.

---

## 🌟 About EtherEver MetaMask

EtherEver (ETE) is an educational Layer-1 PoW blockchain hardforked from
Ethereum mainnet at block **#1,919,999**. Configuring RPC endpoints manually
can be a major friction point for students and developers.

This repository solves that by providing an **out-of-the-box mobile wallet
pre-configured to default directly to the EtherEver network**. By bundling our
nodes, symbols, and explorers directly into the core code, users are
immediately onboarded onto the ETE network with **zero manual setup**.

---

## 🛠️ Technology Stack

- **Framework**: React Native (TypeScript)
- **Package Manager**: Yarn v4 (Berry) with Corepack
- **Target Platforms**: Android (Gradle / Java 21) & iOS (CocoaPods / Xcode)
- **Linting & Type checking**: ESLint & TypeScript Compiler (`tsc`)
- **Branding**: Customized logo assets and preset themes for the ETE ecosystem

---

## 🔧 Core Network Customizations (Under the Hood)

To replace Ethereum Mainnet (`0x1`) with EtherEver (`0xe2c3` / `58051`) as the
default popular network, the following key code integrations are in place:

1. **Network Selection Hooks**
   - `app/components/hooks/useNetworksByNamespace/useNetworksByNamespace.ts`
     and `useNetworksToUse/useNetworksToUse.ts` prioritize the ETE network
   - Non-EVM namespaces (Bitcoin, Solana, Tron) are deactivated by default
     to provide a streamlined, focused EVM experience

2. **Hardcoded Chain-ID Swaps**
   - The default chain ID of `0x1` is swapped to `0xe2c3` in:
     - `app/components/UI/NetworkManager/index.tsx`
     - `app/components/Views/NetworkSelector/NetworkSelector.tsx`
     - `app/core/Engine/controllers/transaction-controller/data-helpers.ts`

3. **Preset Chain Details**
   | Field | Value |
   |---|---|
   | Network Name | EtherEver |
   | RPC URL | `https://rpc-ether.ever-chain.xyz` |
   | Chain ID | `58051` (`0xe2c3`) |
   | Currency Symbol | `ETE` (Ether) |
   | Block Explorer | `https://etherever.ever-chain.xyz` |

4. **Branding Assets**
   - Icons and names are re-mapped dynamically, promoting ETE to the popular
     networks tab with native icon rendering instead of custom-RPC fallbacks

### 🆕 4-Fold Safety Net (only EtherEver visible)

To prevent other EVM networks (Linea/Base/Arbitrum/BSC/Polygon/Optimism…) and
non-EVM chains (BTC/Solana/Tron) from leaking into the wallet through any path,
four independent guards are applied:

| Guard | File | What it does |
|---|---|---|
| ① Receive-address selector whitelist | `app/selectors/multichainAccounts/accounts.ts` | EVM list filtered to `EVER_ALLOWED_HEX = {'0xe2c3'}`. Non-EVM accounts forced to `scopes = []` so no row is rendered |
| ② NetworkController initial state cleanup | `app/core/Engine/controllers/network-controller-init.ts` | Every default chain except `0xe2c3` is `delete`-d from `networkConfigurationsByChainId` |
| ③ MultichainNetworkController empty state | `app/core/Engine/controllers/multichain-network-controller/multichain-network-controller-init.ts` | `multichainNetworkConfigurationsByChainId = {}` forced — no BTC/SOL/TRX auto-add |
| ④ PREINSTALLED_SNAPS surgical exclusion | `app/lib/snaps/preinstalled-snaps.ts` | Solana/Bitcoin/Tron snaps removed from the array (imports preserved so the 27 fence-protected init files don't break) |

### 🆕 Korean activity-tab label

`locales/languages/ko.json`:
```json
"view_full_history_on_etherscan": "EtherEver스캔에서 전체 기록 확인"
```
The URL was already hijacked to `etherever.ever-chain.xyz` upstream — only the
label string is swapped.

### 🆕 Incoming transaction fix (patches/)

v7.62 of `@metamask/transaction-controller` hardcodes
`new AccountsApiRemoteTransactionSource()` which talks only to
`https://accounts.api.cx.metamask.io`. That API does not know about EtherEver
chain `0xe2c3`, so incoming transfers never appeared in the Activity tab.

The fix lives at
`patches/@metamask+transaction-controller+62.4.0.patch` (applied by
`patch-package` post-install). It:
- Adds `0xe2c3` to `SUPPORTED_CHAIN_IDS`
- In `_queryTransactions`, when `chainIds` includes `0xe2c3`, fetches from
  `https://etherever.ever-chain.xyz/api?module=account&action=txlist`
  (Blockscout's etherscan-compatible endpoint) and converts each row into the
  shape `_normalizeTransaction` expects
- Returns early for EtherEver-only requests so the metamask API isn't called

Both `.mjs` and `.cjs` files are patched — RN uses `.cjs` (per
`package.json → main`) so missing that file is a silent no-op.

### 🆕 Metro transformer lint bypass

`metro.transform.js`:
```js
// await lintTransformedFile(getESLintInstance(), filename, processedSource);
```
The build-fence transformer's per-file ESLint call balloons across the
~13,296-module bundle, causing the Metro `jest-worker` SIGTERM after ~9
minutes. Commenting out this one line is enough.

---

## 📁 Repository Structure

```
├── app/                              # Main React Native components, UI layers, hooks, engines
│   ├── components/
│   │   ├── UI/                       # Reusable UI overlays and managers (e.g. NetworkManager)
│   │   └── Views/                    # Screen views (e.g. NetworkSelector)
│   ├── core/Engine/                  # MetaMask core controller engines
│   ├── selectors/                    # Redux selectors (receive-address whitelist)
│   └── lib/snaps/                    # PREINSTALLED_SNAPS array
├── locales/languages/                # Internationalization JSON assets
├── patches/                          # patch-package patches (incoming tx fix)
├── android/                          # Native Android project config and Gradle assets
├── ios/                              # Native iOS project config and CocoaPods/Xcode files
├── scripts/                          # Build automation, setup, and bot integrations
│   ├── build.sh                      # Core compiler script for target builds
│   └── setup.mjs                     # React Native project bootstrap script
└── .github/workflows/
    └── ci.yml                        # GitHub Actions type-check + workspace integrity
```

---

## 🚀 Getting Started (Build & Run)

Make sure your machine meets the React Native build prerequisites
(appropriate Android/iOS SDKs, Node, and Java).

### Prerequisites
- **Node.js**: `^20.18.0` (as defined in `package.json` engines)
- **Yarn**: `^4.12.0` (Corepack is required)
- **Java SDK**: Version 21 (matches the build environment we test against)
- **Android SDK** with NDK `26.1.10909125`
- **Xcode / CocoaPods**: (for iOS builds on macOS only)

---

### Step 1: Install Dependencies
Enable Corepack and install yarn dependencies. `patch-package` will
automatically apply `patches/@metamask+transaction-controller+62.4.0.patch`
during `postinstall`:
```bash
corepack enable
yarn install --no-immutable
```

---

### Step 2: Initialize Build Setup
Run the bootstrap setup script to generate local environment variables, clean
build directories, and prepare key assets:
```bash
node scripts/setup.mjs
```
*(For headless CI environments, run `node scripts/setup.mjs --build-on-github-ci` or `yarn setup:github-ci`.)*

---

### Step 3: Run / Build the App

#### Android (Generate APK)

```bash
yarn build:android:main:prod
```
Or run a localized debug server:
```bash
yarn start:android
```

Output path:
`android/app/build/outputs/apk/prod/release/app-prod-release.apk`

#### iOS (Generate IPA / run Simulator)

```bash
yarn pod:install
yarn build:ios:main:prod
# or:
yarn start:ios
```

---

## ⚙️ Operations notes (from server-side build experience)

For unattended production builds on a constrained host, the following
recipe has proven stable (~13 min on a 6-core / 12 GB Linux VM):

```bash
systemd-run --scope --unit=ethermask-build \
  -p MemoryMax=11G -p MemorySwapMax=8G -p CPUQuota=300% \
  bash -c '
    cd /root/metamask-mobile/android
    set -a; source ../.js.env 2>/dev/null; source ../.android.env 2>/dev/null; set +a
    export METAMASK_BUILD_TYPE=main      # Empty value in .android.env overrides .js.env "main"; force it
    export ANDROID_HOME=/root/android-sdk
    taskset -c 0,1,2 nice -n 19 ionice -c3 ./gradlew assembleProdRelease \
      --max-workers=1 -Dorg.gradle.parallel=false -Dorg.gradle.workers.max=1 --no-daemon
  '
```

Common pitfalls (each one cost us a failed build during initial bring-up):
- NDK CMake ninja wrapper must skip `-j 2` for `-t` subcommands
  (`-t restat`, `-t deps` etc reject `-j`)
- `.android.env`'s `METAMASK_BUILD_TYPE=` (empty) overrides `.js.env`'s
  `"main"` — force `export METAMASK_BUILD_TYPE=main` after sourcing both
- Metro bundle output dir (`createBundleProdReleaseJsAndAssets`) must be
  manually deleted when source files change, otherwise gradle marks it
  UP-TO-DATE and your changes don't reach the APK
- `git commit` may silently fail under husky if `yarn` isn't on PATH —
  use `HUSKY=0 git commit --no-verify` for CI-style flows

---

## ⚠️ Important Precautions & Coding Policies

- **Strict Code Preservation**: Do not alter any core source files under
  `app/` unless deliberately changing default network configs. Documentation
  edits are always safe
- **Type Checking Checkpoint**: Always verify that your changes pass standard
  TypeScript compilation:
  ```bash
  yarn lint:tsc
  ```
- **Signing Keys**: Production builds require correctly configured keystore
  properties (`android/gradle.properties` or iOS provisioning profiles).
  Ensure these files are populated but kept secure

---

## 📜 License

This project is derived from the MetaMask Mobile codebase. All modifications
are distributed under the original license terms (see [LICENSE](./LICENSE) and
[attribution.txt](./attribution.txt)).
