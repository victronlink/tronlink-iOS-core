# Stage 0: Baseline and Provenance

## Repository State

The following pre-edit `git status --short` output was captured before dependency installation or builds:

```text
$ git -C /Users/viccc/source/tronlink-iOS-core status --short
 M .gitignore

$ git -C /Users/viccc/source/tronlink-iOS-core-others/tron-wallet-abi status --short

$ git -C /Users/viccc/source/tronlink-iOS-core-others/tron-wallet-keystore status --short

$ git -C /Users/viccc/source/tronlink-iOS-core-others/tron-wallet-web3swift status --short

$ git -C /Users/viccc/source/tronlink-iOS-core-others/tron-wallet-secp256k1 status --short

$ git -C /Users/viccc/working/4_22_0/TronLink_iOS status --short
 M Podfile.lock
 M TronLink.xcodeproj/project.pbxproj
?? .agents/
?? AGENTS.md
?? BASIC_MODE_API_INVENTORY.md
?? outputs/
?? reports/
?? scripts/assert_entropy_source.sh
```

The final main-app status was byte-for-byte unchanged for both tracked diffs. No main-app Podfile, lockfile, project file, or source file was edited by this stage.

## Source Commits and Tags

| Source | Commit | Tag |
| --- | --- | --- |
| `tronlink-iOS-core` | `eb25afce43edeaae7a3ba2bfaef2bd83e99756f5` | `1.0.7` |
| `tron-wallet-abi` | `c367023e0e141f414c9319c2ccb382eda396f2a4` | `1.0.2` |
| `tron-wallet-keystore` | `be7ad15ffa6fcb4c902bc19529f738a79576c881` | `1.0.5` |
| `tron-wallet-web3swift` | `2b24ba4e65a3cf2026d697396a95ba7e1937e325` | `1.1.2` |
| `tron-wallet-secp256k1` | `5d74ae264f59f5b98c5832f79cc33c2ed9bad82d` | `1.0.0` |
| `TronLink_iOS` consumer | `fdedb46aacf46e2336e50869950eea1369277379` | no tag observed |

The immutable baseline evidence commit is `2f15afc` (`test: capture TLCore migration baselines`).

## Source Manifests

| Manifest | Entries | Verification result |
| --- | ---: | --- |
| `tlcore-1.0.7.sha256` | 57 | exit 0 from `/Users/viccc/source/tronlink-iOS-core` |
| `tron-wallet-abi-1.0.2.sha256` | 124 | exit 0 from `/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-abi` |
| `tron-wallet-keystore-1.0.5.sha256` | 13 | exit 0 from `/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-keystore` |
| `tron-wallet-web3swift-1.1.2.sha256` | 117 | exit 0 from `/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-web3swift` |
| `tron-wallet-secp256k1-1.0.0.sha256` | 45 | exit 0 from `/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-secp256k1` |

Each manifest was checked with `shasum -a 256 -c` from its corresponding source root. The five manifest files contain 356 entries in total.

## Resolved Pod Versions

`pod install --verbose` in `Example` completed with exit 0. `Example/Podfile.lock` was written at `2026-08-25T13:04:01+0800` and is ignored.

The resolved migration pods were: `tronlink-iOS-core 1.0.7`, `TronWalletABI 1.0.2`, `TronWalletKeystore 1.0.5`, `TronWalletWeb3Swift 1.1.2`, and `tron-wallet-secp256k1 1.0.0`. Observed supporting pods included `BigInt 3.1.0`, `CryptoSwift 1.8.4`, `FMDB 2.7.5`, `PromiseKit 6.18.1`, `Protobuf 3.29.6`, `SipHash 1.2.2`, `SwiftProtobuf 1.38.1`, `gRPC 1.68.1`, `BoringSSL-GRPC 0.0.37`, `abseil 1.20240722.0`, and `scrypt.c 0.1.1`.

## TLCore Test Command and Result

The selected simulator was `iPhone 17 Pro` with UDID `017E8DDA-425E-420F-9644-82B896E4907C`.

