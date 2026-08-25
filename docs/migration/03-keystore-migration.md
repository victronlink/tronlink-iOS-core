# ABI and Keystore activation validation

## Scope and commits

Task 2 atomically activated the staged ABI implementation, embedded the pinned
TronWalletKeystore 1.0.5 implementation, and moved the main application away
from `TronCore` and `TronKeystore`.

| Repository | Before | Implementation commit |
| --- | --- | --- |
| `tronlink-iOS-core` (`tlcore`) | `7eba025b7dedf06ebf5403486b372fd3b82e7eef` | `85fd3aac0fa79b8459459248e7b165dedb4927a5` (`refactor: activate ABI and keystore in TLCore`) |
| Main app (`4.22.0_Development_tlcore`) | `fdedb46aacf46e2336e50869950eea1369277379` | `664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b` (`refactor: consume TRON wallet APIs from TLCore`) |

The core commit contains exactly 21 task files. The main commit contains exactly
47 consumer/test files: 41 production files and 6 test files. Neither main-app
`Podfile.lock` nor its project file was staged.

The durable chronological evidence, including every consequential command,
invalid/cancelled runs, exact exit states, terminated identifiers, artifact
availability, extraction commands, and result-bundle hashes, is in
[`evidence/task-2-command-ledger.md`](evidence/task-2-command-ledger.md).

## Source provenance and mapping

The Keystore source came from
`/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-keystore` at pinned
commit `be7ad15ffa6fcb4c902bc19529f738a79576c881`. All 13 source paths are present:

| Upstream path below `TronWalletKeystore/Classes` | TLCore destination below `tronlink-iOS-core/Classes/Keystore` |
| --- | --- |
| `Account.swift` | `Account.swift` |
| `KeyStore.Error.swift` | `KeyStore.Error.swift` |
| `KeyStore.swift` | `KeyStore.swift` |
| `KeystoreKey.swift` | `KeystoreKey.swift` |
| `KeystoreKeyHeader.swift` | `KeystoreKeyHeader.swift` |
| `Scrypt.swift` | `Scrypt.swift` |
| `ScryptParams.swift` | `ScryptParams.swift` |
| `Wallet/DerivationPath.Index.swift` | `Wallet/DerivationPath.Index.swift` |
| `Wallet/DerivationPath.swift` | `Wallet/DerivationPath.swift` |
| `Wallet/HDKey.swift` | `Wallet/HDKey.swift` |
| `Wallet/Mnemonic.swift` | `Wallet/Mnemonic.swift` |
| `Wallet/MnemonicWords.swift` | `Wallet/MnemonicWords.swift` |
| `Wallet/Wallet.swift` | `Wallet/Wallet.swift` |

The copied upstream license is
`ThirdPartyLicenses/TronWalletKeystore-LICENSE`. Source adaptations are limited
to removing obsolete `TronCore`/`TronKeystore` imports, removing four
`TronCore.` qualifiers so `Address` resolves inside TLCore, changing the single
`drop0x()` call to TLCore's existing `drop0x` property, and deleting the now
duplicate private String helper. A pinned-source verifier passed with exactly
that allowed delta.

The full staged whitespace check intentionally reported nine historical bytes:
seven trailing-space lines in `KeyStore.swift`, one in `KeystoreKey.swift`, and
one EOF blank line in `ScryptParams.swift`. Direct upstream diffs proved these
bytes are present in the pinned blobs. They were retained for fidelity. A
scoped cached check covering every other staged path exited 0.

## Module and build configuration

The target remains Swift 4.2. The prior ABI exclusion and
`TronWalletKeystore 1.0.5` dependency were removed in the same commit, so the
static graph never links both old and embedded implementations. The podspec now:

- selects only `h`, `m`, `c`, and `swift` source files;
- preserves Trezor `*.table` files and marks the 16 fragment/internal headers
  private;
- sets the Trezor, AES, ChaCha20-Poly1305, and Ed25519 header-search paths;
- uses ARC only for `gRPC/**/*.pbrpc.m` and
  `ABI/ObjectiveC/EthereumCrypto.m`, leaving generated protobuf Objective-C
  sources under MRC;
- adds direct `BigInt 3.1.0`, `CryptoSwift 1.8.4`, and
  `SwiftProtobuf 1.38.1` dependencies; and
- retains gRPC, Protobuf, FMDB, and `TronWalletWeb3Swift 1.1.2` for Task 3.

The exact hand-written module map is:

```modulemap
framework module TLCore {
  umbrella header "TLCore-umbrella.h"
  export *
}
```

