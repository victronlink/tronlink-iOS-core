# Stage 5: Main-App Validation

## Validated Core and App Commits

| Repository | Task 5 before | Validated after |
| --- | --- | --- |
| Core | `695e0079dc25f7a909f2a1dcd8300114ba87f063` | 生产源码未变；本报告与台账的 carrier commit 用下文路径限定命令发现 |
| Main | `c641762a5502b28478b555b75c51d14f5cc29f39` | `280145c1e029ccf967453d1e17c4ea506570611a` (`test: cover the consolidated TLCore module`) |

Main 提交只在 `TronLinkTests/TronLinkTests.swift` 增加一个 13 行测试，没有修改生产
代码、Podfile、lock 或 project。所有验证前先以
`TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core` 解析本地 TLCore。完整逐命令
证据见 [`evidence/task-5-command-ledger.md`](evidence/task-5-command-ledger.md)。

## Test-to-Risk Matrix

五个指定文件共有 92 个 `test*` 方法：Task 5 开始时 91 个，加上缺失的类型边界
测试 1 个。`精选=是` 表示进入未过滤 49 项矩阵；精确跳过陈旧断言后执行 48 项。
`否` 的原因明确标为网络/手工场景、空实现或与迁移无关，而不强行归入迁移风险。

### `TronLinkTest_importCreate.swift`（29）

整个 class 被显式选择，因此 29 个方法都在未过滤运行中执行。

| 精确方法名 | 风险门禁 | 精选 | 说明 |
| --- | --- | --- | --- |
| `testExample` | 无关 | 是 | XCTest 模板空断言；随整类执行 |
| `testPerformanceExample` | 无关 | 是 | XCTest 性能模板；随整类执行 |
| `testPrivateKeyKeystoreValidationRejectsInvalidCiphertextLengths` | JSON compatibility；create/import | 是 | 拒绝非法 cipher 长度 |
| `testPrivateKeyKeystoreValidationAccepts32ByteCiphertext` | JSON compatibility；create/import | 是 | 接受 32-byte cipher |
| `testKeystoreKDFGateSerializesConcurrentWork` | scrypt bounds/legacy files | 是 | KDF 并发串行化门禁 |
| `testBatchMnemonicImportInvalidPhraseCompletesOnce` | create/import；mnemonic/passphrase | 是 | 非法助记词 completion 单次返回 |
| `testBatchMnemonicImportCallsRealImporterOnce` | create/import；mnemonic/passphrase | 是 | 批量助记词真实 importer 调用计数 |
| `testAssociatedWalletBatchStopsAtFirstFailureAndKeepsEarlierSuccessCount` | create/import | 是 | 关联钱包批处理失败边界 |
| `testAssociatedWalletBatchRunnerSurvivesAsyncStepWithoutExternalRetention` | create/import | 是 | 异步批处理生命周期 |
| `testAssociatedWalletPersistenceRollsBackOnlyCurrentItemOnLinkFailure` | create/import；JSON compatibility | 是 | link 失败的持久化回滚 |
| `testAssociatedWalletPersistenceRollsBackKeystoreOnWalletFailure` | create/import；JSON compatibility | 是 | wallet 失败回滚 keystore |
| `testAssociatedWalletLinkBuilderUsesExplicitSourceWallet` | create/import | 是 | 关联来源钱包确定性 |
| `testHDRootWalletResolverPrefersTheIndexZeroRootWallet` | create/import；mnemonic/passphrase | 是 | HD root index 0 选择 |
| `testHDRootWalletResolverPrefersCurrentMemberWhenRootIsNotImported` | create/import；mnemonic/passphrase | 是 | root 未导入时选择当前成员 |
| `testHDRootWalletResolverSkipsGroupMembersMissingFromTheWalletTable` | create/import | 是 | 跳过缺失 DB 成员 |
| `testHDRootWalletResolverFailsClosedWhenNoGroupMemberIsPresent` | create/import | 是 | 无成员时 fail closed |
| `testHDRootWalletResolverRejectsEmptyRootAddress` | create/import | 是 | 拒绝空 root address |
| `testPrivateKeyKeystoreValidationRejectsUnsupportedVersion` | JSON compatibility；create/import | 是 | 拒绝不支持 JSON version |
| `testLegacyHDKeystoreRiskDetectorRequiresOverlengthEncryptedKeystorePayload` | JSON compatibility；scrypt bounds/legacy files | 是 | legacy HD 风险标记长度条件 |
| `testLegacyHDKeystoreRiskDetectorRejectsOtherImportAndAccountTypes` | JSON compatibility；create/import | 是 | 风险标记类型约束 |
| `testHDKeystoreImportJSONPreservesAddressAfterP0_01Fix` | JSON compatibility；create/import | 是 | HD JSON 导入后地址保持 |
| `testKeystoreKeyInitRejectsMnemonicASCIIPayload` | create/import；mnemonic/passphrase | 是 | 私钥入口拒绝 mnemonic ASCII |
| `testKeystoreKeyInitAcceptsPrintableASCII32BytePrivateKey` | create/import；private-key export | 是 | 32-byte 可打印私钥兼容 |
| `testKeystoreKeyInitAcceptsValid32BytePrivateKey` | create/import；private-key export | 是 | 有效二进制私钥兼容 |
| `testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation` | JSON compatibility；create/import | 是 | 已裁定陈旧断言；未过滤唯一失败，第二轮精确 skip |
| `testLegacyHDKeystoreImportedAsPrivateKeyKeepsAddressAndBecomesUsable` | create/import；JSON compatibility；private-key export | 是 | legacy HD 按私钥导入后地址/可用性 |
| `testClassifyImportAcceptsSelfConsistentKeystore` | create/import；JSON compatibility | 是 | 接受自洽 keystore |
| `testClassifyImportRejectsDeclaredAddressMismatch` | create/import；JSON compatibility | 是 | 拒绝声明地址不一致 |
| `testClassifyImportRejectsKeystoreWhoseAddressCannotBeDerived` | create/import；JSON compatibility | 是 | 拒绝无法派生地址 |

