# Task 4 命令与产物台账

本台账记录“单一 TLCore 模块边界收口”的可复核执行证据。时间均为
Singapore time（`+0800`），日期跨越 2026-08-25 至 2026-08-26。验收以命令
退出码、日志中的测试计数以及日志 SHA-256 为准；不使用会被 Xcode 后续读取
所改变的整个 `xcresult` 目录哈希。

## 固定起点与提交

| 项目 | 值 |
| --- | --- |
| Core 仓库 | `/Users/viccc/source/tronlink-iOS-core` |
| Core Task 4 起点 | `1f2eff7ec420be7d014997aae4c656d2dd65d80a` |
| Core 实现提交 | `9e535d3ca2b70715b0959676eb8b787aa6c12e8f` |
| Main 仓库 | `/Users/viccc/working/4_22_0/TronLink_iOS` |
| Main Task 4 起点 | `45b4b6019de1b93c07eae13ee587d5bd948ecaab` |
| Main 清理提交 | `c641762a5502b28478b555b75c51d14f5cc29f39` |
| Main 本地 core 入口 | `TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core` |

## TDD 与 podspec 契约

| 时间 | 工作目录 | 字面命令 | 退出/结果 | 稳定日志 |
| --- | --- | --- | --- | --- |
| `23:27:44`–`23:29:29` | Core `Example` | `xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-4-red -only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests CODE_SIGNING_ALLOWED=NO` | 65；1 个测试执行、1 个预期失败。计划中的 `Wallet.getKey(...).address.data.first == 0x41` 实际得到 `0xc8`。 | `/tmp/task4-smoke-preinstall.log`，35,819 行，SHA `99db174eafe0e2099ff19ef43048f6345def7ff4e3464b0092ebebad7ef1efdc` |
| `23:29:49`–`23:30:10` | Core `Example` | 同上，仅把断言调整为既有 20-byte Web3 地址与固定私钥 golden，并用 `KeystoreKey` 单独验证 Tron `0x41` 地址前缀 | 0；1/1 通过，0 failure。 | `/tmp/task4-smoke-green.log`，3,964 行，SHA `0d4c2b38a6d8d1a7b3ed32a54b64d665aecf8a82976c19b84ab8f7506bd7c9e2` |
| `23:36:23` | Core | podspec 依赖数量与固定字段扫描 | 0；`podspec_dependencies=9`，`podspec_invariants=8`。 | `/tmp/task4-podspec-contract-scan.log`，SHA `9647048b5851488df7e0f08dcfbc9734295fc8ba5c1e32724c2d9f8fe629f481` |

上述 RED 暴露的是计划样例把两个地址域混在一起，而不是生产实现缺陷。
`Wallet.getKey` 保留 Web3 20-byte address 语义，`KeystoreKey.address` 保留 Tron
21-byte、`0x41` 前缀语义；没有为了测试改变生产代码。

## CocoaPods 重装与解析图

| 时间 | 工作目录 | 字面命令 | 退出/结果 | 稳定日志 |
| --- | --- | --- | --- | --- |
| `23:30:17`–`23:30:18` | Core `Example` | `pod deintegrate` | 0；只移除生成的 CocoaPods integration。 | `/tmp/task4-example-pod-deintegrate.log`，SHA `cf50b7645c6bba65a894ca0c5b3ec10e7c480fcd3d574ceed72648440c6ac0d9` |
| `23:30:30`–`23:33:43` | Core `Example` | `pod install` | 0；14 total pods。 | `/tmp/task4-example-pod-install.log`，SHA `49079cf51b049043c4bc77feb95a8eddbb569d301c15b2134cf79e9f7fd9631b` |
| `23:33:50` 前后 | Main | `TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core pod install` | 0；113 total pods。 | `/tmp/task4-main-pod-install.log`，SHA `94fa01f7c0ad28788d9075cb655f836f0c80d1433656672a89c91aff362fb36a` |
| `23:34:55`–`23:35:07` | Main | 删除已经确认不存在的 `secp256k1.c` target 分支后，再执行 `TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core pod install` | 0；113 total pods。 | `/tmp/task4-main-pod-install-final.log`，SHA `8357c9aa687cfaa0bb90ecccd265b8c3bb416eee89cc109faf33464394ed7abf` |
| `00:36:27` | 两仓 | 对 Example lock/Manifest、保存的 Main resolved lock/Manifest 和两个 `Target Support Files` 目录扫描 `TronWalletABI\|TronWalletKeystore\|TronWalletWeb3Swift\|tron-wallet-secp256k1\|PromiseKit\|scrypt.c` | 0；四项计数均为 0。 | `/tmp/task4-final-resolved-graph-gates.log`，SHA `d9f82af57fd75fb3c4a8da01b7ed2cdf7e178a13efbc71b7ee563dd2e5e6604f` |

