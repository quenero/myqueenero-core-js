# MyQueenero Core JS

### Info

1. Legal
2. What's in This Repo?
3. Usage

### Contributing

1. QA
2. Pull Requests
3. Building MyQueeneroCoreCpp from Scratch
4. Contributors


# Info

## Legal

See `LICENSE.txt` for license.

All source code copyright © 2014-2018 by MyMonero. All rights reserved.

## What's in This Repo?

This repository holds the Javascript source code for the Queenero/CryptoNote cryptography and protocols, plus lightwallet functions which power the official [MyQueenero](https://www.myqueenero.xyz) apps.

There is also a chain of build scripts which is capable of building a JS module by transpiling a subset of Queenero source code via emscripten, which relies upon static boost libs, for which there is also a script for building from source. 

It's possible to run your own lightweight (hosted) wallet server by using either OpenQueenero or the lightweight wallet server which MyQueenero has developed specially to be open-sourced for the Queenero community (PR is in the process of being merged). However, MyQueenero also offers highly optimized, high throughput, secure hosting for a nominal, scaling fee per active wallet per month to wallet app makers who would like to use this library, myqueenero-core-js, to add hosted Queenero wallets to their app. 



### Contents 

`queenero_utils` contains Queenero- and MyQueenero-specific implementations, wrappers, and declarations, and the MyQueenero JS and wasm (and asm.js fallback/polyfill) implementations for the underlying cryptography behind Queenero. 

`queenero_utils/MyQueeneroCoreCpp*` are produced by transpiling Queenero core C++ code to JS via Emscripten (See *Building MyQueeneroCoreCpp*). A Module instance is managed by `queenero_utils/MyQueeneroCoreBridge.js`.

Library integrators may use `MyQueeneroCoreBridge` by `require("./queenero_utils/MyQueeneroCoreBridge")({}).then(function(coreBridge_instance) { })`. (This was formerly accessed via the now-deprecated `queenero_utils/queenero_utils`). You may also access this MyQueeneroCoreBridge promise via the existing `index.js` property `queenero_utils_promise` (the name has been kept the same for API stability). 

Many related utility functions and data structures are located throughout `queenero_utils/`, `cryptonote_utils`, and `hostAPI`. Usage below.

Various convenience scripts are provided in `./bin`.

This readme is located at `README.md`, and the license is located at `LICENSE.txt`.


## Usage

If you would like to package this library to run in a standalone manner within, e.g. a webpage, similarly to how the old myqueenero.com used this library, a script is provided to bundle everything for you. It's located at `bin/package_browser_js`. If you package the library in this manner, the resulting `myqueenero-core.js` file can be included via a script tag. The index.js of the library will then be available as the global variable `myqueenero_core_js`.

Alternatively you can bundle the contents in any other manner you prefer, including directly accessing them via your favorite module system. 


### `hostAPI`

Use the functions in the modules in `hostAPI` for convenience implementations of (a) networking to a MyQueenero-API-compatible server, (b) constructing common request bodies, and (c) parsing responses.

For a working example usage of `hostAPI`, see [myqueenero-app-js/HostedQueeneroAPIClient](https://github.com/myqueenero/myqueenero-app-js/blob/master/local_modules/HostedQueeneroAPIClient). However, there's no need to conform to this example's implementation, especially for sending transactions, as the response parsing and request construction for transactions is now handled within the implementation.

#### Examples

```
const endpointPath = "get_address_info"
const parameters = net_service_utils.New_ParametersForWalletRequest(address, view_key__private)
```

```	
queenero_utils_promise.then(function(queenero_utils) {
	response_parser_utils.Parsed_AddressTransactions__keyImageManaged(
		data,
		address, view_key__private, spend_key__public, spend_key__private,
		queenero_utils,
		function(err, returnValuesByKey) {
			…
		}
	)
})
```
where `data` is the JSON response. Note you must pass in a `resolve`d `queenero_utils` instance (see below for usage) so that such functions can remain synchronous without having to wait for the promise.

`__keyImageManaged` means that key images will be generated and then cached for a large performance boost for you. The caveat of this convenience is that you should make sure to call `DeleteManagedKeyImagesForWalletWith` on `queenero_utils/queenero_keyImage_cache_utils` (below) when you're done with them, such as on the teardown of a related wallet instance.

----
### `cryptonote_utils/mnemonic_languages`

It's not generally at all necessary to interact with this module unless you want to, e.g., construct a GUI that needs a list of support mnemonics.

In other words, if your app only needs to generate a mnemonic, you can avoid using this code module entirely by simply passing a language code (of "en", "en-US", "ja", "zh" etc) to the below `queenero_utils` function which generates wallets.

-----
### `cryptonote_utils/nettype`

You'll need this module to construct the `nettype` argument for passing to various other functions.

#### Examples

`const nettype = require('cryptonote_utils/nettype').network_type.MAINNET`

-----
### `cryptonote_utils/biginteger`

Used extensively for managing Queenero amounts in atomic units to ensure precision.

#### Examples

```
const JSBigInt = require('../myqueenero_core_js/cryptonote_utils/biginteger').BigInteger
const amount = new JSBigInt('12300000')
const amount_str = queenero_amount_format_utils.formatMoney(amount)
```

-----
### `cryptonote_utils/money_format_utils`

It's not necessary to use this module directly. See `queenero_utils/queenero_amount_format_utils`.

-----
### `queenero_utils/queenero_amount_format_utils`

```
const queenero_amount_format_utils = require("queenero_utils/queenero_amount_format_utils");
const formatted_string = queenero_amount_format_utils.formatMoney(a JSBigInt)
```

Functions: `formatMoneyFull`, `formatMoneyFullSymbol`, `formatMoney`, `formatMoneySymbol`, `parseMoney(str) -> JSBigInt`

-----
### `queenero_utils/queenero_txParsing_utils`

Use these functions to derive additional state from transaction rows which were returned by a server and then parsed by `hostAPI`. 

* `IsTransactionConfirmed(tx, blockchain_height)`
* `IsTransactionUnlocked(tx, blockchain_height)`
* `TransactionLockedReason(tx, blockchain_height)`

-----
### `queenero_utils/queenero_keyImage_cache_utils`

Use these functions to directly interact with the key image cache.

* `Lazy_KeyImage(…)` Generate a key image directly and cache it. Returns cached values.
* `DeleteManagedKeyImagesForWalletWith(address)` Call this to avoid leaking keys if you use any of the response parsing methods (above) which are suffixed with `__keyImageManaged`.

-----
### `queenero_utils/queenero_paymentID_utils`

Contains functions to validating payment ID formats. To generate payment IDs, see `queenero_utils`.

-----
### `queenero_utils/queenero_sendingFunds_utils`

Used to contain a convenience implementation of `SendFunds(…)` for constructing arguments to `create_transaction`-type functions. However that's been moved to C++ and exposed via a single function on `MyQueeneroCoreBridge` called `async__send_funds`.

One of the callbacks to this function, `status_update_fn`, supplies status updates via codes that can be translated into messages. Codes are located on `SendFunds_ProcessStep_Code` and messages are located at `SendFunds_ProcessStep_MessageSuffix` within this file, `queenero_sendingFunds_utils`. This lookup will probably make it into `MyQueeneroCoreBridge` for concision.


----
### `queenero_utils/MyQueeneroCoreBridge`

#### Examples

```
const myqueenero = require("myqueenero_core_js/index");
// or just "myqueenero_core_js/queenero_utils/MyQueeneroCoreBridge"
async function foo()
{
	const queenero_utils = await myqueenero.queenero_utils_promise;
	const nettype = myqueenero.nettype_utils.network_type.MAINNET;
	const decoded = queenero_utils.address_and_keys_from_seed("…", nettype);
	// read decoded.address_string
	//
}
foo()
```

```
var decoded = queenero_utils.decode_address("…", nettype);
```

#### Available functions

Each of these functions is implemented<sup>*</sup> in `queenero_utils/MyQueeneroCoreBridge.js`.

The arguments and return values for these functions are explicitly called out by [MyQueeneroCoreBridge.js](https://github.com/myqueenero/myqueenero-core-js/blob/develop/queenero_utils/MyQueeneroCoreBridge.js), so that will be the most complete documentation for the moment. Return values are all embedded within a JS dictionary unless they're singular values. Errors are thrown as exceptions.

<sup>* The functions' actual implementations are in WebAssembly which is produced via emscripten from exactly matching C++ functions in [myqueenero-core-cpp](https://github.com/myqueenero/myqueenero-core-cpp). This allows core implementation to be shared across all platforms.</sup>


```
is_subaddress
```
```
is_integrated_address
```
```
new_payment_id
```
```
new__int_addr_from_addr_and_short_pid
```
```
decode_address
```
```
newly_created_wallet
```
```
are_equal_mnemonics
```
```
mnemonic_from_seed
```
```
seed_and_keys_from_mnemonic
```
```
validate_components_for_login
```
```
address_and_keys_from_seed
```
* This function was known as `create_address` in the previous myqueenero-core-js API.

```
generate_key_image
```
```
generate_key_derivation
```
```
derivation_to_scalar
```
```
derive_public_key
```
```
derive_subaddress_public_key
```
```
decodeRct
```
```
decodeRctSimple
```

```
estimate_fee
```
```
estimated_tx_network_fee
```
```
estimate_tx_weight
```
```
estimate_rct_tx_size
```

```
async__send_funds
```
* This method takes simple, familiar parameters in the form of a keyed dictionary, and has a handful of callbacks which supply pre-formed request parameters for sending directly to a MyQueenero or lightweight wallet-compatible API server. Responses may be sent directly back to the callbacks' callbacks, as they are now parsed and handled entirely within the implementation. This function's interface used to reside in `queenero_sendingFunds_utils`. See `tests/sendingFunds.spec.js` for example usage.


# Contributing

## QA

Please submit any bugs as Issues unless they have already been reported.

Suggestions and feedback are very welcome!

## Pull Requests

We'll merge nearly anything constructive. Contributors welcome and credited in releases.

We often collaborate over IRC in #myqueenero on Freenode.

**All development happens off the `develop` branch like the Gitflow Workflow.**

## Building MyQueeneroCoreCpp from Scratch

There's no need to build queenero_utils/MyQueeneroCoreCpp as a build is provided, but if you were for example interested in adding a C++ function, you could use the information in this section to transpile it to JS.

### Repository Setup

* Execute `bin/update_submodules` 


### Install Emscripten SDK

**A version of emscripten of at least 1.38.13 with [these updates](https://github.com/kripken/emscripten/pull/7096) is required so that random bit generation safety can be ensured.**

Ensure you've [properly installed Emscripten](http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html) and exposed the Emscripten executables in your PATH, e.g.:

	source ./emsdk_env.sh


### Boost for Emscripten

*Depends upon:* Emscripten SDK

Download a copy of the contents of the Boost source into `./contrib/boost-sdk/`.

* Execute `bin/build-boost-emscripten.sh`



### Emscripten Module

*Depends upon:* Repository Setup, Emscripten SDK, Boost for Emscripten

* Execute `bin/build-emcpp.sh`

Or if you want to copy the build products to their distribution locations, 

* Execute `bin/archive-emcpp.sh`

**NOTE** If you want to build for asmjs instead of wasm, edit `CMakeLists.txt` to turn the `MM_EM_ASMJS` option to `ON` before you run either the `build` or `archive` script. Finally, at every place you instantiate a `MyQueeneroCoreBridge` instance, ensure that the `asmjs` flag passed as an init argument is set to `true` (If not, loading will not work). 


## Maintainers and Advisors

* 💿 `endogenic` ([Paul Shapiro](https://github.com/paulshapiro)) Maintainer

* 🍄 `luigi` Major contiributor of original JS core crypto and Queenero-specific routines; Advisor


## Authors

* Paul Shapiro

* luigi1111

* Lucas Jones     

* gutenye

* HenryNguyen5

* cryptochangements34

* bradoyler

* rex4539

* paullinator