The focused migration characterization run used:

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-0 CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/SingleModuleMigrationBaselineTests
```

It exited 0 at `2026-08-25T13:09:34+0800`: 4 tests, 0 failures, `** TEST SUCCEEDED **`.

The unfiltered suite used the same command without `-only-testing`; it exited 65 at `2026-08-25T13:11:40+0800`: 33 tests with 2 failures. The observed pre-existing failures were `Tests.testBase58CheckRoundTripWithFlickrAlphabet` at `Tests.swift:593` (`XCTAssertEqual failed: (\"nil\") is not equal to (\"Optional(8 bytes)\")`) and `Tests.testSignTransaction` at `Tests.swift:698` (`XCTAssertTrue failed`) after gRPC reported `SSL_ERROR_SSL: ... WRONG_VERSION_NUMBER`.

The deterministic comparison command skipped only those exact XCTest identifiers:

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-0 CODE_SIGNING_ALLOWED=NO -skip-testing:tronlink-iOS-core_Tests/Tests/testBase58CheckRoundTripWithFlickrAlphabet -skip-testing:tronlink-iOS-core_Tests/Tests/testSignTransaction
```

It exited 0 at `2026-08-25T13:13:12+0800`: 31 tests, 0 failures, `** TEST SUCCEEDED **`. The Flickr failure is the local baseline; the sign-transaction failure depends on the external gRPC response. Both remain unchanged and must reproduce until their owning scope changes.

## TronLink Build Command and Result

```bash
xcodebuild -workspace TronLink.xcworkspace -scheme TronLink -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tronlink-tlcore-stage-0 CODE_SIGNING_ALLOWED=NO build
```

Run from `/Users/viccc/working/4_22_0/TronLink_iOS`, the command exited 0 at `2026-08-25T13:14:09+0800` with `** BUILD SUCCEEDED **`. It emitted existing compiler and run-script warnings only.

## Golden Vectors

| Test | Observed shipping behavior and protected regression |
| --- | --- |
| `testLegacyABIVector` | ABI encoding of `BigUInt(42)` is 32 bytes ending in `2a`; catches integer ABI padding/encoding regressions. |
| `testLegacyWeb3SignAndRecoverVector` | Private key `00…01` signs 32 bytes of `11`, recovers its signer, and maps to `0x7e5f4552091a69125d5dfcb7b8c2659029395bdf`; catches signing, recovery, and address-derivation regressions. |
| `testLegacyTronDerivationVector` | The standard abandon mnemonic produces private key `b5a4cea271ff424d7c31dc12a3e43e401df7a40d7412a15750f3f0b6b5449a28`; `Wallet.getKey(at: 0)` exposes a 20-byte raw address, while public `KeystoreKey(password:mnemonic:)` exposes the corresponding 21-byte `0x41`-prefixed Tron address; catches derivation path and address-boundary regressions. |
| `testLegacyTronBase58CheckVector` | `T9yD14Nj9j7xAB4dbGeiX9h8unkKHxuWwb` decodes to `410000000000000000000000000000000000000000` and re-encodes unchanged; catches Base58Check prefix/checksum regressions. |

The originally planned mnemonic private-key literal `426e488d2a2edaa1b69e6f3a252b60e3d98b062092dc3126908b7efc0a3aa78e` was falsified by the pinned shipping implementation. The focused test also established that `HDKey.address` is 20 bytes rather than a Tron-prefixed 21 bytes. The passing test uses the observed values above. No production code was written or modified, so no TDD RED phase was appropriate. The test fixture property was named `messageHash`, rather than the planned `hash`, because XCTestCase inherits an Objective-C `hash` member.

## Existing Dirty Files Preserved

Before CocoaPods and Xcode, the main app had a 202-line `Podfile.lock` diff and a 36-line `TronLink.xcodeproj/project.pbxproj` diff. The lockfile changes switch the four source pods from `security-optimization` git branches to released pods (including ABI `1.0.3` to `1.0.2`) and change React-related checksums. The project-file diff relocates and adds the existing `TronLinkGasFreeTokenSupport.swift` references. These diffs were captured before the stage and compared after the build with:

```bash
git diff -- Podfile.lock | diff -u /tmp/task-0-preexisting-Podfile.lock.diff -
git diff -- TronLink.xcodeproj/project.pbxproj | diff -u /tmp/task-0-preexisting-project.pbxproj.diff -
```

Both comparisons exited 0. The preserved full snapshots are also recorded in the Task 0 execution report and were not staged in either Stage 0 commit.

## Risks and Rollback

The two unfiltered-suite failures are baseline facts, not migration regressions. The signing test also depends on an external gRPC endpoint. CocoaPods generated ignored `Example/Pods`, `Example/Podfile.lock`, and workspace artifacts. The main app had user-owned dirty files before the stage; they remain unchanged.

Rollback is `git revert 2f15afc` for immutable vectors/manifests and `git revert` of the documentation commit recorded below. No rollback action is required in the main app.

## Stage 1 Entry Gate

Met: all five manifests verify, all four migration baseline vectors pass, the comparison run passes 31 of 31 tests after excluding only the two recorded baseline failures, and the unmodified main-app baseline build exits 0. Later stages must retain this evidence and reproduce the two excluded baseline failures unless their owning scope changes.