最终 Main resolved `Podfile.lock` 与 `Pods/Manifest.lock` 字节相同，SHA-256 均为
`1ec58bfc333a9d4175cbaebb5d7035892858e93e9ed2c1b5ddb39e00795648a1`。
仓库内用户自有 lock 在验证后恢复，未用 resolved lock 覆盖它。

第一次沙箱内 Example `pod install` 因 CocoaPods cache/network 权限失败，随后在允许
依赖解析的环境重跑成功；失败日志被成功日志覆盖，不作为验收证据。

## 源码与模块边界

执行的核心扫描：

```bash
rg -n '^[[:space:]]*(@testable[[:space:]]+)?import[[:space:]]+(TronCore|TronKeystore|web3swift|tron_wallet_secp256k1|secp256k1)([.]|[[:space:]]|$)' \
  tronlink-iOS-core Example \
  /Users/viccc/working/4_22_0/TronLink_iOS/TronLink \
  /Users/viccc/working/4_22_0/TronLink_iOS/TronLinkTests
find . -name '*.modulemap' -not -path './Example/Pods/*' -print
rg -n 'NS_SWIFT_NAME\(TronProtoAddress\)' \
  tronlink-iOS-core/Classes/gRPC/api/Api_Tron.pbobjc.h
```

结果：旧模块 import 0；非生成 modulemap 恰好 1 个，即
`tronlink-iOS-core/TLCore.modulemap`，且声明 `framework module TLCore`；
`TronProtoAddress` annotation 恰好 1 个；podspec dependency 恰好 9 个。

计划里的裸文本 `var hex: String` 扫描在 `Data+Extension.swift:140` 命中的是
`extension UInt8` 内仍被使用的 `UInt8.hex`，并非 `Data.hex` 重复声明。按 extension
receiver 追踪的扫描对迁入 Web3 文件中的 `Data.hex` 与 `String.hex` 均返回 0；空结果
日志 `/tmp/task4-receiver-aware-web3-hex-scan.log` SHA 为
`e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`。

Objective-C 重复门禁只提取真实类声明 `@interface Name : Superclass`，结果为 0；
`GAPIAnnotationsRoot (DynamicMethods)` 单独识别为 1 个合法 category，不是重复类型：

```bash
rg -n '^@interface [A-Za-z_][A-Za-z0-9_]*[[:space:]]*:' tronlink-iOS-core/Classes \
  | sed -E 's/.*@interface ([A-Za-z_][A-Za-z0-9_]*)[[:space:]]*:.*/\1/' \
  | sort | uniq -d
```

真实类重复日志为空，SHA
`e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`；category
记录日志 SHA `93835580f4d89d54878f252aad73343196c602cad8b357a4e33ec30501b6f7a9`。

## 串行测试、构建与 lint 矩阵

所有下列命令串行执行。

