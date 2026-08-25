# Web3 and secp256k1 migration

## Scope and commits

Task 3 consolidates the wallet's required Web3Swift and libsecp256k1 surface
inside TLCore. The source inputs were fixed to these clean repositories:

- Web3Swift: `2b24ba4e65a3cf2026d697396a95ba7e1937e325`
- secp256k1: `5d74ae264f59f5b98c5832f79cc33c2ed9bad82d`

The implementation moved through these repository boundaries:

| Repository | Before | After | Commit |
| --- | --- | --- | --- |
| TLCore | `c84aa8d5d4315001fe530357a63c3b1bc0ef0af9` | `7f20566c229c82c6abe731f0e941addcbcdaf17a` | `refactor: embed the required Web3 and secp256k1 APIs` |
| Main app | `664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b` | `b87d495eaac377a9ade9f3b6baa2710696a76b85` | `refactor: consume Web3 APIs from TLCore` |

The main commit contains `Podfile`, 11 source files, and only the project-file
hunks that remove obsolete PromiseKit, scrypt, and web3swift header/link
settings. The pre-existing GasFree project-file reorder remains unstaged.
`Podfile.lock` also remains unstaged because it had a user-owned pre-task diff
that could not be separated safely. The local validation lock resolved TLCore
through `/Users/viccc/source/tronlink-iOS-core`; it is evidence, not a commit
input.

## Imported source boundary

All 45 paths under `tronlink-iOS-core/Classes/Secp256k1` match the pinned
source byte-for-byte. The only path adaptation is
`secp256k1.c -> wallet_secp256k1.c`, which prevents a basename/object collision
with TrezorCrypto. Both files have SHA-256
`335ffe244402cd7b228a38cad9b43f7c7a3da511c36aecaacb5206996def154d`.

The exact Web3 subset is 21 files: 19 pinned whitelist inputs plus two focused
compatibility files.

```text
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Decoding.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Elements.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Encoding.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2ParameterTypes.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Parsing.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2TypeParser.swift
tronlink-iOS-core/Classes/Web3Subset/Crypto/LibSecp256k1Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Crypto/PrivateKey.swift
tronlink-iOS-core/Classes/Web3Subset/Crypto/keccak.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Array+Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/CryptoExtensions.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Data+Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Int+Sequence.swift
tronlink-iOS-core/Classes/Web3Subset/Support/NSRegularExpressionExtension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/NativeTypesEncoding+Extensions.swift
tronlink-iOS-core/Classes/Web3Subset/Support/String+Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/UInt256.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Web3Address.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Web3CompatibilityTypes.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Web3Utils.swift
```

The two licenses are exact copies. Their SHA-256 values are
`9e886690e7511028efff4d42b2f4bf1efa4730252dbfb9a422fb8f311b845bfd`
for Web3Swift and
`ea3c96f6e9b90d277a8577868e8063eed03340b05d725cad61b353a16acbbe5f`
for secp256k1.

### Explicit exclusions

The subset does not import Base58, RIPEMD160, AES/Cryptor, DerivedKey, legacy
scrypt, Web3 keystores, RLP/transaction implementations, contracts, providers,
networking, Objective-C bridges, Ethereum APIs, EIPs, ERC implementations, or
PromiseKit. TLCore already owns the required address, encoding, hashing, and
keystore primitives; importing those families would duplicate behavior or
reintroduce the dependencies this stage removes.

The exclusion scan has no executable-code matches. Three textual matches are
upstream comments/examples only: two ERC20 documentation lines in
`Web3Address.swift` and one ERC20 compatibility comment in
`ABIv2Decoding.swift`.

## Source adaptations

- Removed `import tron_wallet_secp256k1`; the TLCore umbrella exposes
  `secp256k1-wallet.h` from the embedded tree.
- Renamed Web3Swift's `Address`/`AddressError` surface to
  `Web3Address`/`Web3AddressError`, leaving TLCore's existing `Address` intact.
