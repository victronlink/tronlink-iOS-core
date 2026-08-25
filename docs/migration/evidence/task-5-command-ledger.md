# Task 5 命令与产物台账

本台账记录主工程功能兼容门禁与最终收口证据。时间均为 Singapore time
（`+0800`），日期为 2026-08-26。所有主工程 CocoaPods 解析都显式使用本地
`/Users/viccc/source/tronlink-iOS-core`。

## 证据记录规则

- `mtime` 是现存日志的文件系统修改时间；没有保存 shell 开始时间的命令不以
  `mtime` 冒充开始时间。
- Xcode 命令来自日志中的 `Command line invocation`，退出码来自当次执行结果，
  测试数来自 XCTest summary。
- `/tmp` 日志是当前工作机审计附件；稳定指纹使用单一日志或提取后的测试清单
  SHA-256，不使用读取时会变化的整个 `xcresult` tree hash。
- 用户自有 `Podfile.lock` 与 project 文件只在验证期间被 CocoaPods 更新，门禁结束
  后已恢复为 Task 5 起始字节。

## 固定起点、快照与提交

| 项目 | 值 |
| --- | --- |
| Core 仓库 | `/Users/viccc/source/tronlink-iOS-core` |
| Core Task 5 起点 | `695e0079dc25f7a909f2a1dcd8300114ba87f063` |
| Main 仓库 | `/Users/viccc/working/4_22_0/TronLink_iOS` |
| Main Task 5 起点 | `c641762a5502b28478b555b75c51d14f5cc29f39` |
| Main 类型边界测试提交 | `280145c1e029ccf967453d1e17c4ea506570611a` |
| Simulator UDID | `017E8DDA-425E-420F-9644-82B896E4907C` |
| Main 本地 core 入口 | `TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core` |

起始用户文件证据：

| 文件/证据 | SHA-256 |
| --- | --- |
| `/tmp/task5-main-start-Podfile.lock` | `3513cae6cf97a837f66a04240feb143fbc826701786f9ceb93652bfcf9921c65` |
| `/tmp/task5-main-start-project.pbxproj` | `739e70443445ad475c46dc5bfc49a688dbde923657b1221ce29a9a56ae3b12f8` |
| `/tmp/task5-main-start-Podfile.lock.diff` | `af0938ab42ec83a2ec8915d4aeb7c5efbb7a483e842e4070c4b9fa6df3c32896` |
| `/tmp/task5-main-start-project.pbxproj.diff` | `d015ab9f57155ccf26b30347515440ac0a7d81a5b8009150a13f0d14ea17fa60` |
| `/tmp/task5-main-start-status.txt` | `82d2f624ed54e4da9fb890875963cb5c5cca08da81494f5783ef3bd95d6a9828` |

## 类型边界测试与 CocoaPods 解析

### 首次 focused 运行：基础设施失败

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际进程命令：

```bash
xcodebuild test -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-5-focused CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTests/testTLCoreTypeNamesRemainUnambiguousInTheApp
```

- 时间证据：日志 `mtime=2026-08-26T01:10:34+0800`；shell 精确开始时间未单独保存。
- exit：65；在 XCTest discovery 前因 `[CP] Check Pods Manifest.lock` 失败，测试 0 项。
- 输出：`/tmp/task5-main-focused-type-resolution.log`，25,397,886 bytes，SHA-256
  `0c21a51e36e0ebfbf42feb8c7609f2197a34e544688eafd362693b9fea500e0f`；
  日志保留 `real 148.86`。
- 处置：按规定以本地 core 重新解析 Pods 后原命令重试；这不是功能失败。

### 本地 core CocoaPods 解析

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际可执行命令：

```bash
env TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core pod install
```

- 时间证据：日志 `mtime=2026-08-26T01:11:08+0800`。
- exit：0；113 total pods，12.41s。
- 输出：`/tmp/task5-main-pod-install.log`，2,449 bytes，SHA-256
  `61458d3856697e7ac045a631b4442f2e737e881001432ec6b8b3ae186abf053b`。
- 保存的 resolved `Podfile.lock` 与 `Manifest.lock` 分别位于
  `/tmp/task5-main-resolved-Podfile.lock`、`/tmp/task5-main-resolved-Manifest.lock`；两者
  SHA-256 均为
  `1ec58bfc333a9d4175cbaebb5d7035892858e93e9ed2c1b5ddb39e00795648a1`。

### focused 重试

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际进程命令：

```bash
xcodebuild test -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-5-focused CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTests/testTLCoreTypeNamesRemainUnambiguousInTheApp
```

