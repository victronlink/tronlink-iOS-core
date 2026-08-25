# Task 2 command and artifact ledger

This is the durable execution record for Task 2 (ABI and Keystore activation).
Times are Singapore time (`+0800`) on 2026-08-25. Commands, working directories,
exit states, test counts, terminated identifiers, and artifact retention are
preserved here because `/tmp` result bundles are not repository artifacts.

The command history was reconstructed from the task session transcript. Where
the terminal result or literal command was not captured, this ledger says so
instead of inferring it. No `.xcresult`, DerivedData, log, or temporary xcconfig
is committed.

## Repositories and fixed environment

| Item | Value |
| --- | --- |
| Core repository | `/Users/viccc/source/tronlink-iOS-core` |
| Core Example working directory | `/Users/viccc/source/tronlink-iOS-core/Example` |
| Main repository | `/Users/viccc/working/4_22_0/TronLink_iOS` |
| Pinned Keystore repository | `/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-keystore` |
| Keystore pin | `be7ad15ffa6fcb4c902bc19529f738a79576c881` |
| Simulator destination | `platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C` |
| Simulator runtime | iPhone 17 Pro, arm64, iOS 26.5 |
| Test signing override | `CODE_SIGNING_ALLOWED=NO` |
| Lint-only xcconfig | `/tmp/tlcore-lint-xcode26.xcconfig`, containing `IPHONEOS_DEPLOYMENT_TARGET = 13.0` |

## Literal command catalog

The chronological tables below refer to these exact shell commands. Repeated
commands retain separate rows because their outcomes and artifacts differ.

### Core RED and test commands

`C-RED-TEE`:

```bash
set -o pipefail
printf 'started=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')"
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-red CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedABIGoldenTests 2>&1 | tee /tmp/tlcore-stage-2-red.log
TLCORE_RED_STATUS=${pipestatus[1]}
printf 'finished=%s exit=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')" "$TLCORE_RED_STATUS"
exit "$TLCORE_RED_STATUS"
```

`C-RED` is the same `xcodebuild test` invocation without the `tee` wrapper.
`C-RED-FRESH` changes only the DerivedData path to
`/tmp/tlcore-stage-2-red-valid`. `C-RED-FRESH-QUIET` additionally places
`-quiet` after `test`:

```bash
xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-red-valid CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedABIGoldenTests
```

Post-activation dependency generation:

```bash
pod install --verbose
```

Focused combined, ABI, and Keystore commands:

```bash
xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-green CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests -only-testing:tronlink-iOS-core_Tests/EmbeddedABIGoldenTests

xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-green CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedABIGoldenTests

xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-green CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests
```

Full, four-skip comparison, and isolated core commands:

```bash
xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO

xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO -skip-testing:tronlink-iOS-core_Tests/Tests/testBase58CheckRoundTripWithFlickrAlphabet -skip-testing:tronlink-iOS-core_Tests/Tests/testSignTransaction -skip-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests/testKeystoreKeyRejectsMnemonicASCIIPayload -skip-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests/testKeystoreKeyRejectsAllPrintableASCIIInput

xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests/testHDObjectsDoNotRetainMnemonicOrPassphrase
```

### Main-app commands

Dependency and build commands:

```bash
pod install --verbose

xcodebuild -workspace TronLink.xcworkspace -scheme TronLink -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tronlink-tlcore-stage-2 CODE_SIGNING_ALLOWED=NO build -quiet
```

`M-SELECTED`:

```bash
xcodebuild test -quiet -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTest_importCreate -only-testing:TronLinkTests/TronLinkTest_scryptParams -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPrivateKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPublicKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletMnemonic -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsFailureWhenKeystoreAccountIsMissing -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks
```

The first post-fix run used this literal variant, which accidentally selected
the nonexistent `testCreateHDWalletMnemonicMissing` and omitted the wallet
failure method:

```bash
xcodebuild test -quiet -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTest_importCreate -only-testing:TronLinkTests/TronLinkTest_scryptParams -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPrivateKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPublicKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletMnemonic -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletMnemonicMissing -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks
```