### `TronLinkTest_scryptParams.swift`（5）

| 精确方法名 | 风险门禁 | 精选 | 说明 |
| --- | --- | --- | --- |
| `testDefaultScryptParamsUseBalancedPreset` | scrypt bounds/legacy files | 是 | 默认 balanced 参数 |
| `testBalancedPresetValidates` | scrypt bounds/legacy files | 是 | 参数边界校验 |
| `testFreshKeystoreOnDiskUsesBalancedPreset` | scrypt bounds/legacy files；JSON compatibility | 是 | 新 JSON 落盘参数 |
| `testScryptDeterministicForBalancedPreset` | scrypt bounds/legacy files | 是 | KDF 确定性 |
| `testLegacyLightPresetKeystoreStillDecrypts` | scrypt bounds/legacy files；JSON compatibility | 是 | legacy light 文件解密 |

### `TronLinkTest_walletCore.swift`（11）

| 精确方法名 | 风险门禁 | 精选 | 说明/排除原因 |
| --- | --- | --- | --- |
| `testGetAccountBandwith` | 无关 | 否 | live gRPC 账户资源，网络重 |
| `testCheckTransferAddressActive` | 无关 | 否 | live gRPC 地址激活查询，网络重 |
| `testCreateHDWalletPrivateKey` | create/import；private-key export | 是 | 方法体当前全注释，是弱/空覆盖，不能单独证明导出 |
| `testCreateHDWalletPublicKey` | create/import；mnemonic/passphrase | 是 | 生成 HD wallet 并取公钥 |
| `testVoteInfo` | 无关 | 否 | live gRPC 投票账户查询 |
| `testCreateHDWalletMnemonic` | create/import；mnemonic/passphrase | 是 | 128-bit 助记词生成 |
| `testMemoryPasswordWipedOnEnterBackground` | password update/delete | 是 | 后台切换清空内存密码 |
| `testUpdateWalletReportsFailureWhenKeystoreAccountIsMissing` | password update/delete | 是 | 缺失账户更新失败 |
| `testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks` | password update/delete；private-key export | 是 | 新密码可导出、旧密码不可导出；defer 删除测试账户 |
| `testWalletShastaBalance` | 无关 | 否 | live token balance 路径 |
| `testVoteReward` | 无关 | 否 | live reward gRPC，且无等待/断言 |

