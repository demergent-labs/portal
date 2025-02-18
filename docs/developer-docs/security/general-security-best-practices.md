---
sidebar_position: 1
sidebar_label: General
---
# General security best practices

## Certify query responses if they are relevant for security

### Security Concern

The responses to [query calls](/references/ic-interface-spec.md#https-interface) (as opposed to update calls) are not threshold-signed by the canister/subnet. Thus, a single malicious replica or boundary node may change the data, violating its authenticity. This is especially risky if update calls depend on the response to query calls.

### Recommendation

-   All security-relevant query response data that needs authenticity guarantees (this needs to be assessed for each dApp) should be certified by the IC using certified variables. Consider using existing data structures such as [certified-map](https://github.com/dfinity/cdk-rs/tree/main/library/ic-certified-map). The data certification must be validated in the frontend.

-   Alternatively, these calls have to be issued as update calls by the caller (e.g. in agent-js), but that impacts performance: it takes a few seconds. Note that every query can also be issued as an update by the caller.

-   Examples are asset certification in [Internet Identity](https://github.com/dfinity/internet-identity/blob/b29a6f68bbe5a49d048e12bc7a3263a9f43d080b/src/internet_identity/src/main.rs#L775-L808), [NNS dApp](https://github.com/dfinity/nns-dapp/blob/372c3562127d70c2fde059bc9c268e8ae858583e/rs/src/assets.rs#L121-L145), or the [canister signature implementation in Internet Identity](https://github.com/dfinity/internet-identity/blob/main/src/internet_identity/src/signature_map.rs).

## Nonspecific to the Internet Computer

The best practices in this section are very general and not specific to the Internet Computer. This list is by no means complete and only lists a few very specific concerns that have led to issues in the past.

### Don’t use third-party components with known vulnerabilities

#### Security Concern

Using vulnerable and outdated components is a [big security risk](https://owasp.org/Top10/A06_2021-Vulnerable_and_Outdated_Components/).

#### Recommendation

-   Regularly check your third party components against databases of known vulnerabilities:

-   Rust: use [cargo audit](https://crates.io/crates/cargo-audit).

-   JavaScript / NPM: use [npm audit](https://docs.npmjs.com/cli/v8/commands/npm-audit)

-   This should be integrated into the build process, the build should fail if there are known vulnerabilities.

-   Don’t use forked versions of repositories that are not maintained and may not be trustworthy.

-   Avoid using third party components that are not widely used and may not have had sufficient (ideally third party) review.

-   Pin the versions of the components you are using to avoid switching to corrupt updates automatically.

### Don’t implement crypto yourself

#### Security Concern

It is easy to make mistakes when implementing cryptographic algorithms, leading to security bugs.

#### Recommendation

-   Use well known libraries that may be open source and have been reviewed by many people. For example, use the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) in JavaScript, use crates such as [sha256](https://crates.io/crates/sha256) in Rust

### Use secure cryptographic schemes

#### Security Concern

Some cryptographic schemes have been broken (old TLS versions, MD5, SHA1, DES, …​), or they could be so new that they have not yet been appropriately researched. Using these introduces security issues.

#### Recommendation

If you need to use crypto, only use cryptographic schemes that have not been broken and do not have known issues. Ideally use algorithms that have been standardized by e.g. NIST or IETF.

References:

-   [OWASP Cryptographic Failures](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)

### Test your code

#### Security Concern

Having small test coverage is risky, as code changes become difficult and may violate correctness and security properties, leading to bugs. It is hard to verify correctness and security properties in reviews (and security reviews) if there are no corresponding tests.

#### Recommendation

Write tests for canister implementations and frontend code, especially for security relevant properties and invariants.

-   In [Effective Rust Canisters](https://mmapped.blog/posts/01-effective-rust-canisters.html): [test upgrades](https://mmapped.blog/posts/01-effective-rust-canisters.html#test-upgrades), [make code target-independent](https://mmapped.blog/posts/01-effective-rust-canisters.html#target-independent)

-   See also [Test your canister code even in presence of System API calls](rust-canister-development-security-best-practices#test-your-canister-code)

-   Consider the [DFINITY Rust guidelines on testing](https://docs.dfinity.systems/dfinity/spec/meta/rust.html#_tests)

-   For wasm-level unit testing, consider using [Motoko Matchers](https://github.com/kritzcreek/motoko-matchers)

-   For Motoko-level unit testing, consider [the Canister module](https://kritzcreek.github.io/motoko-matchers/Canister.html). There are also some example tests [here](https://github.com/dfinity/motoko-base/blob/master/test/resultTest.mo) and [here](https://github.com/dfinity/motoko-base/blob/master/test/textTest.mo). As an example see also the end-to-end tests and unit tests for the [invoice canister](https://github.com/dfinity/invoice-canister).

-   For long-running test scenarios, consider [Motoko BigTest](https://github.com/matthewhammer/motoko-bigtest)

### Avoid test and dev code in production

#### Security Concern

It is risky to include code paths in production code that are only used for development or testing setups. If something goes wrong (and it sometimes does!), this may introduce security bugs in production.

For example, we have seen issues where the public key to verify certification was fetched from an untrusted source, since this is what is done on test networks.

#### Recommendation

Avoid test and dev code in production code whenever possible.