`M-ISOLATED-PAIR`:

```bash
xcodebuild test -quiet -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTest_importCreate/testClassifyImportRejectsDeclaredAddressMismatch -only-testing:TronLinkTests/TronLinkTest_scryptParams/testScryptDeterministicForBalancedPreset
```

The 37/38 comparison was `M-SELECTED` plus:

```text
-skip-testing:TronLinkTests/TronLinkTest_importCreate/testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation
```

The isolated wallet command was:

```bash
xcodebuild test -quiet -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-2-tests CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks
```

The final comparison was the 37/38 command with
`-retry-tests-on-failure` immediately after `-quiet`. The result contains
exactly 38 test-case nodes, so the retry facility did not execute a retry.

### Lint and static commands

```bash
ruby -c tronlink-iOS-core.podspec

pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests

env IPHONEOS_DEPLOYMENT_TARGET=13.0 pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests

env XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests
```

Representative final residue scans and preservation checks were:

```bash
test -z "$(rg -l '^[[:space:]]*(@testable[[:space:]]+)?import[[:space:]]+((class|struct|enum|protocol|func|var|let|typealias)[[:space:]]+)?(TronCore|TronKeystore)([.]|[[:space:]]|$)|\b(TronCore|TronKeystore)\.' TronLink TronLinkTests || true)"

test -z "$(rg -l 'TronWalletABI|TronWalletKeystore' Podfile.lock Pods/Manifest.lock || true)"

shasum -a 256 Podfile Podfile.lock TronLink.xcodeproj/project.pbxproj

git diff --cached --check
```

The literal main preservation snapshot command was:

```bash
cp Podfile /tmp/task2-main-Podfile.pre
cp Podfile.lock /tmp/task2-main-Podfile.lock.pre
cp TronLink.xcodeproj/project.pbxproj /tmp/task2-main-project.pbxproj.pre
shasum -a 256 Podfile Podfile.lock TronLink.xcodeproj/project.pbxproj
```

## Chronological execution

### TDD RED phase

All rows used the core Example working directory and the fixed simulator/signing
environment above.

| Time | Command | Exit/count | Disposition and artifact |
| --- | --- | --- | --- |
| 14:54:42 | `C-RED-TEE` in the restricted sandbox | 66 | Rejected infrastructure result: CoreSimulator was unavailable and xcodebuild reported `'tronlink-iOS-core.xcworkspace' is not a workspace file`. `/tmp/tlcore-stage-2-red.log` was temporary and is not retained in Git. |
| 14:55:01 | `C-RED-TEE`, approved simulator access | 65 | Rejected RED: copied test insertion produced unrelated `Cannot find 'final' in scope`, consecutive-statement, top-level-expression, and unary-separation syntax errors. Bundle `14-55-06` survives in `/tmp/tlcore-stage-2-red/Logs/Test`. |
| 14:55:18 | `C-RED`, same DerivedData path | Terminal exit not captured | Overlapped/superseded the previous run; bundle `14-55-19` survives but has no usable Swift issue summary. Rejected. |
| 14:55:31 | `C-RED-FRESH` | 65 | Fresh bundle `14-55-33` survives; still rejected for the same unrelated `}final` syntax splice. |
| 14:58:24 | `C-RED-FRESH-QUIET` after repairing the splice | 65 | Bundle `14-58-24` survives. Planned missing surface was present, but the run also contained internal-access and Swift diagnostic-production failures in copied tests. Rejected. |
| 14:58:43 | `C-RED-FRESH-QUIET` after the access repair | 65 | Bundle `14-58-43` survives. Planned missing surface remained, but an unrelated Swift 4 AES slice type-inference diagnostic remained. Rejected. |
| 14:59:11–14:59:12 | `C-RED-FRESH-QUIET` after making the AES expression Swift-4-compatible | 65; compilation stopped before test discovery | **Accepted RED.** Bundle `14-59-12` survives. Its 14 Swift compiler errors are confined to the planned absent ABI/Keystore/protobuf surface and contextual cascades listed below. |