- Removed the copied `Data.hex: String` and `String.hex: Data` declarations;
  callers use the existing TLCore helpers with the same required lowercase and
  odd-nibble behavior.
- Adapted `Data.fromHex` and `String.dataFromHex()` to the existing TLCore hex
  initializer while preserving legacy empty-data rejection.
- Added only the ABIv2-reachable `Web3Error`, `EventLog`, Codable
  `Web3Address`, approved `Web3Utils` methods, and exact upstream `Web3Units`.
- Pruned unused legacy scrypt/PBKDF and Cryptor-backed digest tails.
  `Web3Utils.sha256` delegates to the existing CryptoSwift-backed
  `Data.sha256T()`.
- Removed copied `Data.bytes` after the main test target proved it collided
  with `CryptoSwift.Data.bytes`. Four internal Web3 call sites now use explicit
  `Array(data)` conversions. A whole copied-Data/String member scan has no
  remaining CryptoSwift intersection.
- `TLWalletCore` now calls `Web3Utils.hashECRecover` directly.

The main migration applies only the approved module/type mappings. The initial
module-import scan found 10 files. A subsequent test-target compile exposed one
additional indirect dependency on Web3Swift's global Objective-C
`NSData.hexString()` bridge. That 11th file now imports TLCore and uses
`Data(zeroArray).hexString`, which preserves lowercase output.

## Build configuration

The podspec keeps libsecp implementation headers private, exposes only the
umbrella-referenced public header, and adds these C flags:

```text
$(inherited) -pedantic -Wall -Wextra -Wcast-align -Wnested-externs
-Wshadow -Wstrict-prototypes -Wno-shorten-64-to-32
-Wno-conditional-uninitialized -Wno-unused-function -Wno-long-long
-Wno-overlength-strings -O3
```

No field/scalar configuration macro is supplied by the build. The pinned
`secp256k1-config.h` remains authoritative.

After dependency removal, both Example locks and both main validation locks
contain zero occurrences of `TronWalletWeb3Swift`,
`tron-wallet-secp256k1`, `PromiseKit`, or `scrypt.c`. Example `pod install`
removed all four old pod targets. The main project also needed its pre-existing
hard-coded PromiseKit header/link settings and scrypt/web3swift linker entries
removed; generated Pods xcconfigs already omitted them.

## Validation evidence

| Gate | Result | Artifact |
| --- | --- | --- |
| Accepted RED | Exit 65; only the planned missing TLCore Web3 symbols | `/tmp/tlcore-stage-3-red-valid.log`, SHA-256 `b7846de227a3043208d241b256c07f1116067ecd5bf748f498181d4a9b275bd1` |
| Dynamic legacy differential | 3 passed, 0 failed | `/tmp/tlcore-stage-3-differential-structural-green.log`, SHA-256 `1eb769d05ce5a29dce4e5509fbe15827f33f71b19927b392845be6a613369523` |
| Post-removal Web3 golden | 5 passed, 0 failed | `/tmp/tlcore-stage-3-collision-fix-golden.log`, SHA-256 `1391b1210271bf3dba46b2a76a9157ec07fca8874dd3de2ec01d5a2139af4caf` |
| Accepted full Example suite | 61 passed, 0 failed, 0 skipped in the collected result, with exactly four known tests excluded by selector | `/tmp/tlcore-stage-3-complete-example.log`, SHA-256 `596bd9821661a503fc5fc643534cf88d034b9ee2e00325d4817c4f3149aa5fd1` |
| CocoaPods lint | Passed with warnings under the established Xcode 26 iOS 13 xcconfig override | `/tmp/tlcore-stage-3-pod-lib-lint-ios13-xcconfig.log`, SHA-256 `86da45acda070a1e809855f0a548c00697787b062b6839672b6326bcf6fac300` |
| Main Debug simulator build | Exit 0 | `/tmp/tlcore-stage-3-main-build-final.log`, SHA-256 `3f7ee59a7958e2b8bb9060d853f99afe9429aa3df97d3037f75fe9935eca48f7` |
| Main focused tests | 8 passed, 0 failed | `/tmp/tlcore-stage-3-main-focused-eight-tests.log`, SHA-256 `dacbe7dbe27e4777ddf5ee9620bbdbe4611525ecbd36bc18295ab668392317ad` |