### `TronLinkTest_exchange.swift`（8）

| 精确方法名 | 风险门禁 | 精选 | 说明/排除原因 |
| --- | --- | --- | --- |
| `testGetTokenExchangeBalanceContract` | ABIv2 routing | 是 | exchange balance calldata |
| `testGetTokenApproveContract` | ABIv2 routing | 是 | approve calldata |
| `testTrxToTokenSwapInputContract` | ABIv2 routing | 是 | TRX→token exact-input route |
| `testTrxToTokenSwapOutputContract` | ABIv2 routing | 是 | TRX→token exact-output route |
| `testTokenToTRXSwapInputContract` | ABIv2 routing | 是 | token→TRX route |
| `testTokenToTokenSwapInputContract` | ABIv2 routing | 是 | token→token exact-input route |
| `testTokenToTokenSwapOtputContract` | ABIv2 routing | 是 | token→token exact-output route（保留既有拼写） |
| `testGetTokenExchangeBalance` | ABIv2 routing | 否 | live contract/network 查询，非确定性；上面 7 项已覆盖本地编码路由 |

### `TronLinkTests.swift`（39）

| 精确方法名 | 风险门禁 | 精选 | 说明/排除原因 |
| --- | --- | --- | --- |
| `testTLCoreTypeNamesRemainUnambiguousInTheApp` | 模块/类型边界专项 | 是 | 21-byte Tron、20-byte Web3、App `Wallet`、`TLCore.Wallet.defaultPath` 四断言 |
| `testPerformanceExample` | 无关 | 否 | XCTest 性能模板 |
| `testDepositTRX` | transaction/message signing | 否 | sidechain live network/broadcast；不适合作为确定性门禁 |
| `testDepositTRC10` | transaction/message signing | 否 | sidechain live network/broadcast |
| `testDepositTRC20` | transaction/message signing；ABIv2 routing | 否 | sidechain live network/broadcast |
| `testWithdrawTRX` | transaction/message signing | 否 | 方法体全注释，无有效覆盖 |
| `testWithdrawTRC10` | transaction/message signing | 否 | 方法体全注释，无有效覆盖 |
| `testWithdrawTRC20` | transaction/message signing；ABIv2 routing | 否 | 方法体全注释，无有效覆盖 |
| `testRewordInfo` | 无关 | 否 | live reward 查询，无确定性断言 |
| `testBrokerageInfo` | 无关 | 否 | live brokerage 查询，无确定性断言 |
| `testCreatewitness` | transaction/message signing | 否 | 方法体全注释，无有效覆盖 |
| `testGetAccount` | 无关 | 否 | live account gRPC |
| `testCreateProposal` | transaction/message signing；cold wallet/multisig | 否 | 方法体全注释 |
| `testCreateProposalMulti` | cold wallet/multisig | 否 | live network；真正 multiSign 调用已注释 |
| `testDeleteProposalMulti` | cold wallet/multisig | 否 | live network；真正 multiSign 调用已注释 |
| `testListProposals` | 无关 | 否 | live governance gRPC |
| `testListProposals1` | 无关 | 否 | live governance route，最长 600s |
| `testGetProposalById` | 无关 | 否 | 固定远端节点 live 查询 |
| `testApproveProposal` | cold wallet/multisig | 否 | live network；真正 multiSign 调用已注释 |
| `testWithdrawRewards` | cold wallet/multisig | 否 | live network；真正 multiSign 调用已注释 |
| `testAccountPermissionUpdateContract` | cold wallet/multisig | 否 | 空方法 |
| `testAccountPermissionUpdateContractMulti` | cold wallet/multisig | 否 | 方法体全注释 |
| `testUnfreezeBalanceContractMulti` | cold wallet/multisig | 否 | live network；真正 multiSign 调用已注释 |
| `testBalanceOf` | ABIv2 routing | 否 | live TRC20 balance 查询 |
| `testFree` | transaction/message signing；cold wallet/multisig | 否 | 方法体全注释 |
| `testVoteTest` | transaction/message signing | 否 | 方法体全注释 |
| `testUpdateUserCreateBlockNum` | 无关 | 否 | live block/API 更新 |
| `testNFTTransferFromRejectsInvalidABIAddress` | ABIv2 routing；cold wallet/multisig | 是 | 普通/多签两条 route 的非法地址与 100-byte calldata |
| `testTransactionTRC20` | ABIv2 routing | 否 | live `triggerContract`，没有签名断言 |
| `testSubtracting` | 无关 | 否 | Decimal 算术 |
| `testAdd` | 无关 | 否 | Decimal 算术 |
| `testMultiplying` | 无关 | 否 | Decimal 算术 |
| `testDividing` | 无关 | 否 | Decimal 算术 |
| `testExplicitMainlandTrueIsEligibleToShow` | 无关 | 否 | Mainland popup 业务规则 |
| `testExplicitMainlandFalseIsNotEligibleToShow` | 无关 | 否 | Mainland popup 业务规则 |
| `testInvalidBusinessCodeIsNotEligibleToShow` | 无关 | 否 | Mainland popup 业务规则 |
| `testMissingRequiredDataIsNotEligibleToShow` | 无关 | 否 | Mainland popup 业务规则 |
| `testIncorrectFieldTypesAreNotEligibleToShow` | 无关 | 否 | Mainland popup 业务规则 |
| `testNilAndNonDictionaryResponsesAreNotEligibleToShow` | 无关 | 否 | Mainland popup 业务规则 |