The accepted RED bundle reports these unique diagnostics verbatim:

```text
Call can throw but is not marked with 'try'
Cannot find 'TronProtoAddress' in scope
Cannot infer contextual base in reference to member 'uint'
Module 'TLCore' has no member named 'ABIEncoder'
Module 'TLCore' has no member named 'Function'
Module 'TLCore' has no member named 'KeystoreKey'
Module 'TLCore' has no member named 'RLP'
Module 'TLCore' has no member named 'Wallet'
Reference to member 'address' cannot be resolved without a contextual type
Reference to member 'bool' cannot be resolved without a contextual type
Reference to member 'string' cannot be resolved without a contextual type
Type 'Any' has no member 'dynamicArray'
Type 'Any' has no member 'uint'
Value of type 'Address' has no member 'eip55String'
```

The throwing-initializer diagnostic is also expected pre-activation evidence:
without embedded ABI `Address`, `TLCore.Address` resolved to the generated
protobuf type. No unrelated production compile failure appears in this accepted
bundle.

### Core activation and validation

| Time | Command | Exit/count | Disposition and artifact |
| --- | --- | --- | --- |
| 15:01:30 | Original pinned-source transform verifier | 1 | Rejected verifier implementation: its transform reported `mismatch KeystoreKey.swift` even though the license comparison passed. The later direct diff and corrected 13/13 verifier established that the verifier transform, not the embedded source, was wrong. |
| 15:01:55–15:02:01 | `pod install --verbose` | 0 | Generated Example graph removed `TronWalletABI` and `TronWalletKeystore`; 18 pods installed. Generated files were not committed. |
| 15:02:07–15:08:10 | Focused combined ABI + Keystore command | Cancelled with `Ctrl-C`; counts unavailable | Build completed, but the runner did not produce a summary before cancellation. Bundle `15-02-08` survives but is not acceptance evidence. |
| 15:08:24; bundle 15:08:25 | Focused ABI command | 0; 4/4 passed | Accepted focused ABI/protobuf evidence; bundle survives. |
| 15:09:21–15:22:08 | Focused Keystore command | 65; 25/27 passed | Exact failures: `EmbeddedKeystoreTests/testKeystoreKeyRejectsAllPrintableASCIIInput` and `EmbeddedKeystoreTests/testKeystoreKeyRejectsMnemonicASCIIPayload`. Both are ruled pinned-test contradictions. Bundle survives. |
| 15:17:09 | `ruby -c tronlink-iOS-core.podspec` plus diff/comment checks | 0; `Syntax OK` | Static check passed. |
| 15:24:08; bundle 15:24:09 | Full core command | 65; 56/60 passed | Exact failures were the two ruled Keystore tests plus Stage 0 baselines `Tests/testBase58CheckRoundTripWithFlickrAlphabet` and `Tests/testSignTransaction`. Bundle later became unavailable because the same DerivedData path was reused. |
| 15:47:38; bundle 15:47:41 | Four-skip comparison | 65; 55/56 passed | `EmbeddedKeystoreTests/testHDObjectsDoNotRetainMnemonicOrPassphrase` terminated with signal `TERM` while concurrent main/lint work was active. Bundle later became unavailable. |
| 16:00 (recorded command second) | `ruby -c tronlink-iOS-core.podspec` | 0; `Syntax OK` | Second syntax gate passed. |
| 16:05:20; bundle 16:05:21 | Isolated core command | 0; 1/1 passed in 43.3 s | The formerly terminated identifier passed. Partial bundle survives; transcript output is authoritative because the current summary extractor returns no test summary for this partial result. |
| 16:06:25 | Four-skip comparison, run exclusively/serially | 0; 56/56 passed, 0 failed, 0 skipped, 892.601 s | Accepted final core comparison. Bundle `16-06-25` survives. |
| 17:26:44 | Review-round transform verifier, first spelling | Nonzero; shell bookkeeping error | It found only a trailing-newline mismatch in `KeystoreKey.swift`, then attempted to assign zsh's read-only `status` variable. Recorded as a verifier-command error, not a source failure. |
| 17:26:57 | Corrected review-round transform verifier | 0; `pinned_source_matches=13/13` | Compared all 13 files byte-for-byte after only the documented transforms. No implementation changed. |