The hand-written umbrella exports UIKit/Foundation, all required generated
protobuf/gRPC public headers, `EthereumCrypto.h`, and `TrezorCrypto.h`. It does
not recursively expose Trezor fragment headers. The generated Objective-C
protobuf class remains runtime class `Address`, while
`NS_SWIFT_NAME(TronProtoAddress)` imports it into Swift without colliding with
the embedded ABI `TLCore.Address`.

## Core test results

Tests ran on iPhone 17 Pro simulator
`017E8DDA-425E-420F-9644-82B896E4907C`, arm64, iOS 26.5.
The focused ABI/protobuf result bundle is:

`/tmp/tlcore-stage-2-green/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_15-08-25-+0800.xcresult`

All four tests passed:

| Test | Result |
| --- | --- |
| `EmbeddedABIGoldenTests/testEmbeddedAddressAndIntegerEncodingMatchGoldenValues` | Passed |
| `EmbeddedABIGoldenTests/testEmbeddedFunctionDynamicValuesAndRLPMatchGoldenValues` | Passed |
| `EmbeddedABIGoldenTests/testEmbeddedTronDerivationAndBase58MatchGoldenValues` | Passed |
| `EmbeddedABIGoldenTests/testProtobufAddressRetainsBehaviorUnderSwiftRename` | Passed |

The focused Keystore result bundle is:

`/tmp/tlcore-stage-2-green/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_15-09-21-+0800.xcresult`

It executed all 27 embedded tests: 25 passed and the two pinned-test
contradictions described below failed.

| Test | Result |
| --- | --- |
| `testAES128CBCKeystoreRemainsDecryptable` | Passed |
| `testConcurrentImportStoresOneAccount` | Passed |
| `testDeleteRequiresCorrectPassword` | Passed |
| `testDifferentPassphrasesDeriveDifferentKeys` | Passed |
| `testEncodedAddressRemainsLowercaseAndPrefixFree` | Passed |
| `testExportedMnemonicExcludesPassphrase` | Passed |
| `testGeneratedHDWalletReturnsMnemonicWithoutRetainingIt` | Passed |
| `testHDKeystoreJSONRoundTripPreservesAddress` | Passed |
| `testHDObjectsDoNotRetainMnemonicOrPassphrase` | Passed |
| `testHostileKDFParamsDecodeButFailToDecrypt` | Passed |
| `testImportRejectsAddressMismatch` | Passed |
| `testInvalidPrivateKeyThrowsInsteadOfTrapping` | Passed |
| `testKeystoreKeyAcceptsValid32BytePrivateKey` | Passed |
| `testKeystoreKeyRejectsAllPrintableASCIIInput` | Expected pinned-test contradiction |
| `testKeystoreKeyRejectsMnemonicASCIIPayload` | Expected pinned-test contradiction |
| `testKeyWithoutPassphraseSurvivesReload` | Passed |
| `testLegacyLightPresetStillDerivesUnderNewBounds` | Passed |
| `testLegacyOversizedEncryptedKeyUsesFirst32BytesAcrossKeyStoreAPIs` | Passed |
| `testOverlongMultiBytePassphraseThrows` | Passed |
| `testPassphraseAtByteLimitDerives` | Passed |
| `testPassphraseSurvivesReload` | Passed |
| `testUpdatePasswordPreservesCustomDerivationPathAndRejectsAddressChange` | Passed |
| `testUppercaseCryptoJSONRemainsDecodable` | Passed |
| `testValidateAcceptsShippedAndStandardPresets` | Passed |
| `testValidateMatchesAndroidBoundsForRPAndDklen` | Passed |
| `testValidateRejectsEmptySalt` | Passed |
| `testValidateRejectsHostileParametersWithoutTrapping` | Passed |

The complete core suite executed 60 tests: 56 passed and exactly four known
tests failed. In addition to the two Keystore contradictions, the failures were
the Stage 0 baselines `Tests/testBase58CheckRoundTripWithFlickrAlphabet` and
`Tests/testSignTransaction`. The result bundle is:

`/tmp/tlcore-stage-2-tests/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_15-24-09-+0800.xcresult`

That bundle is no longer present because later runs reused its DerivedData
directory. The command, exit 65, counts, and exact four identifiers survive in
the tracked command ledger.

A fully serial comparison skipping exactly those four tests passed 56/56 with
exit 0 in 892.601 seconds. Its result bundle is:

`/tmp/tlcore-stage-2-tests/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_16-06-25-+0800.xcresult`