精选矩阵的 49 项构成：import/create 29、scrypt 5、wallet core 6、exchange ABI 7、
NFT ABI 1、类型边界 1。没有把 live-network/空实现测试计为已验证功能。主工程没有
确定性的 signer recovery/deduplication 专项测试；该门禁由 Core 的 Stage 0/3
sign/recover golden 与 legacy differential 承担。transaction/message signing 的功能
证据同样来自 Core golden/differential，而不是上述 live-network App 方法。

## Targeted XCTest Commands and Results

新增测试前检索五个文件，没有现存方法同时验证四项类型解析，因此按计划原样新增
`testTLCoreTypeNamesRemainUnambiguousInTheApp`。首次 isolated 运行在测试 discovery
前因 Pods Manifest 不同步 exit 65；执行本地 core `pod install` 后，原命令重试 1/1
通过，0 failure。

精选矩阵的两次对照：

| 运行 | exit | 执行 | 失败 | 结论 |
| --- | ---: | ---: | ---: | --- |
| 未过滤 | 65 | 49 | 1 | 唯一失败是已裁定陈旧断言 `testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation`，0 unexpected |
| 仅精确 skip 该断言 | 0 | 48 | 0 | 48/48 通过 |

未过滤日志 SHA-256 为
`cd0aebf3c5cce4ff68d167f27755153ac574b12dce033611865e0df54b9de664`；
skip-only 日志 SHA-256 为
`9e9fc5b4da443e2992228090d31b2fb5eb4e2acfcae5e3dbfbba318c3e1fe530`。
从两份日志提取的唯一 started 条目分别是 49 和 48；集合差恰好只有该陈旧断言，
证明所有 intended selector 都实际执行。完整命令、xcresult 和清单 SHA 见 Task 5
台账。SPEC review 只读重提取分别得到相同 SHA
`97664ce13b6158463ca9d8ffd60561cf7b69fc8ec3a4f82154b0f72e5a65277e` 与
`94516a14aaf6e5ce12836a37b13ca1ad05c8e32ba2644c78a9440113a7630d2f`；`comm`
得到 unfiltered-only 1 行、skip-only 0 行，两个 expected-set `cmp` 及与原清单的两个
`cmp` 全为 0。

## Debug Simulator Build

Main Debug generic iOS Simulator 构建 exit 0，`BUILD SUCCEEDED`，`real 427.49`。
日志 `/tmp/task5-main-debug-build.log` 的 SHA-256 为
`d26bdf1c647c77c9b876ee832a0ce3b0d6e01286a1b578fefa6674d286bc9bc5`。

## Release Device Build