The successful review-round verifier was:

```bash
pin=be7ad15ffa6fcb4c902bc19529f738a79576c881
upstream=/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-keystore
passed=0
for f in Account.swift KeyStore.Error.swift KeyStore.swift KeystoreKey.swift KeystoreKeyHeader.swift Scrypt.swift ScryptParams.swift Wallet/DerivationPath.Index.swift Wallet/DerivationPath.swift Wallet/HDKey.swift Wallet/Mnemonic.swift Wallet/MnemonicWords.swift Wallet/Wallet.swift; do
  if cmp -s <(git -C "$upstream" show "$pin:TronWalletKeystore/Classes/$f" | perl -0pe 's/^import TronCore\n//mg; s/TronCore\.//g; s/\.drop0x\(\)/.drop0x/g; s/\nprivate extension String \{\n    func drop0x\(\) -> String \{\n        if hasPrefix\("0x"\) \{\n            return String\(dropFirst\(2\)\)\n        \}\n        return self\n    \}\n\}\n$//s') "tronlink-iOS-core/Classes/Keystore/$f"; then
    passed=$((passed+1))
  else
    echo "MISMATCH $f"
  fi
done
test "$passed" -eq 13
rc=$?
echo "pinned_source_matches=$passed/13 exit=$rc"
exit "$rc"
```

### Main dependency graph, build, and selected tests

All test rows used the main repository working directory and the same simulator
and signing override. The build used the generic simulator destination shown in
the command catalog.

| Time | Command | Exit/count | Disposition and artifact |
| --- | --- | --- | --- |
| 15:24:52 | `cp` preservation snapshots followed by `shasum -a 256 Podfile Podfile.lock TronLink.xcodeproj/project.pbxproj` | 0 | Captured pre-install hashes. Snapshot files were temporary and are not committed. |
| 15:25:59 | `pod install --verbose` | 0 | 84 dependencies/117 pods; local TLCore selected and split ABI/Keystore pods removed. |
| 15:26:24 | Old-pod scans, hashes, and `git status` | 0 | Zero `TronWalletABI`/`TronWalletKeystore` matches in `Podfile.lock` and `Pods/Manifest.lock`; preserved-file hashes recorded below. |
| 15:26:34 | Main static simulator build | 0; `BUILD SUCCEEDED` | No duplicate-symbol diagnostic. Together with the generated graph's absence of both split pods, this is the duplicate-link gate. No separate `nm` command was invoked. |
| 15:27:47 | Module-map and generated `Pods.xcodeproj` scans | 0 | Confirmed TLCore graph and absence of the two legacy framework references. |
| 15:35:44; bundle 15:35:45 | `M-SELECTED` | 65; compile failed before tests | 17 errors: one `Ambiguous use of 'addressString'` in `TronLinkTest_walletCore.swift:183` and 16 `Ambiguous use of 'hexEncoded'` diagnostics in `TronLink_EIP721Test.swift` at lines 40, 53, 82, 99, 121, 149, 153, 184, 188, 192, 292, 302, 312, 364, 368, and 372. Bundle later became unavailable. |
| After 15:35 | Source fix | Not a command result | Removed the unused broad Core import from `TronLink_EIP721Test.swift`; changed the wallet test to selective `TLCore.HDKey`, `TLCore.KeystoreKey`, `TLCore.Mnemonic`, and `TLCore.Wallet` imports. No production logic changed for this diagnostic. |
| 15:44:58; bundle 15:44:59 | Accidental selected-command variant shown above | 65; 35/38 passed | Failed stale assertion: `TronLinkTest_importCreate/testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation`. Terminated: `.../testClassifyImportRejectsDeclaredAddressMismatch` and `TronLinkTest_scryptParams/testScryptDeterministicForBalancedPreset`. One was `TERM` and one `KILL`; the transcript did not preserve which signal mapped to which identifier. Bundle later became unavailable. |
| 16:37:52 | `M-ISOLATED-PAIR` | 65; no tests | React codegen `build/generated/ios` state was missing/stale, so compilation stopped before test execution. No test conclusion. |
| 16:39:39 | `pod install` | 1 | Could not remove the generated directory. |
| 16:39:45 | `ls -ld build build/generated build/generated/ios` and `find` inspection | 0 | Confirmed generated-path state. |
| 16:39:48 | `rm -rf build/generated/ios` | 0 | Removed only the generated React codegen directory so CocoaPods could recreate it; recoverable through pod generation. |
| 16:39:55 | `pod install` | 1 | Sandbox could not recreate `build/generated/ios`. |
| 16:39:58 | Path inspection and ordinary `mkdir -p build/generated/ios` | 1 | Confirmed sandbox permission was the blocker. |
| 16:40:03 | Approved `mkdir -p build/generated/ios` | 0 | Recreated only the generated directory. |
| 16:40:11 | Approved `pod install` | 0 | Regenerated React codegen and synchronized pods. |
| 16:41:02; bundle 16:41:03 | `M-ISOLATED-PAIR` | 0; 2/2 passed | Both formerly terminated identifiers passed. Bundle later became unavailable because the DerivedData path was reused. |
| 16:45:05 | `M-SELECTED` plus stale-test skip | 65; 37/38 passed | `TronLinkTest_walletCore/testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks` terminated with signal `KILL`. Bundle later became unavailable. |
| 16:54:30; bundle 16:54:31 | Isolated wallet command | 0; 1/1 passed in 80.255 s | Formerly killed identifier passed. Bundle survives. |
| 16:56:18 | Final selected command with stale-test skip and `-retry-tests-on-failure` | 0; 38/38 passed, 0 failed, 0 skipped, 464.532 s | Accepted final main comparison. Exactly 38 result nodes prove no retry occurred. Bundle `16-56-19` survives. |