| 时间 | 命令摘要 | 退出/验收结果 | 稳定日志 SHA-256 |
| --- | --- | --- | --- |
| `23:36:39`–`23:54:00` | `xcodebuild test ... -derivedDataPath /tmp/tlcore-stage-4-tests-full` | 65；66 tests，恰好 4 个已裁定失败，0 unexpected；执行 940.803s。 | `3f9c7999b39b1ddafeb800cff478a2d2a17fa59e8904c910bdd967e5fa8d3e50` |
| `23:54:15`–`00:09:22` | 同一 full-suite 命令，精确追加四个 `-skip-testing` selector | 0；其余 62/62 通过，0 failure；执行 887.165s；包含新 single-module smoke。 | `cc474eb863fdad22d3c643995e8d617e93c1abbe1faa71be9563b88bbbaa9921` |
| `00:09:34`–`00:12:19` | `xcodebuild build ... tronlink-iOS-core_Example -configuration Debug -sdk iphonesimulator ... -derivedDataPath /tmp/tlcore-stage-4-debug` | 0，`BUILD SUCCEEDED`。 | `439f7dbd40e1942a941b158d5de32d7b0d1e7207502731546f170fd041450bf8` |
| `00:12:31`–`00:15:36` | `xcodebuild build ... tronlink-iOS-core_Example -configuration Release -sdk iphonesimulator ... -derivedDataPath /tmp/tlcore-stage-4` | 0，`BUILD SUCCEEDED`。 | `e7a28c317cf953ffbaf1b597d075b3b43ec58121a676a13044c5334b0fc0b559` |
| `00:15:55`–`00:21:31` | `env XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests` | 0，`tronlink-iOS-core passed validation.` | `da2f0cb51221d5c1116e332733bf978c4d3330c82eb4c902f075013fba4a4f9c` |
| `00:21:46`–`00:28:48` | `xcodebuild -workspace /Users/viccc/working/4_22_0/TronLink_iOS/TronLink.xcworkspace -scheme TronLink -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tronlink-tlcore-stage-4-debug CODE_SIGNING_ALLOWED=NO build` | 0，`BUILD SUCCEEDED`。 | `da0aeaf0eecd67a4e84e9af662dcbdbe576a506f12922f42fd26199bac4c1294` |
| `00:29:01`–`00:32:52` | 同一 Main workspace 的 `Release`、`iphoneos`、`generic/platform=iOS`、unsigned device build，DerivedData `/tmp/tronlink-tlcore-stage-4-device` | 0，`BUILD SUCCEEDED`；这是 static-link/arm64 权威门禁。 | `42a0585cfc0d5fba2c2b3c4205e4239cf2ece18cb1f26372d18a14c748d496fb` |
| `00:38:32`–`00:40:10` | 恢复 Example 工程文件后，用当前模拟器执行 `-only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests` | 0；1/1 通过，0 failure，11.447s。 | `92de94dd6a334b027fc3e8150086c4d09de1b0cdd502e53ddff3d3a0827f93f0` |

四个既有且已裁定的失败名称恰好是：

- `EmbeddedKeystoreTests.testKeystoreKeyRejectsAllPrintableASCIIInput()`
- `EmbeddedKeystoreTests.testKeystoreKeyRejectsMnemonicASCIIPayload()`
- `Tests.testBase58CheckRoundTripWithFlickrAlphabet()`
- `Tests.testSignTransaction()`

lint 临时 xcconfig 只含 `IPHONEOS_DEPLOYMENT_TARGET = 13.0`，SHA-256
`e8a40fb39aef577173fd073eb85cdc95a67447360f2453d0ef5a3e7aaa7653ae`；
它用于适配 Xcode 26 已移除旧 libarclite 的环境差异，没有提交到仓库。

最终 focused smoke 第一次复验使用了已被删除的旧 simulator UDID，exit 70，未进入
编译/测试；随后使用当前可用 simulator 重跑得到上表 exit 0 结果。该基础设施失败不
作为代码结论。

## 产物、manifest 与符号门禁

执行 `find`、`lipo -archs`、`file` 与 `nm` 后得到：

| 项目 | 结果 |
| --- | --- |
| Release simulator TLCore | `x86_64 arm64`，动态 framework；binary SHA `4561a044ab17725e1d7348451b3b6c701cb6ae454accd18c68cfda641e984a8e` |
| Device TLCore | `arm64`，static `ar` archive；SHA `e1df238edb49fd2eced6c43794c901887303d83e517f2e0e74482084a4651074` |
| Main Debug app | `x86_64 arm64`；SHA `38a3f9cc4cb6948519d99f90bdf70762d8beebf1d790fa8a4d38f198d5b57987` |
| Main Device app | `arm64`；SHA `c6babfc9957c480f701135d316d78c43e6e66ed125f5403ecb2384d8da2f4048` |
| TLCore module artifacts | 9 个，全部是 `TLCore.swiftmodule` 相关文件或 `module.modulemap` |
| Main 两个构建中的 legacy framework | 0 |
| Device TLCore defined symbols | 11,809 行 |
| Device TLCore `secp256k1_*`/`ecdsa_*` symbols | 55 行，重复 0 |
| Main Device app 导出同类符号 | 0（static dead-strip/hidden），重复 0 |