Main Release generic iOS device、`CODE_SIGNING_ALLOWED=NO` 构建 exit 0，
`BUILD SUCCEEDED`，`real 209.25`。最终 App 是 arm64 Mach-O；TLCore 是 arm64 static
archive。日志 `/tmp/task5-main-release-device-build.log` 的 SHA-256 为
`cbd4f1a724de5c2ee16db2b53bd182e64814506be86913452ce1ba08be681872`。

同一最终状态的 Core pod lint 使用 Xcode 26/iOS 13 临时 xcconfig，exit 0，
`tronlink-iOS-core passed validation.`；日志 SHA-256
`5c959eac379fac75eaddf99793435b0204b90d0442257cf9e60b847e3880dbb8`。

## Import and Qualified-Name Scan

对 Main `TronLink` 与 `TronLinkTests` 扫描旧 import、selective import 与旧模块限定名：
0 matches。Revision wrapper 显式捕获 `rg exit=1`、`tee exit=0`、0 rows，避免将
`tee` 的 0 错当成 `rg` 的状态。Release build tree 中旧 framework directory：0，
`find/sort/tee` 各段均 exit 0。没有把 TLCore 自身合法的 `secp256k1.h` header 名
误报成旧 module。

Device TLCore 的 `secp256k1_*`/`ecdsa_*` defined symbols 为 55 个 unique、0 duplicate；
arm64 App 最终链接没有 duplicate-symbol error。组合扫描日志 SHA-256 为
`277ff26b58af372ab52f41c24b58e1d25269b5a0ab1347beb74fcc03d9696e75`。Revision
wrapper 用两条完整 `nm -gjU | rg | sort` pipeline 分别生成 55 行 unique list 与
0 行 duplicate list，SHA-256 分别为
`2ffb61709a9739f5648464caa5ddd6fcf66ce3a1b3b480a5f99e676b065dd3cd`、
`e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`；同时重新确认
TLCore 是 arm64 static archive、App 是 arm64 Mach-O。

## Resolved Dependency Scan

本地 core `pod install` exit 0，113 total pods。保存的 resolved `Podfile.lock` 与
`Pods/Manifest.lock` SHA-256 均为
`1ec58bfc333a9d4175cbaebb5d7035892858e93e9ed2c1b5ddb39e00795648a1`。
对 `TronWalletABI`、`TronWalletKeystore`、`TronWalletWeb3Swift`、
`tron-wallet-secp256k1`、`PromiseKit`、`scrypt.c` 的扫描为 0 matches。
Revision wrapper 显式记录该 `rg exit=1`、`tee exit=0`、0 rows。

## Existing Dirty Files Preserved

Main 的两个用户自有 tracked dirty 文件以 accepted Task 4 的 immutable evidence
作为真正 pre-Task-5 ownership baseline。Task 5 当时保存的 `/tmp/task5-main-start-*`
mtime 是 `01:10:41`，晚于类型测试添加及首次 Manifest infrastructure failure
（日志 mtime `01:10:34`），且 status 已包含测试文件修改；因此它们只是
pod-install 前操作中快照，不能称为 strict pre-action snapshot。

最终 live 文件的 SHA 与 full `git diff --binary` SHA 逐项等于 Task 4 baseline：

| 文件 | Task 4 baseline / 最终文件 SHA-256 | Task 4 baseline / 最终 full-diff SHA-256 | 与操作中快照 `cmp` |
| --- | --- | --- | ---: |
| `Podfile.lock` | `3513cae6cf97a837f66a04240feb143fbc826701786f9ceb93652bfcf9921c65` | `af0938ab42ec83a2ec8915d4aeb7c5efbb7a483e842e4070c4b9fa6df3c32896` | 0 |
| `TronLink.xcodeproj/project.pbxproj` | `739e70443445ad475c46dc5bfc49a688dbde923657b1221ce29a9a56ae3b12f8` | `d015ab9f57155ccf26b30347515440ac0a7d81a5b8009150a13f0d14ea17fa60` | 0 |

Task 4 committed evidence 对 untracked 的可用 provenance 是“既有 untracked 文件均未
暂存”，没有提交 exact path inventory。最早保留的 exact Task 5 清单来自上述
01:10:41 操作中 status；其 `--untracked-files=all` 46 项与最终 46 项逐字节一致，
`cmp=0`，两份清单 SHA-256 均为
`45ee828dd788aad57dcd5108da0d1c741c56bab54d3b308b04b0b98584caccda`。这准确证明
操作中时点之后未增删用户路径，但不伪造缺失的 pre-action 46-path snapshot。