The printable-ASCII failures are retained tests from upstream history, but the
pinned 1.0.5 implementation accepts a valid 32-byte printable private-key
scalar. Restoring the historical heuristic would change shipped behavior and
would fail the main compatibility test
`testKeystoreKeyInitAcceptsPrintableASCII32BytePrivateKey`, which passed.

JSON compatibility is covered explicitly: uppercase `Crypto` decodes,
AES-128-CBC decrypts, HD export/import preserves address and mnemonic/private
key, and encoded addresses remain lowercase without `0x`. The pinned legacy
oversized encrypted-key behavior also passed: the first 32 bytes remain usable
across KeyStore APIs.

## Lint and static-link gates

`ruby -c tronlink-iOS-core.podspec` passed. CocoaPods lint passed with warnings:

```bash
XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig \
  pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests
```

The temporary xcconfig sets `IPHONEOS_DEPLOYMENT_TARGET = 13.0`. It is required
only because Xcode 26 no longer provides `libarclite` for the transitive
historical `scrypt.c` target's iOS 8 declaration; plain lint otherwise reaches
that environment failure.

The generated Example project removed the two legacy framework references.
The main app then completed a generic Debug simulator static build with exit 0
at `/tmp/tronlink-tlcore-stage-2`; there were no duplicate symbols. Generated
dependency graphs contain neither `TronWalletABI` nor `TronWalletKeystore`.

## Main-app migration and tests

The mechanical migration replaces old module imports and qualifications with
TLCore. The final source scan across `TronLink` and `TronLinkTests` returned
zero matches for imports or qualifications of `TronCore`/`TronKeystore`.

The initial selected test compile exposed 17 ambiguous `Data` extension uses
because two test files imported all of TLCore alongside app extensions. The
fix was limited to removing an unused old Core import from
`TronLink_EIP721Test.swift` and selectively importing `HDKey`, `KeystoreKey`,
`Mnemonic`, and `Wallet` in `TronLinkTest_walletCore.swift`. The failed compile
evidence is:

`/tmp/tronlink-tlcore-stage-2-tests/Logs/Test/Test-TronLink-2026.08.25_15-35-45-+0800.xcresult`

That bundle is no longer present because later runs reused its DerivedData
directory. The ledger retains all 17 compile diagnostics and the exact
two-file import correction.

The final comparison excluded only the ruled stale oversized-key assertion and
passed 38/38 with no skips or retries in 464.532 seconds. Its result bundle is:

`/tmp/tronlink-tlcore-stage-2-tests/Logs/Test/Test-TronLink-2026.08.25_16-56-19-+0800.xcresult`

Executed identifiers and results:

| Test group | Passed identifiers |
| --- | --- |
| `TronLinkTest_importCreate` | `testAssociatedWalletBatchRunnerSurvivesAsyncStepWithoutExternalRetention`, `testAssociatedWalletBatchStopsAtFirstFailureAndKeepsEarlierSuccessCount`, `testAssociatedWalletLinkBuilderUsesExplicitSourceWallet`, `testAssociatedWalletPersistenceRollsBackKeystoreOnWalletFailure`, `testAssociatedWalletPersistenceRollsBackOnlyCurrentItemOnLinkFailure`, `testBatchMnemonicImportCallsRealImporterOnce`, `testBatchMnemonicImportInvalidPhraseCompletesOnce`, `testClassifyImportAcceptsSelfConsistentKeystore`, `testClassifyImportRejectsDeclaredAddressMismatch`, `testClassifyImportRejectsKeystoreWhoseAddressCannotBeDerived`, `testExample`, `testHDKeystoreImportJSONPreservesAddressAfterP0_01Fix`, `testHDRootWalletResolverFailsClosedWhenNoGroupMemberIsPresent`, `testHDRootWalletResolverPrefersCurrentMemberWhenRootIsNotImported`, `testHDRootWalletResolverPrefersTheIndexZeroRootWallet`, `testHDRootWalletResolverRejectsEmptyRootAddress`, `testHDRootWalletResolverSkipsGroupMembersMissingFromTheWalletTable`, `testKeystoreKDFGateSerializesConcurrentWork`, `testKeystoreKeyInitAcceptsPrintableASCII32BytePrivateKey`, `testKeystoreKeyInitAcceptsValid32BytePrivateKey`, `testKeystoreKeyInitRejectsMnemonicASCIIPayload`, `testLegacyHDKeystoreImportedAsPrivateKeyKeepsAddressAndBecomesUsable`, `testLegacyHDKeystoreRiskDetectorRejectsOtherImportAndAccountTypes`, `testLegacyHDKeystoreRiskDetectorRequiresOverlengthEncryptedKeystorePayload`, `testPerformanceExample`, `testPrivateKeyKeystoreValidationAccepts32ByteCiphertext`, `testPrivateKeyKeystoreValidationRejectsInvalidCiphertextLengths`, `testPrivateKeyKeystoreValidationRejectsUnsupportedVersion` |
| `TronLinkTest_scryptParams` | `testBalancedPresetValidates`, `testDefaultScryptParamsUseBalancedPreset`, `testFreshKeystoreOnDiskUsesBalancedPreset`, `testLegacyLightPresetKeystoreStillDecrypts`, `testScryptDeterministicForBalancedPreset` |
| `TronLinkTest_walletCore` | `testCreateHDWalletMnemonic`, `testCreateHDWalletPrivateKey`, `testCreateHDWalletPublicKey`, `testUpdateWalletReportsFailureWhenKeystoreAccountIsMissing`, `testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks` |