- 时间证据：XCTest 完成时间 `2026-08-26T01:13:24+0800`；日志
  `mtime=2026-08-26T01:13:32+0800`。
- exit：0；1/1 通过，0 failure，测试 0.003s，`real 137.92`。
- xcresult：
  `/tmp/tronlink-tlcore-stage-5-focused/Logs/Test/Test-TronLink-2026.08.26_01-11-15-+0800.xcresult`。
- 输出：`/tmp/task5-main-focused-type-resolution-retry.log`，5,708,301 bytes，
  SHA-256 `03ce6dd9e5c34e44fa926224576b7e0ca690353e231abc5d87ed109185919d1d`。

## 主工程精选测试矩阵

两次运行的共同命令（第二次只在末尾增加一个精确 skip）是：

```bash
xcodebuild test -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-5-selected CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTest_importCreate -only-testing:TronLinkTests/TronLinkTest_scryptParams -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPrivateKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPublicKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletMnemonic -only-testing:TronLinkTests/TronLinkTest_walletCore/testMemoryPasswordWipedOnEnterBackground -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsFailureWhenKeystoreAccountIsMissing -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks -only-testing:TronLinkTests/TronLinkTest_exchange/testGetTokenExchangeBalanceContract -only-testing:TronLinkTests/TronLinkTest_exchange/testGetTokenApproveContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTrxToTokenSwapInputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTrxToTokenSwapOutputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTokenToTRXSwapInputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTokenToTokenSwapInputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTokenToTokenSwapOtputContract -only-testing:TronLinkTests/TronLinkTests/testNFTTransferFromRejectsInvalidABIAddress -only-testing:TronLinkTests/TronLinkTests/testTLCoreTypeNamesRemainUnambiguousInTheApp
```

### 未过滤运行

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 时间证据：XCTest 完成时间 `2026-08-26T01:25:28+0800`；日志
  `mtime=2026-08-26T01:25:47+0800`。
- exit：65；49 tests，恰好 1 failure，0 unexpected，测试 471.128s，
  `real 707.25`。
- 唯一失败：
  `TronLinkTest_importCreate.testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation()`；
  它是任务 brief 已裁定的陈旧断言，没有第二项失败。
- xcresult：
  `/tmp/tronlink-tlcore-stage-5-selected/Logs/Test/Test-TronLink-2026.08.26_01-14-01-+0800.xcresult`。
- 输出：`/tmp/task5-main-selected-unfiltered.log`，27,906,846 bytes，SHA-256
  `cd0aebf3c5cce4ff68d167f27755153ac574b12dce033611865e0df54b9de664`。
- 从日志提取的 49 条唯一 started 记录：
  `/tmp/task5-main-selected-unfiltered-tests.txt`，5,450 bytes，SHA-256
  `97664ce13b6158463ca9d8ffd60561cf7b69fc8ec3a4f82154b0f72e5a65277e`。

### 精确跳过陈旧断言

实际进程命令是共同命令再追加：

```text
-skip-testing:TronLinkTests/TronLinkTest_importCreate/testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation
```

完整字面命令：

```bash
xcodebuild test -workspace TronLink.xcworkspace -scheme TronLink -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tronlink-tlcore-stage-5-selected CODE_SIGNING_ALLOWED=NO -only-testing:TronLinkTests/TronLinkTest_importCreate -only-testing:TronLinkTests/TronLinkTest_scryptParams -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPrivateKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletPublicKey -only-testing:TronLinkTests/TronLinkTest_walletCore/testCreateHDWalletMnemonic -only-testing:TronLinkTests/TronLinkTest_walletCore/testMemoryPasswordWipedOnEnterBackground -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsFailureWhenKeystoreAccountIsMissing -only-testing:TronLinkTests/TronLinkTest_walletCore/testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks -only-testing:TronLinkTests/TronLinkTest_exchange/testGetTokenExchangeBalanceContract -only-testing:TronLinkTests/TronLinkTest_exchange/testGetTokenApproveContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTrxToTokenSwapInputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTrxToTokenSwapOutputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTokenToTRXSwapInputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTokenToTokenSwapInputContract -only-testing:TronLinkTests/TronLinkTest_exchange/testTokenToTokenSwapOtputContract -only-testing:TronLinkTests/TronLinkTests/testNFTTransferFromRejectsInvalidABIAddress -only-testing:TronLinkTests/TronLinkTests/testTLCoreTypeNamesRemainUnambiguousInTheApp -skip-testing:TronLinkTests/TronLinkTest_importCreate/testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation
```

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 时间证据：XCTest 完成时间 `2026-08-26T01:34:05+0800`；日志
  `mtime=2026-08-26T01:34:21+0800`。