The accepted full-suite xcresult is
`/tmp/tlcore-stage-3-complete-example/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_18-57-13-+0800.xcresult`, tree-content SHA-256
`09edbe05aeb3d699d1ba2bfd50bfc538c5a8789ff3a77e8b6ebc1fdc59a0bd4b`.
Its 61 tests include all five Web3 golden tests. The four command-level
exclusions are the two previously accepted legacy tests and two previously
accepted ASCII-keystore incompatibility tests; none is a Task 3 test.

The post-`Data.bytes` focused xcresult has tree-content SHA-256
`8ffa57bd9d919a535cdcdac7e9defa8781fffa576ccdf319fb73396a734cf9f9`.
The main eight-test xcresult has tree-content SHA-256
`019493b80bbaa0d3fed0f06f9e7e9780a45d35babfd2ed14aa3bc82d6046cb9c`.

Plain `pod lib lint` is expected to fail under Xcode 26 because FMDB's
historical deployment target requests the removed `libarclite`; the accepted
run uses the Task 2 workaround
`XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig`, containing
`IPHONEOS_DEPLOYMENT_TARGET = 13.0`. It changes no repository file.

## Header and symbol evidence

The built TLCore public Headers directory contains exactly one `ecdsa.h`, the
Trezor header. Libsecp's root `ecdsa.h` remains private.

The literal brief command `nm -gU TLCore | sort | uniq -d` returns 18 rows on
the statically archived framework product, rather than an empty result. These
rows are archive-level repeated Swift compiler/profile records and include
address-qualified entries; this command is not a valid duplicate-definition
oracle for an archive. The authoritative per-architecture name-only scan
(`nm -gjU`, then duplicate names) reports zero duplicate names matching
`secp256k1_*` or `ecdsa_*`. The final main static link also succeeded without a
duplicate-symbol diagnostic. The exact 18-row artifact has SHA-256
`6c0b185cd0978175232cb4c64a1844fe1887fae93f7896b06af60d66276932a5`.

The public header artifact is
`/tmp/tlcore-stage-3-public-ecdsa-headers.txt`, SHA-256
`22579a98d2e39886c9b68e149b56ad5487def0fc62316f52bc809a1fc9848891`.

## Source-fidelity whitespace exception

`git diff --cached --check` reported 70 whitespace diagnostics in 14 imported
files: two diagnostics in byte-identical secp files and 68 in mapped pinned
Web3 files. Every warned Web3 line/EOF was found byte-for-byte in its pinned
source file. Custom compatibility files, tests, podspec, umbrella,
`TLWalletCore`, licenses, and the Example project had zero diagnostics.
Normalizing the pinned whitespace would reduce source fidelity, so the
inherited diagnostics are deliberately retained.

## Risks, rollback, and Task 4 entry gate

Remaining risks are limited to the intentionally preserved upstream Swift 4.2
surface and static-archive tooling ambiguity described above. Functional,
dependency, public-header, lint, full-suite, main-build, and main-test gates are
green.

Rollback the consumer first, then the provider:

```bash
git -C /Users/viccc/working/4_22_0/TronLink_iOS revert b87d495eaac377a9ade9f3b6baa2710696a76b85
git -C /Users/viccc/source/tronlink-iOS-core revert 7f20566c229c82c6abe731f0e941addcbcdaf17a
```

Task 4 may begin only from the two implementation commits above plus this
documentation commit, with no old Web3/secp imports or dependency names, the
61-test accepted Example suite green, the 8-test main gate green, and the
user-owned main lock/project changes still preserved outside Task 3 commits.