The broad-import correction is evidenced by the initial compile diagnostics and
the succeeding compile/test runs. The exact two-file patch remains in main
commit `664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b`.

### Lint, scans, fidelity, staging, and commits

| Time | Command | Exit/result | Disposition |
| --- | --- | --- | --- |
| 15:49:38–15:50:42 | Plain `pod lib lint ...` in restricted network sandbox | 1 | CDN DNS resolution failure; infrastructure-only, not a source result. |
| 15:50:50–15:57:17 | Approved plain `pod lib lint ...` | 1 after explicit `Ctrl-C`; `[!] Cancelled` | Cancelled because it overlapped long simulator work. Not an acceptance result. |
| 16:22:19–16:30:24 | Approved plain `pod lib lint ...` | 1 | Sole blocker was Xcode 26 missing `libarclite` for transitive `scrypt.c` target declaring iOS 8. |
| 16:30:44–16:31:24 | `env IPHONEOS_DEPLOYMENT_TARGET=13.0 pod lib lint ...` | 1 | Same `libarclite` failure; environment variable did not override generated pod targets. |
| 16:31:38 | Created `/tmp/tlcore-lint-xcode26.xcconfig` with `IPHONEOS_DEPLOYMENT_TARGET = 13.0` | 0 | Temporary environment workaround only; not committed. |
| 16:31:42–16:37:42 | `env XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig pod lib lint ...` | 0 | `tronlink-iOS-core passed validation`; warnings allowed. |
| 15:45, 15:52, 17:04 | Source/module/dependency residue scans | 0 | Final scan: 47 main consumer files and zero old-module matches. Generated locks/manifests also had zero split-pod matches. |
| 17:06:35–17:06:40 | Pinned blob display and direct `diff -u` checks | Expected deltas only | Verified the three whitespace-bearing files against pinned blobs and the documented import/qualifier/`drop0x`/helper adaptations. |
| 17:05:03 | Core task-file staging and full `git diff --cached --check` | Nonzero; exactly 9 diagnostics | Seven trailing-space lines in `KeyStore.swift`, one in `KeystoreKey.swift`, and one EOF blank line in `ScryptParams.swift`. Direct upstream comparison proved all nine bytes came from pinned source. |
| 17:06:46 | Scoped cached check excluding those exact three pinned files | 0 | Every other staged path passed. |
| 17:06:50 | `git commit -m "refactor: activate ABI and keystore in TLCore"` | 0 | Core implementation commit `85fd3aac0fa79b8459459248e7b165dedb4927a5`, exactly 21 task files. The literal preceding `git add` path list is recoverable from the commit tree but was not retained as a standalone transcript field. |
| 17:07:00 | Main `git add` under default sandbox | 1 | `.git/index.lock` permission denial; no partial staging conclusion used. |
| 17:07:04 | Approved main `git add` of the explicit 47-file list | 0 | User-owned `Podfile.lock` and project file stayed unstaged. The literal 47-path argument list was not retained as a standalone transcript field; `git diff-tree --no-commit-id --name-only -r 664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b` is the authoritative path set. |
| 17:07:09 | Main cached count/check | 0 | Exactly 47 staged files; `git diff --cached --check` passed. |
| 17:07:13 | `git commit -m "refactor: consume TRON wallet APIs from TLCore"` | 0 | Main commit `664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b`. |
| 17:07:17 | Final main hashes/residue/status | 0 | Confirmed zero old imports/qualifiers and preserved dirty-file hashes. |
| 17:09:29 | Core documentation `git add` under default sandbox | 1 | `.git/index.lock` permission denial. |
| 17:09:32–17:09:38 | Approved add/check/commit | 0 | Documentation commit `c350af06a48e6cb752954f1644e269d32395c677`, message `docs: record keystore migration validation`. |
| 17:09:59–17:10:05 | Core/main post-commit assertions | 0 | Core clean at expected parent chain; main commit has 47 files, empty index, zero old-module/split-pod matches. |