- exit：0；48/48 通过，0 failure，测试 443.170s，`real 501.35`。
- xcresult：
  `/tmp/tronlink-tlcore-stage-5-selected/Logs/Test/Test-TronLink-2026.08.26_01-26-01-+0800.xcresult`。
- 输出：`/tmp/task5-main-selected-skip-stale.log`，2,490,450 bytes，SHA-256
  `9e9fc5b4da443e2992228090d31b2fb5eb4e2acfcae5e3dbfbba318c3e1fe530`。
- 从日志提取的 48 条唯一 started 记录：
  `/tmp/task5-main-selected-skip-stale-tests.txt`，5,318 bytes，SHA-256
  `94516a14aaf6e5ce12836a37b13ca1ad05c8e32ba2644c78a9440113a7630d2f`。
- 两份 started 清单的集合差恰好只有被精确 skip 的陈旧断言；因此不是仅凭 exit
  status 推断覆盖。

## 最终 lint 与构建

### Core pod lint

- cwd：`/Users/viccc/source/tronlink-iOS-core`
- 实际命令：

```bash
env XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests
```

- 临时配置 `/tmp/tlcore-lint-xcode26.xcconfig` 的 SHA-256 为
  `e8a40fb39aef577173fd073eb85cdc95a67447360f2453d0ef5a3e7aaa7653ae`，内容仅把
  deployment target 设为 iOS 13。
- 时间证据：日志 `mtime=2026-08-26T01:37:05+0800`。
- exit：0；`tronlink-iOS-core passed validation.`；`real 138.84`。
- 输出：`/tmp/task5-pod-lib-lint.log`，81,759 bytes，SHA-256
  `5c959eac379fac75eaddf99793435b0204b90d0442257cf9e60b847e3880dbb8`。

### Main Debug generic simulator

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际命令：

```bash
xcodebuild -workspace TronLink.xcworkspace -scheme TronLink -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tronlink-tlcore-final-debug CODE_SIGNING_ALLOWED=NO build
```

- 时间证据：日志 `mtime=2026-08-26T01:44:22+0800`。
- exit：0；`BUILD SUCCEEDED`；`real 427.49`。
- 输出：`/tmp/task5-main-debug-build.log`，48,687,282 bytes，SHA-256
  `d26bdf1c647c77c9b876ee832a0ce3b0d6e01286a1b578fefa6674d286bc9bc5`。

### Main Release generic device unsigned

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际命令：

```bash
xcodebuild -workspace TronLink.xcworkspace -scheme TronLink -configuration Release -sdk iphoneos -destination 'generic/platform=iOS' -derivedDataPath /tmp/tronlink-tlcore-final-release CODE_SIGNING_ALLOWED=NO build
```

- 时间证据：日志 `mtime=2026-08-26T01:48:49+0800`。
- exit：0；`BUILD SUCCEEDED`；`real 209.25`；主 App arm64 最终链接通过。
- 输出：`/tmp/task5-main-release-device-build.log`，25,958,035 bytes，SHA-256
  `cbd4f1a724de5c2ee16db2b53bd182e64814506be86913452ce1ba08be681872`。

## 最终静态与产物扫描

- cwd：`/Users/viccc/source/tronlink-iOS-core`
- timestamp：`2026-08-26T01:51:10+0800`
- exit：0
- 字面扫描命令：

```bash
rg -n '^[[:space:]]*(@testable[[:space:]]+)?import[[:space:]]+((class|struct|enum|protocol|func|var|let|typealias)[[:space:]]+)?(TronCore|TronKeystore|web3swift|tron_wallet_secp256k1|secp256k1)([.]|[[:space:]]|$)|\b(TronCore|TronKeystore|web3swift)\.' /Users/viccc/working/4_22_0/TronLink_iOS/TronLink /Users/viccc/working/4_22_0/TronLink_iOS/TronLinkTests
rg -n 'TronWalletABI|TronWalletKeystore|TronWalletWeb3Swift|tron-wallet-secp256k1|PromiseKit|scrypt[.]c' /tmp/task5-main-resolved-Podfile.lock /tmp/task5-main-resolved-Manifest.lock
find /tmp/tronlink-tlcore-final-release/Build/Products/Release-iphoneos -type d \( -name 'TronWalletABI.framework' -o -name 'TronWalletKeystore.framework' -o -name 'TronWalletWeb3Swift.framework' -o -name 'tron-wallet-secp256k1.framework' -o -name 'TronCore.framework' -o -name 'TronKeystore.framework' -o -name 'web3swift.framework' -o -name 'secp256k1.framework' \)
nm -gjU /tmp/tronlink-tlcore-final-release/Build/Products/Release-iphoneos/tronlink-iOS-core/TLCore.framework/TLCore | rg '^_?(secp256k1_|ecdsa_)' | sort | uniq -d
file /tmp/tronlink-tlcore-final-release/Build/Products/Release-iphoneos/tronlink-iOS-core/TLCore.framework/TLCore
lipo -info /tmp/tronlink-tlcore-final-release/Build/Products/Release-iphoneos/tronlink-iOS-core/TLCore.framework/TLCore
file /tmp/tronlink-tlcore-final-release/Build/Products/Release-iphoneos/TronLink.app/TronLink
```

