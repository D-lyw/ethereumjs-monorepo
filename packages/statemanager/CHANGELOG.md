# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
(modification: no type change headlines) and this project adheres to
[Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## 2.1.0 - 2023-10-23

### New Diff-Based Code Cache

This release introduces a new code cache implementation, see PR [#3022](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3022) and [#3080](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3080). The new cache complements the expanded account and storage caches and now also tracks stored/deployed-code-changes along commits and reverts and so only keeps code in the cache which made it to the final state change.

The new cache is substantially more robust towards various type of revert-based attacks and grows a more-used cache over time, since never-applied values are consecutively sorted out.

### Peformance Option to store Storage Keys with Prefix

This release introduces a new option `prefixStorageTrieKeys` which triggers the underlying trie to store storage key values with a prefix based on the account address, see PR [#3023](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3023). This significantly increases performance for consecutive storage accesses for the same account on especially larger tries, since trie node accesses get noticeably faster when performed by the underlying key-value store since values are stored close to each other.

While this option is deactivated by default it is recommended for most use cases for it to be activated. Note that this option is not backwards-compatible with existing databases and therefore can't be used if access to existing DBs needs to be guaranteed.

### Bugfixes

- Fix for `dumpStorage()` for `EthersStateManager`, PR [#3009](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3009)

### Other Changes

- Allow for users to decide if to either downlevel (so: adopt them for a short-lived scenario) caches or not on `shallowCopy()` by adding a new `downlevelCaches` parameter (default: `true`), PR [#3063](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3063)
- Return zero values for `getProof()` as `0x0`, PR [#3038](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3038)
- Deactivate storage/account caches for cache size 0, PR [#3012](https://github.com/ethereumjs/ethereumjs-monorepo/pull/3012)

## 2.0.0 - 2023-08-09

Final release version from the breaking release round from Summer 2023 on the EthereumJS libraries, thanks to the whole team for this amazing accomplishment! ❤️ 🥳

See [RC1 release notes](https://github.com/ethereumjs/ethereumjs-monorepo/releases/tag/%40ethereumjs%2Fstatemanager%402.0.0-rc.1) for the main change description.

Following additional changes since RC1:

- `Breaking`: new `dumpStorageRangeAt()` implementation + EVMStateManager interface addition (in @ethereumjs/common), PR [#2922](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2922)

## 2.0.0-rc.1 - 2023-07-18

This is the release candidate (RC1) for the upcoming breaking releases on the various EthereumJS libraries. The associated release notes below are the main source of information on the changeset, also for the upcoming final releases, where we'll just provide change addition summaries + references to these RC1 notes.

At time of the RC1 releases there is/was no plan for a second RC round and breaking releases following relatively shorty (2-3 weeks) after the RC1 round. Things may change though depending on the feedback we'll receive.

### Introduction

This round of breaking releases brings the EthereumJS libraries to the browser. Finally! 🤩

While you could use our libraries in the browser libraries before, there had been caveats.

WE HAVE ELIMINATED ALL OF THEM.

The largest two undertakings: First: we have rewritten all (half) of our API and elimited the usage of Node.js specific `Buffer` all over the place and have rewritten with using `Uint8Array` byte objects. Second: we went throuh our whole stack, rewrote imports and exports, replaced and updated dependencies all over and are now able to provide a hybrid CommonJS/ESM build, for all libraries. Both of these things are huge.

Together with some few other modifications this now allows to run each (maybe adding an asterisk for client and devp2p) of our libraries directly in the browser - more or less without any modifications - see the `examples/browser.html` file in each package folder for an easy to set up example.

This is generally a big thing for Ethereum cause this brings the full Ethereum Execution Layer (EL) protocol stack to the browser in an easy accessible way for developers, for the first time ever! 🎉

This will allow for easy-to-setup browser applications both around the existing as well as the upcoming Ethereum EL protocol stack in the future. 🏄🏾‍♂️ We are beyond excitement to see what you guys will be building with this for "Browser-Ethereum". 🤓

Browser is not the only thing though why this release round is exciting: default Shanghai hardfork, full Cancun support, significantly smaller bundle sizes for various libraries, new database abstractions, a simpler to use EVM, API clean-ups throughout the whole stack. These are just the most prominent additional things here to mention which will make the developer heart beat a bit faster hopefully when you are scanning to the vast release notes for every of the 15 (!) releases! 🧑🏽‍💻

So: jump right in and enjoy. We can't wait to hear your feedback and see if you agree that these releases are as good as we think they are. 🙂 ❤️

The EthereumJS Team

### Default Shanghai HF / Merge -> Paris Renaming / Full Cancun Hardfork Support

The Shanghai hardfork is now the default HF in `@ethereumjs/common` and therefore for all libraries who use a Common-based HF setting internally (e.g. Tx, Block or EVM), see PR [#2655](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2655).

Also the Merge HF has been renamed to Paris (`Hardfork.Paris`) which is the correct HF name on the execution side, see [#2652](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2652). To set the HF to Paris in Common you can do:

```ts
import { Chain, Common, Hardfork } from '@ethereumjs/common'
const common = new Common({ chain: Chain.Mainnet, hardfork: Hardfork.Paris })
```

And third on hardforks 🙂: the upcoming Cancun hardfork is now fully supported and all EIPs are included (see PRs [#2659](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2659) and [#2892](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2892)). The Cancun HF can be activated with:

```ts
import { Chain, Common, Hardfork } from '@ethereumjs/common'
const common = new Common({ chain: Chain.Mainnet, hardfork: Hardfork.Cancun })
```

Note that not all Cancun EIPs are in a `FINAL` EIP state though and particularly `EIP-4844` will likely still receive some changes.

### StateManager / Cache Refactoring

With this release the StateManager has been completely refactored, see PR [#2630](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2630) and [#2634](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2634). While the overall API has been preserved for the most part, the API DOES come with some changes where things needed a clean-up, which will need some adoption. The cache backend has been completely rewritten and there is now a cleaner separation between the StateManager and the account and storage caches. The `BaseStateManager` class - being rather restrictive than useful - has been removed.

All this makes it significantly easier to write an own StateManager implementation or customize the existing implementation.

To integrate an already existing StateManager implementation it is likely best to start from the updated `statemanager.ts` file including the `DefaultStateManager` class and re-include the own functionality parts in the existing methods there, or to inherit from this class if your changes are not so extensive.

### New Permanent Account and Storage LRU Caches

This release comes with a significantly more elaborate caching mechanism for account and storage caches, see PR [#2630](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2634), [#2634](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2634) and [#2618](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2618).

There are now two cache options available: an unbounded cache (`CacheType.ORDERED_MAP`) for short-lived usage scenarios (this one is the default cache) and a fixed-size cache (`CacheType.LRU`) for a long-lived large cache scenario.

Caches now "survive" a flush operation and especially long-lived usage scenarios will benefit from increased performance by a growing and more "knowing" cache leading to less and less trie reads.

Have a loot at the extended `CacheOptions` on how to use and leverage the new cache system.

### API Changes

Also along PR [#2630](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2630) and [#2634](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2634): the StateManager API has been significantly cleaned up with one of the major changes being `getAccount()` not returning an empty account any more if no result was found. While this needs some adoption this one single change makes state handling a lot cleaner.

API Change Summary:

```ts
getAccount(address: Address): Promise<Account> // old
getAccount(address: Address): Promise<Account | undefined> // new

putAccount(address: Address, account: Account): Promise<void> // old
putAccount(address: Address, account: Account | undefined): Promise<void> // new (now also allows for deletion)

accountIsEmpty(address: Address): Promise<boolean> // removed

setStateRoot(stateRoot: Uint8Array): Promise<void> // old
setStateRoot(stateRoot: Uint8Array, clearCache?: boolean): Promise<void> // new

clearCaches(): void // new
```

The `StateManagerInterface` has now been moved to the `@ethereum/common` package for more universal access and should be loaded from there with:

```ts
import type { StateManagerInterface } from '@ethereumjs/common'
```

### Hybrid CJS/ESM Build

We now provide both a CommonJS and an ESM build for all our libraries. 🥳 This transition was a huge undertaking and should make the usage of our libraries in the browser a lot more straight-forward, see PR [#2685](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2685), [#2783](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2783), [#2786](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2786), [#2764](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2764), [#2804](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2804) and [#2809](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2809) (and others). We rewrote the whole set of imports and exports within the libraries, updated or completely removed a lot of dependencies along the way and removed the usage of all native Node.js primitives (like `https` or `util`).

There are now two different build directories in our `dist` folder, being `dist/cjs` for the CommonJS and `dist/esm` for the `ESM` build. That means that direct imports (which you generally should try to avoid, rather open an issue on your import needs), need an update within your code (do a `dist` or the like code search).

Both builds have respective separate entrypoints in the distributed `package.json` file.

A CommonJS import of our libraries can then be done like this:

```ts
const { Chain, Common } = require('@ethereumjs/common')
const common = new Common({ chain: Chain.Mainnet })
```

And this is how an ESM import looks like:

```ts
import { Chain, Common } from '@ethereumjs/common'
const common = new Common({ chain: Chain.Mainnet })
```

Using ESM will give you additional advantages over CJS beyond browser usage like static code analysis / Tree Shaking which CJS can not provide.

Side note: along this transition we also rewrote our whole test suite (yes!!!) to now work with [Vitest](https://vitest.dev/) instead of `Tape`.

### Buffer -> Uint8Array

With these releases we remove all Node.js specific `Buffer` usages from our libraries and replace these with [Uint8Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) representations, which are available both in Node.js and the browser (`Buffer` is a subclass of `Uint8Array`). While this is a big step towards interoperability and browser compatibility of our libraries, this is also one of the most invasive operations we have ever done, see the huge changeset from PR [#2566](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2566) and [#2607](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2607). 😋

We nevertheless think this is very much worth it and we tried to make transition work as easy as possible.

#### How to upgrade?

For this library you should check if you use one of the following constructors, methods, constants or types and do a search and update input and/or output values or general usages and add conversion methods if necessary:

```ts
// statemanager / StateManagerInterface (in @ethereumjs/common)
StateManager.putContractCode(address: Address, value: Uint8Array): Promise<void>
StateManager.getContractCode(address: Address): Promise<Uint8Array>
StateManager.getContractStorage(address: Address, key: Uint8Array): Promise<Uint8Array>
StateManager.putContractStorage(address: Address, key: Uint8Array, value: Uint8Array): Promise<void>
StateManager.clearContractStorage(address: Address): Promise<void>
StateManager.getStateRoot(): Promise<Uint8Array>
StateManager.setStateRoot(stateRoot: Uint8Array, clearCache?: boolean): Promise<void>
StateManager.getProof?(address: Address, storageSlots: Uint8Array[]): Promise<Proof>
```

We have converted existing Buffer conversion methods to Uint8Array conversion methods in the [@ethereumjs/util](https://github.com/ethereumjs/ethereumjs-monorepo/tree/master/packages/util) `bytes` module, see the respective README section for guidance.

#### Prefixed Hex Strings as Default

The mixed usage of prefixed and unprefixed hex strings is a constant source of errors in byte-handling code bases.

We have therefore decided to go "prefixed" by default, see PR [#2830](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2830) and [#2845](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2845).

The `hexToBytes` and `bytesToHex` methods, also similar methods like `intToHex`, now take `0x`-prefixed hex strings as input and output prefixed strings. The corresponding unprefixed methods are marked as `deprecated` and usage should be avoided.

Please therefore check you code base on updating and ensure that values you are passing to constructors and methods are prefixed with a `0x`.

### Other Changes

- Support for `Node.js 16` has been removed (minimal version: `Node.js 18`), PR [#2859](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2859)
- `EthersStateManager` now uses `Ethers` `v6`, PR [#2720](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2720)
- Integrate an `OriginalStorage` cache, other VM/EVM EEI refactoring related changes, PR [#2702](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2702)
- Breaking: The `copy()` method has been renamed to `shallowCopy()` (same underlying state DB), PR [#2826](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2826)
- Breaking: `StateManager._common` property has been renamed to `StateManager.common` and made public, PR [#2857](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2857)

## 1.0.5 - 2023-04-20

- Update ethereum-cryptography from 1.2 to 2.0 (switch from noble-secp256k1 to noble-curves), PR [#2641](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2641)
- Bump `@ethereumjs/util` `@chainsafe/ssz` dependency to 0.11.1 (no WASM, native SHA-256 implementation, ES2019 compatible, explicit imports), PRs [#2622](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2622), [#2564](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2564) and [#2656](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2656)

## 1.0.4 - 2023-02-27

- Pinned `@ethereumjs/util` `@chainsafe/ssz` dependency to `v0.9.4` due to ES2021 features used in `v0.10.+` causing compatibility issues, PR [#2555](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2555)

## 1.0.3 - 2023-02-21

**DEPRECATED**: Release is deprecated due to broken dependencies, please update to the subsequent bugfix release version.

Maintenance release with dependency updates, PR [#2521](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2521)

## 1.0.2 - 2022-12-09

Added `EthersStateManager` to direct exports (if you use please fix our deep imports), see PR [#2419](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2419).

Import is now simplified to:

```ts
import { EthersStateManager } from '@ethereumjs/statemanager'
```

## 1.0.1 - 2022-10-18

### New EthersStateManager (experimental)

There is a new dedicated state manager `EthersStateManager` added to the library. This new state manager gets its state via Ethers RPC calls and allows e.g. for a stateless execution of selected mainnet (or other Ethereum network) blocks and transactions, see PR [#2315](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2315).

There are some caveats and therefore the new state manager is marked as `experimental` for now:

- No `stateRoot` calculation for now, only secondary measures like `gasUsed`
- Performance is rather slow (when using a remote provider for state data), particularly for more recent `mainnet` blocks
- The API of this new state manager might change in future (bugfix or minor) releases

This should nevertheless be useful for a certain number of use cases, if there is e.g. the need for some quick analysis of certain mainnet (EVM) behavior e.g.. The `StateManager` package [README](https://github.com/ethereumjs/ethereumjs-monorepo/tree/master/packages/statemanager#ethersstatemanager) contains a new dedicated section on how to use the new state manager.

**Note:** Usage of this StateManager can cause a heavy load regarding state request API calls, so be careful (or at least: aware) if used in combination with an Ethers provider connecting to a third-party API service like Infura!

### Other Changes and Fixes

- Migrated from `rbtree` to [js-sdsl](https://github.com/js-sdsl/js-sdsl) package for caching functionality (maintained, better performance), PR [#2285](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2285)

## 1.0.0 - 2022-09-06

Final release - tada 🎉 - of a wider breaking release round on the [EthereumJS monorepo](https://github.com/ethereumjs/ethereumjs-monorepo) libraries, see the Beta 1 release notes for the main long change set description as well as the Beta 2, Beta 3 and Release Candidate (RC) 1 release notes for notes on some additional changes ([CHANGELOG](https://github.com/ethereumjs/ethereumjs-monorepo/blob/master/packages/statemanager/CHANGELOG.md)).

### Changes

- Internal refactor: removed ambiguous boolean checks within conditional clauses, PR [#2251](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2251)

## 1.0.0-rc.1 - 2022-08-29

Release candidate 1 for the upcoming breaking release round on the [EthereumJS monorepo](https://github.com/ethereumjs/ethereumjs-monorepo) libraries, see the Beta 1 release notes for the main long change set description as well as the Beta 2 and 3 release notes for notes on some additional changes ([CHANGELOG](https://github.com/ethereumjs/ethereumjs-monorepo/blob/master/packages/statemanager/CHANGELOG.md)).

### Other Changes

- \*\*Attention:" Removed unused `common` option from `StateManager`, PR [#2197](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2197)
- Added `prefixCodeHashes` flag defaulting to `true` which allows to deactivate prefix code hashes saved in the DB (see PR [#1438](https://github.com/ethereumjs/ethereumjs-monorepo/pull/1438) for context), PR [#2179](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2179)

### Maintenance Updates

- Added `engine` field to `package.json` limiting Node versions to v14 or higher, PR [#2164](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2164)
- Replaced `nyc` (code coverage) configurations with `c8` configurations, PR [#2192](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2192)
- Code formats improvements by adding various new linting rules, see Issue [#1935](https://github.com/ethereumjs/ethereumjs-monorepo/issues/1935)

## 1.0.0-beta.3 - 2022-08-10

Beta 3 release for the upcoming breaking release round on the [EthereumJS monorepo](https://github.com/ethereumjs/ethereumjs-monorepo) libraries, see the Beta 1 release notes for the main long change set description as well as the Beta 2 release notes for notes on some additional changes ([CHANGELOG](https://github.com/ethereumjs/ethereumjs-monorepo/blob/master/packages/statemanager/CHANGELOG.md)).

### Merge Hardfork Default

Since the Merge HF is getting close we have decided to directly jump on the `Merge` HF (before: `Istanbul`) as default in the underlying `@ethereumjs/common` library and skip the `London` default HF as we initially intended to set (see Beta 1 CHANGELOG), see PR [#2087](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2087).

This change should not directly affect this library but might be relevant since it is not recommended to use different Common library versions between the different EthereumJS libraries.

### Other Changes

- Upgrades the `@ethereumjs/trie` library to Beta 3 which allows to pass in a custom hash function/library (for performance), PR [#2043](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2043)

## 1.0.0-beta.2 - 2022-07-15

Beta 2 release for the upcoming breaking release round on the [EthereumJS monorepo](https://github.com/ethereumjs/ethereumjs-monorepo) libraries, see the Beta 1 release notes ([CHANGELOG](https://github.com/ethereumjs/ethereumjs-monorepo/blob/master/packages/statemanager/CHANGELOG.md)) for the main change set description.

### Removed Default Exports

The change with the biggest effect on UX since the last Beta 1 releases is for sure that we have removed default exports all accross the monorepo, see PR [#2018](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2018), we even now added a new linting rule that completely disallows using.

Default exports were a common source of error and confusion when using our libraries in a CommonJS context, leading to issues like Issue [#978](https://github.com/ethereumjs/ethereumjs-monorepo/issues/978).

Now every import is a named import and we think the long term benefits will very much outweigh the one-time hassle of some import adoptions.

#### Common Library Import Updates

Since our [@ethereumjs/common](https://github.com/ethereumjs/ethereumjs-monorepo/tree/master/packages/common) library is used all accross our libraries for chain and HF instantiation this will likely be the one being the most prevalent regarding the need for some import updates.

So Common import and usage is changing from:

```ts
import Common, { Chain, Hardfork } from '@ethereumjs/common'

const common = new Common({ chain: Chain.Mainnet, hardfork: Hardfork.Merge })
```

to:

```ts
import { Common, Chain, Hardfork } from '@ethereumjs/common'

const common = new Common({ chain: Chain.Mainnet, hardfork: Hardfork.Merge })
```

### Removed Default Imports in this Library

The main `DefaultStateManager` class import has been updated, so import changes from:

```ts
import DefaultStateManager from '@ethereumjs/statemanager'
```

to:

```ts
import { DefaultStateManager } from '@ethereumjs/statemanager'
```

## Other Changes

- Added `ESLint` strict boolean expressions linting rule, PR [#2030](https://github.com/ethereumjs/ethereumjs-monorepo/pull/2030)

## 1.0.0-beta.1 - 2022-06-30

This release is part of a larger breaking release round where all [EthereumJS monorepo](https://github.com/ethereumjs/ethereumjs-monorepo) libraries (VM, Tx, Trie, other) get major version upgrades. This round of releases has been prepared for a long time and we are really pleased with and proud of the result, thanks to all team members and contributors who worked so hard and made this possible! 🙂 ❤️

We have gotten rid of a lot of technical debt and inconsistencies and removed unused functionality, renamed methods, improved on the API and on TypeScript typing, to name a few of the more local type of refactoring changes. There are also broader structural changes like a full transition to native JavaScript `BigInt` values as well as various somewhat deep-reaching refactorings, both within a single package as well as some reaching beyond the scope of a single package. Also two completely new packages - `@ethereumjs/evm` (in addition to the existing `@ethereumjs/vm` package) and `@ethereumjs/statemanager` - have been created, leading to a more modular Ethereum JavaScript VM.

We are very much confident that users of the libraries will greatly benefit from the changes being introduced. However - along the upgrade process - these releases require some extra attention and care since the changeset is both so big and deep reaching. We highly recommend to closely read the release notes, we have done our best to create a full picture on the changes with some special emphasis on delicate code and API parts and give some explicit guidance on how to upgrade and where problems might arise!

So, enjoy the releases (this is a first round of Beta releases, with final releases following a couple of weeks after if things go well)! 🎉

The EthereumJS Team

### New Package

The `StateManager` has been extracted from the `VM` and is now a separate package, see PR [#1817](https://github.com/ethereumjs/ethereumjs-monorepo/pull/1817). The new package can be installed separately with:

```shell
npm i @ethereumjs/statemanager
```

The `@ethereumjs/vm` package still has this package added as a dependency and it is automatically integrated. The `StateManager` provides a high-level interface to an underlying state storage solution. This is classically a `Trie` (in our case: an `@ethereumjs/trie`) instance, but can also be something else, e.g. a plain database, an underlying RPC connection or a Verkle Tree in the future.

The extraction of this module allows to easier customize a `StateManager` and provide or use your own implementations in the future. It is now also possible to use the `StateManager` standalone for high-level state access in a non-VM context.

A `StateManager` must adhere to a predefined interface `StateManager` and implement a certain set of state access methods like `getAccount()`, `putContractCode()`,... Such an implementation is then guaranteed to work e.g. in the `@ethereumjs/vm` implementation.

### StateManager Refactoring

Along with the package extraction parts of the old `StateManager` has also been reworked. So if you are building on the old `StateManager` class/interface it is likely not enough to just change on the import statement but do some adjustments to get things working. Here is a summary of the changes.

Methods added:

- `flush()`

Methods removed:

- `touchAccount()` (EVM-specific, remained in `EVMStateAccess` interface in EVM)
- All methods from `EIP2929StateManager` (removed as separate interface) (EVM-specific, remained in `EVMStateAccess` interface in EVM)
- `getOriginalContractStorage()` (EVM-specific, remained in `EVMStateAccess` interface in EVM)
- `hasGenesisState()` (removed)
- `generateGenesis()` (removed)
- `generateCanonicalGenesis()` (EVM-specific, remained in `EVMStateAccess` interface in EVM)
- `cleanupTouchedAccounts()` (EVM-specific, remained in `EVMStateAccess` interface in EVM)
- `clearOriginalStorageCache()` (EVM-specific, remained in `EVMStateAccess` interface in EVM)

Other Changes:

- New partial parent interface `StateAccess` with just the access focused functionality

So overall the `StateManager` interface got a lot leaner requiring fewer methods to be implemented which should make an implementation and/or adoption a lot easier.

The `StateManager` package ships with a Trie-based `StateManager` implementation extending from a `BaseStateManager` which might be a suitable starting point for your own implementations. This will very much depend on the specific needs though.

### BigInt Introduction / ES2020 Build Target

With this round of breaking releases the whole EthereumJS library stack removes the [BN.js](https://github.com/indutny/bn.js/) library and switches to use native JavaScript [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) values for large-number operations and interactions.

This makes the libraries more secure and robust (no more BN.js v4 vs v5 incompatibilities) and generally comes with substantial performance gains for the large-number-arithmetic-intense parts of the libraries (particularly the VM).

To allow for BigInt support our build target has been updated to [ES2020](https://262.ecma-international.org/11.0/). We feel that some still remaining browser compatibility issues on the edges (old Safari versions e.g.) are justified by the substantial gains this step brings along.

See [#1671](https://github.com/ethereumjs/ethereumjs-monorepo/pull/1671) and [#1771](https://github.com/ethereumjs/ethereumjs-monorepo/pull/1771) for the core `BigInt` transition PRs.

### Disabled esModuleInterop and allowSyntheticDefaultImports TypeScript Compiler Options

The above TypeScript options provide some semantic sugar like allowing to write an import like `import React from "react"` instead of `import * as React from "react"`, see [esModuleInterop](https://www.typescriptlang.org/tsconfig#esModuleInterop) and [allowSyntheticDefaultImports](https://www.typescriptlang.org/tsconfig#allowSyntheticDefaultImports) docs for some details.

While this is convenient, it deviates from the ESM specification and forces downstream users into using these options, which might not be desirable, see [this TypeScript Semver docs section](https://www.semver-ts.org/#module-interop) for some more detailed argumentation.

Along with the breaking releases we have therefore deactivated both of these options and you might therefore need to adapt some import statements accordingly. Note that you still can activate these options in your bundle and/or transpilation pipeline (but now you also have the option _not_ to, which you didn't have before).