产物路径与架构日志 SHA 分别为
`228e6bfe79a46c94eb7ff16d475a2bc4d7232222b6864fbb791585b3273f8a98`
和 `6ebaff065ad1ce382412a5bad11cdd29df3474f7efc35d1ed444f9803e0b0052`。

manifest 生成命令：

```bash
find tronlink-iOS-core/Classes -type f -print0 | sort -z \
  | xargs -0 shasum -a 256 \
  > docs/migration/manifests/final-tlcore-sources.sha256
rg -n '^public (final )?(class|struct|enum|protocol|typealias)|^public extension ' \
  tronlink-iOS-core/Classes | sort \
  > docs/migration/manifests/final-public-types.txt
find /tmp/tlcore-stage-4/Build/Products/Release-iphonesimulator/tronlink-iOS-core/TLCore.framework/Headers \
  -type f -exec basename {} \; | sort \
  > docs/migration/manifests/final-public-headers.txt
nm -gjU /tmp/tlcore-stage-4/Build/Products/Release-iphonesimulator/tronlink-iOS-core/TLCore.framework/TLCore \
  | sort > docs/migration/manifests/final-c-symbols.txt
```

实际首次生成 `final-c-symbols.txt` 时 `find -print -quit` 选中了同一 Release build 的
arm64 intermediate binary；对最终 fat framework binary 重新执行上面的 `nm` 后得到
2,714 行、相同 SHA，并由 `cmp` 证明字节一致。因此已提交 manifest 的产物来源可归一为
最终 Release `TLCore.framework/TLCore`。

Swift public type 重复、真实 Objective-C class 重复、最终 C symbol 重复三项均为 0。

## 用户文件恢复与提交命令

Main 的 `Podfile.lock` 与 `TronLink.xcodeproj/project.pbxproj` 在 Task 4 前已包含用户
改动。验证前保存文件与逐文件 diff；验证结束后按保存副本恢复，并执行 `cmp`：

| 文件 | Task 4 起始/最终文件 SHA | 起始/最终 diff SHA | `cmp` |
| --- | --- | --- | --- |
| `Podfile.lock` | `3513cae6cf97a837f66a04240feb143fbc826701786f9ceb93652bfcf9921c65` | `af0938ab42ec83a2ec8915d4aeb7c5efbb7a483e842e4070c4b9fa6df3c32896` | 0 |
| `TronLink.xcodeproj/project.pbxproj` | `739e70443445ad475c46dc5bfc49a688dbde923657b1221ce29a9a56ae3b12f8` | `d015ab9f57155ccf26b30347515440ac0a7d81a5b8009150a13f0d14ea17fa60` | 0 |

Example project 在 `pod deintegrate`/`pod install` 后的 126 行 diff 仅重写 CocoaPods
file reference、build phase 与 group 随机 ID；路径、配置与 phase 内容无意图变化。
恢复到 HEAD 后 focused smoke 仍通过，所以该生成噪音没有提交。

提交前都使用显式白名单路径，并执行 `git diff --cached --stat` 与
`git diff --cached --check`：

```bash
git add Example/Tests/Tests.swift \
  docs/migration/manifests/final-tlcore-sources.sha256 \
  docs/migration/manifests/final-public-types.txt \
  docs/migration/manifests/final-public-headers.txt \
  docs/migration/manifests/final-c-symbols.txt
git commit -m "chore: enforce the single TLCore module boundary"

git -C /Users/viccc/working/4_22_0/TronLink_iOS add Podfile
git -C /Users/viccc/working/4_22_0/TronLink_iOS commit \
  -m "chore: remove legacy TLCore transitive pods"
```

Core 提交只含 5 个白名单文件；Main 提交只含 `Podfile` 的 9 行删除。用户的
`.agents/`、`AGENTS.md`、`BASIC_MODE_API_INVENTORY.md`、`outputs/`、`reports/`、
`scripts/assert_entropy_source.sh`、lock diff 与 GasFree project reorder 均未暂存。