结果：旧 import/qualified name 0；resolved 旧依赖 0；旧 framework directory 0；
TLCore 中 `secp256k1_*`/`ecdsa_*` unique 55、duplicate 0；TLCore 是 arm64 static
archive；App 是 arm64 Mach-O executable。组合输出
`/tmp/task5-final-static-scans.log` 为 661 bytes，
`mtime=2026-08-26T01:51:11+0800`，SHA-256
`277ff26b58af372ab52f41c24b58e1d25269b5a0ab1347beb74fcc03d9696e75`。

## 用户文件恢复

恢复命令：

```bash
cp /tmp/task5-main-start-Podfile.lock /Users/viccc/working/4_22_0/TronLink_iOS/Podfile.lock
cp /tmp/task5-main-start-project.pbxproj /Users/viccc/working/4_22_0/TronLink_iOS/TronLink.xcodeproj/project.pbxproj
```

`2026-08-26T01:51:20+0800` 复核：两个 live file 与起始快照的 `cmp -s` 均为
0；文件 SHA 分别仍为 `3513cae6…`、`739e7044…`；逐文件 `git diff --binary` SHA
分别仍为 `af0938ab…`、`d015ab9f…`。输出
`/tmp/task5-main-user-file-restoration.log` 为 685 bytes，SHA-256
`a9939ffef3deadcb2a1463e07f6d5388573f251d6beba02fe9cbc0135d520d12`。

Main 未跟踪的 `.agents/`、`AGENTS.md`、`BASIC_MODE_API_INVENTORY.md`、`outputs/`、
`reports/`、`scripts/assert_entropy_source.sh` 均保留；Task 5 没有把它们加入 index。

## 最终 history/worktree 审计

在 `07-final-report.md` 的独立 carrier commit
`2451f0e6584d28a02407cd61329393c6d27972ee` 创建后，逐字执行计划要求的六条命令：

```bash
git -C /Users/viccc/source/tronlink-iOS-core status --short --ignored docs/migration docs/superpowers
git -C /Users/viccc/source/tronlink-iOS-core log --oneline --decorate -8
git -C /Users/viccc/working/4_22_0/TronLink_iOS status --short
git -C /Users/viccc/working/4_22_0/TronLink_iOS log --oneline --decorate -8
git -C /Users/viccc/source/tronlink-iOS-core log --all -- docs/migration/00-single-tlcore-design.md docs/migration/00-design-audit.md docs/superpowers/plans/2026-08-25-single-tlcore-migration.md
git -C /Users/viccc/source/tronlink-iOS-core ls-files -- docs/migration/00-single-tlcore-design.md docs/migration/00-design-audit.md docs/superpowers/plans/2026-08-25-single-tlcore-migration.md
```

- timestamp：`2026-08-26T01:56:15+0800`；组合 shell exit 0。
- Core 顶部是 `2451f0e`、`03c9916`；Main 顶部是 `280145c1e`。
- Core status 仅显示三个预期 ignored 路径：`00-design-audit.md`、
  `00-single-tlcore-design.md`、`docs/superpowers/`。
- Main status 仅保留起始用户 dirty `Podfile.lock`、project 和已列出的未跟踪路径。
- 最后两条 design-history/index 命令均为**空输出**，证明三个设计/计划路径未进入
  任何 Git history 或 index。
- 输出：`/tmp/task5-final-history-audit.log`，1,275 bytes，
  `mtime=2026-08-26T01:56:15+0800`，SHA-256
  `e07e8fb6f346be6e095958cb209ab915205c5a96e0af9e73b707d61595c54734`。

本节必然晚于原始 Stage 5 证据提交和 final-report carrier，因此由单独的 docs-only
审计提交承载；`06-main-app-validation.md` 的 path-limited rollback 会按最新到最旧
发现本次台账修订与原始 `03c9916`。
