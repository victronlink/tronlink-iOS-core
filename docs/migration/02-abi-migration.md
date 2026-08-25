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

At `2026-08-25T14:47:13+0800`, the following fail-fast check exited 0. It
asserted the pinned upstream HEAD, listed each permitted path from the pinned
tree, and streamed each pinned commit blob directly into `cmp`; neither tracked
nor untracked upstream worktree pollution can influence a comparison. It
compared all 123 class-source mappings (with the Objective-C relocation), both
license copies, and asserted a total of 125 mappings. A missing target or the
first byte mismatch terminates the command with exit 1.

```bash
set -euo pipefail
TLCORE_UPSTREAM_REPO=/Users/viccc/source/tronlink-iOS-core-others/tron-wallet-abi
TLCORE_PINNED=c367023e0e141f414c9319c2ccb382eda396f2a4
TLCORE_STAGED=/Users/viccc/source/tronlink-iOS-core/tronlink-iOS-core/Classes/ABI
TLCORE_LICENSE_DIR=/Users/viccc/source/tronlink-iOS-core/ThirdPartyLicenses
test "$(git -C "$TLCORE_UPSTREAM_REPO" rev-parse HEAD)" = "$TLCORE_PINNED"
TLCORE_SOURCE_COUNT=0
while IFS= read -r TLCORE_PATH; do
  case "$TLCORE_PATH" in
    TronWalletABI/Classes/TronCore-umbrella.h) continue ;;
    TronWalletABI/Classes/EthereumCrypto.h|TronWalletABI/Classes/EthereumCrypto.m)
      TLCORE_DESTINATION="$TLCORE_STAGED/ObjectiveC/${TLCORE_PATH##*/}" ;;
    TronWalletABI/Classes/*)
      TLCORE_DESTINATION="$TLCORE_STAGED/${TLCORE_PATH#TronWalletABI/Classes/}" ;;
    *) printf 'unexpected source path: %s\n' "$TLCORE_PATH" >&2; exit 1 ;;
  esac
  test -f "$TLCORE_DESTINATION" || {
    printf 'missing destination: %s\n' "$TLCORE_DESTINATION" >&2; exit 1;
  }
  git -C "$TLCORE_UPSTREAM_REPO" show "$TLCORE_PINNED:$TLCORE_PATH" | cmp -s - "$TLCORE_DESTINATION" || {
    printf 'mismatch: %s -> %s\n' "$TLCORE_PINNED:$TLCORE_PATH" "$TLCORE_DESTINATION" >&2; exit 1;
  }
  TLCORE_SOURCE_COUNT=$((TLCORE_SOURCE_COUNT + 1))
done < <(git -C "$TLCORE_UPSTREAM_REPO" ls-tree -r --name-only "$TLCORE_PINNED" -- TronWalletABI/Classes)
test "$TLCORE_SOURCE_COUNT" -eq 123 || { printf 'unexpected source count: %s\n' "$TLCORE_SOURCE_COUNT" >&2; exit 1; }
git -C "$TLCORE_UPSTREAM_REPO" show "$TLCORE_PINNED:LICENSE" | cmp -s - "$TLCORE_LICENSE_DIR/TronWalletABI-LICENSE" || { printf 'outer license mismatch\n' >&2; exit 1; }
git -C "$TLCORE_UPSTREAM_REPO" show "$TLCORE_PINNED:TronWalletABI/Classes/TrezorCrypto/trezor-crypto/LICENSE" | cmp -s - "$TLCORE_LICENSE_DIR/trezor-crypto-LICENSE" || { printf 'Trezor license mismatch\n' >&2; exit 1; }
TLCORE_TOTAL_COUNT=$((TLCORE_SOURCE_COUNT + 2))
test "$TLCORE_TOTAL_COUNT" -eq 125 || { printf 'unexpected total count: %s\n' "$TLCORE_TOTAL_COUNT" >&2; exit 1; }
printf 'verified=%s source_mappings=%s license_mappings=2 total_mappings=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')" "$TLCORE_SOURCE_COUNT" "$TLCORE_TOTAL_COUNT"
```

Its output was `verified=2026-08-25T14:47:13+0800 source_mappings=123
license_mappings=2 total_mappings=125`. The Solidity directory comparison also
exited 0:

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

To roll back Task 1, first inspect documentation-only amendments newer than the
initial report and revert every displayed commit in the displayed
newest-to-oldest order:

```bash
git log --format='%H %s' f19c61e56f081c1be42ea07d40a09fec8e0277a9..HEAD -- docs/migration/02-abi-migration.md .superpowers/sdd/2026-08-25-single-tlcore-migration/task-1-report.md
```

Then run these fixed commands, in this order:

```bash
git revert f19c61e56f081c1be42ea07d40a09fec8e0277a9
git revert 53da320b988d54ae74c969574f5ee6038a2e02c0
```

No rollback is required in the main-app checkout, whose pre-existing changes
were preserved.