原恢复日志 SHA-256 为
`a9939ffef3deadcb2a1463e07f6d5388573f251d6beba02fe9cbc0135d520d12`；SPEC review
只读 revision log SHA-256 为
`86c1e2a83354890f21b7fe320884157fd9ef58510134e8f6709d91b97e140caa`。

## Deviations and Residual Risks

- 计划原本期望精选矩阵直接 exit 0；任务 brief 已明确要求对已知陈旧断言做两次
  对照。因此未过滤 49 项保留唯一已裁定失败，再用 exact skip 证明其余 48/48。
- focused 首次运行因 CocoaPods Manifest 不同步失败；本地 core 解析后同一 selector
  通过。这是完整保留的基础设施重试，不计为产品 RED。
- 原 Task 5 文档曾把 01:10:41 操作中快照称为起始/pre-action snapshot；本修订按
  mtime 与已含测试修改的 status 纠正时间线，并将真正 ownership baseline 锚定到
  accepted Task 4 Core `695e007…` / Main `c641762…` evidence。没有倒推或伪造缺失
  的更早 Task 5 snapshot。
- `testCreateHDWalletPrivateKey` 当前方法体全注释；private-key export 的有效 App
  断言来自 `testUpdateWalletReportsSuccessOnlyAfterNewPasswordWorks`，并由 Core golden
  补强。
- App inventory 没有可离线确定运行的 signer recovery/deduplication 或完整
  transaction/message signing 测试；Stage 0/3 的 Core sign/recover golden 和 legacy
  differential 是权威功能证据。cold wallet/multisig 在本次只执行了 NFT normal/multi
  ABI 路由；其余既有方法依赖 live network、sign 调用被注释或为空，未冒充为通过。
- lint 继续使用临时 iOS 13 xcconfig 适配 Xcode 26 的旧 libarclite 工具链差异；没有
  写入产品配置。
- Xcode/CocoaPods 仍有既有 deprecated/generated/run-script/network warning；没有
  TLCore 迁移相关 error。
- 用户 lock 按要求恢复，所以从 checkout 复验前必须重新执行本地路径 `pod install`；
  resolved 无旧依赖结论来自保存的 lock/Manifest。

## Rollback Commits

Task 5 文档提交不能在自身内容中硬编码自身 SHA。用一次三路径并集发现
Task 5 文档 carrier commits，确保同时修改多个路径的提交只回滚一次，再回滚
Main 的单测提交；命令明确限定仓库：

```bash
CORE_REPO=/Users/viccc/source/tronlink-iOS-core
MAIN_REPO=/Users/viccc/working/4_22_0/TronLink_iOS

git -C "$CORE_REPO" log --format='%H' 695e0079dc25f7a909f2a1dcd8300114ba87f063..HEAD -- docs/migration/06-main-app-validation.md docs/migration/07-final-report.md docs/migration/evidence/task-5-command-ledger.md |
while IFS= read -r task5_docs_commit; do
  test -n "$task5_docs_commit" || continue
  git -C "$CORE_REPO" revert --no-edit "$task5_docs_commit"
done

git -C "$MAIN_REPO" revert --no-edit 280145c1e029ccf967453d1e17c4ea506570611a
```

修正 carrier 产生前，在独立临时 clone 中以 Core
`0343feb1b1e4633c67588ab38a86da099622cd0d` 实际执行上述 Core 循环。发现顺序精确为
`0343feb` → `3f4b22d` → `e0b1110` → `2451f0e` → `03c9916`，循环 exit 0；
回滚后三个路径相对 `695e0079dc25f7a909f2a1dcd8300114ba87f063` 的 diff 为 0 行，
临时 clone 工作区也为 0 行。当前及以后修改这三个路径的 carrier 会被同一
union-path discovery 自动排在更旧提交之前，无需硬编码自身 SHA。

执行 rollback 前先确认两个仓库没有会与指定提交冲突的 staged change。Main 用户
自有 lock/project 与未跟踪文件不属于上述 revert 目标。