The pre/post main preservation hashes were:

| File | Initial SHA-256 | Final SHA-256 | Disposition |
| --- | --- | --- | --- |
| `Podfile` | `c6a2a56b4832430865f116e86e849f6fb38ab2f607d82d4aa0b3c222fc348ded` | same | Restored; clean |
| `Podfile.lock` | `0544e38266de756e89efa1df3bbb8ffa9978aa82795707b794e973fbe9bbac56` | `df93f25f0130d3edf33f7c53bcde28fd757c17ce70c261fca911bad912e7e54d` | User-owned working diff preserved and unstaged |
| `TronLink.xcodeproj/project.pbxproj` | `17cc8f86b7fa92be3ce92cbac6cb64bcbf7de363921cf6d52f2e1d254d5196d7` | same | Pre-existing working diff preserved and unstaged |

## Artifact inventory and durable extraction

Availability was checked after Task 2. A path marked unavailable was overwritten
or removed by later reuse of the same DerivedData directory; its transcript
result is retained above.

| Result | Bundle timestamp/path suffix | Current status | Durable result |
| --- | --- | --- | --- |
| Invalid RED | core red `14-55-06` | Survives | Rejected unrelated syntax failure |
| Superseded RED | core red `14-55-19` | Survives | Terminal exit not captured; unusable issue summary |
| Invalid RED | core red-valid `14-55-33` | Survives | Rejected unrelated syntax failure |
| Invalid RED | core red-valid `14-58-24` | Survives | Rejected internal-access/diagnostic failure |
| Invalid RED | core red-valid `14-58-43` | Survives | Rejected Swift 4 inference failure |
| Accepted RED | core red-valid `14-59-12` | **Survives** | Exit 65, compilation stopped, 14 planned-surface diagnostics |
| Combined focused | core green `15-02-08` | Survives | Cancelled; no acceptance summary |
| Focused ABI | core green `15-08-25` | Survives | 4/4 passed |
| Focused Keystore | core green `15-09-21` | Survives | 25/27 passed; exact two ruled failures |
| Full core | core tests `15-24-09` | **Unavailable** | 56/60; exact four failures retained in transcript/ledger |
| First four-skip | core tests `15-47-41` | **Unavailable** | 55/56; exact TERM identifier retained |
| Isolated core | core tests `16-05-21` | Survives, partial summary | 1/1 passed; transcript authoritative |
| Final core | core tests `16-06-25` | **Survives** | 56/56 passed |
| Initial main ambiguity | main tests `15-35-45` | **Unavailable** | 17 exact diagnostics retained above |
| Initial main execution | main tests `15-44-59` | **Unavailable** | 35/38; stale failure and two exact terminated IDs retained |
| Main isolated pair | main tests `16-41-03` | **Unavailable** | 2/2 passed |
| Main 37/38 | main tests, 16:45 run | **Unavailable** | Exact KILL identifier retained |
| Main isolated wallet | main tests `16-54-31` | Survives | 1/1 passed |
| Final main | main tests `16-56-19` | **Survives** | 38/38 passed, no retry |