Two earlier selected runs terminated individual scrypt-heavy tests with signal
TERM/KILL under concurrent simulator memory pressure. Each terminated test
passed when isolated, and the final serial comparison above passed all 38.

`TronLinkTest_importCreate/testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation`
is the one deliberately excluded stale assertion. It conflicts with pinned
commit `f341275`, which intentionally truncates oversized historical encrypted
private-key payloads to 32 bytes, and with the passing upstream compatibility
test. The migration preserves the published behavior rather than changing
legacy-wallet compatibility.

## Dependency and dirty-file preservation

The task-time resolved lock removes `TronWalletABI` and
`TronWalletKeystore`, makes `BigInt`, `CryptoSwift`, and `SwiftProtobuf` direct
TLCore dependencies, and retains Web3. The working lock scan contains no old
split modules. It was not committed because it began with broad user-owned
source/checksum changes that cannot be separated into a coherent index blob.

| File | Initial SHA-256 | Final working SHA-256 | Commit disposition |
| --- | --- | --- | --- |
| `Podfile` | `c6a2a56b4832430865f116e86e849f6fb38ab2f607d82d4aa0b3c222fc348ded` | same | Restored; not dirty |
| `Podfile.lock` | `0544e38266de756e89efa1df3bbb8ffa9978aa82795707b794e973fbe9bbac56` | `df93f25f0130d3edf33f7c53bcde28fd757c17ce70c261fca911bad912e7e54d` | Preserved working diff; unstaged |
| `TronLink.xcodeproj/project.pbxproj` | `17cc8f86b7fa92be3ce92cbac6cb64bcbf7de363921cf6d52f2e1d254d5196d7` | same | Pre-existing working diff preserved; unstaged |

Relative to repository HEAD, the preserved lock is 51 insertions and 82
deletions. Its relevant graph delta removes the old source/check-out/checksum
entries for ABI and Keystore and adds the three direct dependencies; the same
working diff also contains pre-existing React checksum and source-resolution
changes. Untracked user artifacts were untouched.

## Risks, rollback, and Task 3 gate

Known risks are limited and explicit:

- two upstream printable-ASCII rejection tests contradict the pinned runtime;
- one main-app crafted oversized-key assertion contradicts required legacy
  compatibility;
- Xcode 26 lint requires the deployment-target xcconfig for an old transitive C
  dependency; and
- the main branch relies on the preserved resolved lock or a fresh resolution
  until the user decides how to commit lockfile changes.

To roll back Task 2, first list the documentation commits newer than the core
implementation and revert every displayed commit in newest-to-oldest order:

```bash
git -C /Users/viccc/source/tronlink-iOS-core log --format='%H %s' \
  85fd3aac0fa79b8459459248e7b165dedb4927a5..HEAD -- \
  docs/migration/03-keystore-migration.md \
  docs/migration/evidence/task-2-command-ledger.md
```

Then revert the two implementation commits in consumer-first order:

```bash
git -C /Users/viccc/working/4_22_0/TronLink_iOS revert 664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b
git -C /Users/viccc/source/tronlink-iOS-core revert 85fd3aac0fa79b8459459248e7b165dedb4927a5
```

Task 3 may begin once it treats the current TLCore module as the only ABI and
Keystore provider, keeps the protobuf Swift rename, does not reintroduce the
old split pods, and preserves the ruled compatibility behavior. Its remaining
planned boundary is `TronWalletWeb3Swift 1.1.2` and the existing
`Web3.Utils.hashECRecover` call.
