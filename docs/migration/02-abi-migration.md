# ABI source staging validation

## Scope

Commit `53da320` stages the pinned `TronWalletABI` 1.0.2 source snapshot from
`c367023e0e141f414c9319c2ccb382eda396f2a4`. It adds 123 files below
`tronlink-iOS-core/Classes/ABI`, preserves the inner TrezorCrypto hierarchy,
tables, and license, and retains both required third-party licenses. The
unmapped upstream `TronCore-umbrella.h` is intentionally not staged because it
is outside the approved mapping.

The current target explicitly excludes the staged tree:

```ruby
s.exclude_files = 'tronlink-iOS-core/Classes/ABI/**/*'
```

Legacy dependencies are unchanged. This keeps the addition behavior-neutral
and prevents duplicate C symbols while the statically linked consumer still
includes the legacy TronCore pod. Task 2 must remove the exclusion and remove
the old ABI linkage atomically; no ABI symbols are exported by TLCore in this
intermediate state.

## Fidelity

At `2026-08-25T14:38:34+0800`, a path-by-path `cmp -s` check covered 125
mapping pairs: 123 staged ABI files and the two copied license files. The
Solidity directory comparison also exited 0:

```bash
git diff --no-index \
  /Users/viccc/source/tronlink-iOS-core-others/tron-wallet-abi/TronWalletABI/Classes/Solidity \
  /Users/viccc/source/tronlink-iOS-core/tronlink-iOS-core/Classes/ABI/Solidity
```

`EthereumCrypto.m` has SHA-256
`b3809314bd8a28b03a9006298b6e479d91bb1e0e917efb3adeee4fb6c6bff464`
at both paths. The copied outer and Trezor license hashes are respectively
`9e886690e7511028efff4d42b2f4bf1efa4730252dbfb9a422fb8f311b845bfd` and
`b53d2eb93806f3d618370726f27eafeb6c279c7a90351297f79e073e49e569be`.
The pre-existing [ABI manifest](manifests/tron-wallet-abi-1.0.2.sha256)
records the 124 upstream class entries, including the deliberately unmapped
umbrella header.

## Validation

All tests used iPhone 17 Pro (`017E8DDA-425E-420F-9644-82B896E4907C`), arm64,
iOS Simulator 26.5.

| Gate | Command/result | Timestamp |
| --- | --- | --- |
| Upstream original harness | `xcodebuild test -workspace TronWalletABI.xcworkspace -scheme TronWalletABI-Example -destination "platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C" -derivedDataPath /tmp/tronwalletabi-source-baseline CODE_SIGNING_ALLOWED=NO` exited 65 before test discovery: Swift 4-mode test closure needs an explicit `return`. | `2026-08-25T14:16:49+0800` |
| Upstream Swift 5 attempt | Same command with `SWIFT_VERSION=5.0` exited 65 before tests: obsolete `UIApplicationLaunchOptionsKey` in the example harness. | `2026-08-25T14:24:33+0800` |
| Isolated compatibility harness | A disposable `/tmp` clone at the pinned commit changed only the test closure to return and the example signature to `UIApplication.LaunchOptionsKey`. With `SWIFT_VERSION=5.0`, it passed 7/7 with 0 failures (the six pinned functional tests plus the project performance test). The original upstream checkout remained at the pinned SHA and clean. | `2026-08-25T14:36:56+0800` |
| Core full suite | 33 tests: 31 passed, 2 failed, 0 skipped. The failures exactly match Stage 0: `Tests/testBase58CheckRoundTripWithFlickrAlphabet` and `Tests/testSignTransaction` (`WRONG_VERSION_NUMBER`). | `2026-08-25T14:21:24+0800` |
| Core comparison suite | Skipping only those two Stage 0 failures passed 31/31 with 0 failures. | `2026-08-25T14:27:10+0800` |
| Core migration vectors | `-only-testing:tronlink-iOS-core_Tests/SingleModuleMigrationBaselineTests` passed 4/4 with 0 failures or skips. | `2026-08-25T14:30:21+0800` |
| Core generated project | After `Example/pod install` (exit 0 at `2026-08-25T14:19:03+0800`), no `tronlink-iOS-core/Classes/ABI` path appears in `Example/Pods/Pods.xcodeproj/project.pbxproj`; legacy ABI remains separately linked. | `2026-08-25T14:24:08+0800` |
| Main app | Debug simulator build passed (`** BUILD SUCCEEDED **`) without running `pod install`. The existing `Podfile.lock` and project-file diffs both still byte-match Stage 0 evidence (each comparison exit 0). | build completed before `2026-08-25T14:35:00+0800`; diff check at that time |

The original full core command was:

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace \
  -scheme tronlink-iOS-core_Tests \
  -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' \
  -derivedDataPath /tmp/tlcore-stage-1-full-evidence CODE_SIGNING_ALLOWED=NO
```

The required main-app command was:

```bash
xcodebuild -workspace TronLink.xcworkspace -scheme TronLink -configuration Debug \
  -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' \
  -derivedDataPath /tmp/tronlink-tlcore-stage-1 CODE_SIGNING_ALLOWED=NO build
```

## Limitations, risks, and rollback

Xcode 26.6 cannot run the unmodified historical upstream example harness; the
isolated two-line compatibility run establishes the six functional tests
without changing upstream or library source. `git diff --check` reports
historical whitespace in byte-preserved upstream ABI/Trezor artifacts; those
files were intentionally not reformatted. The two full-suite failures remain
accepted Stage 0 baseline failures, not regressions from this excluded staging
change.

To roll back this task after its report commit, revert the documentation commit
first and then `53da320` (`refactor: stage TronWalletABI sources for
consolidation`). No rollback is required in the main-app checkout, whose
pre-existing changes were preserved.