The three decisive surviving bundles were hashed as a deterministic digest of
their sorted file-content SHA-256 values:

```bash
(cd "$bundle" && find . -type f -print0 | sort -z | xargs -0 shasum -a 256 | shasum -a 256 | awk '{print $1}')
```

| Bundle | Size | Tree-content SHA-256 | Extraction |
| --- | --- | --- | --- |
| `/tmp/tlcore-stage-2-red-valid/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_14-59-12-+0800.xcresult` | 508 KiB | Pre-extraction: `101b235583c7707f372120f90a18b0cdb1b7de8fff47471139d7dfdac57511ef`; stable post-extraction: `17ef9effa2af3bc18a4985ea6bd9311b5914784c2979cfaa2787aaa19b4386d8` | `xcresulttool get build-results build` returned the 14 diagnostics quoted above. The read updated the bundle's internal `database.sqlite3`; no diagnostic changed. |
| `/tmp/tlcore-stage-2-tests/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_16-06-25-+0800.xcresult` | 59 MiB | `dc080f2c268e36975abe3ba60ad0c806ae4454eaddd5f7b0518b56532dfa0644` | `result=Passed`, `passedTests=56`, `failedTests=0`, `skippedTests=0` |
| `/tmp/tronlink-tlcore-stage-2-tests/Logs/Test/Test-TronLink-2026.08.25_16-56-19-+0800.xcresult` | 57 MiB | `444e0f339133b2bc8b8d552fb3451003ed481950be84c6b82c59b8a120cb7e73` | `result=Passed`, `passedTests=38`, `failedTests=0`, `skippedTests=0` |

Extraction commands:

```bash
xcrun xcresulttool get build-results build --path "$RED" --format json | jq -r '.. | objects | select(.issueType? == "Swift Compiler Error") | .message'

xcrun xcresulttool get test-results summary --path "$CORE" --format json | jq '{title,result,testCount,passedTests,failedTests,skippedTests,startTime,endTime}'

xcrun xcresulttool get test-results summary --path "$MAIN" --format json | jq '{title,result,testCount,passedTests,failedTests,skippedTests,startTime,endTime}'
```

## Rollback commands

Rollback was documented but not invoked. List every documentation-only Task 2
commit after the implementation, including the ledger, and revert the displayed
commits newest first. Then revert consumer before provider:

```bash
git -C /Users/viccc/source/tronlink-iOS-core log --format='%H %s' 85fd3aac0fa79b8459459248e7b165dedb4927a5..HEAD -- docs/migration/03-keystore-migration.md docs/migration/evidence/task-2-command-ledger.md

git -C /Users/viccc/working/4_22_0/TronLink_iOS revert 664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b
git -C /Users/viccc/source/tronlink-iOS-core revert 85fd3aac0fa79b8459459248e7b165dedb4927a5
```

The first `git log` command is read-only. The `git revert` commands were never
executed during Task 2.
