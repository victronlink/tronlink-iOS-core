# 阶段 4：单一 TLCore 模块边界收口

## 结论

Task 4 已将最终依赖、源码、Swift/Objective-C/C 符号与主工程链接边界收口为单一
`TLCore` 模块。Core 与 Main 的实现提交分别为：

- Core：`9e535d3ca2b70715b0959676eb8b787aa6c12e8f`
- Main：`c641762a5502b28478b555b75c51d14f5cc29f39`

Core 提交只增加单模块公共 API smoke 和四个最终 manifest；Main 提交只删除已经
不存在的 `secp256k1.c` CocoaPods target post-install 分支。podspec 本阶段无需文本
修改，Example project 的随机 CocoaPods ID churn 已恢复，没有提交。

完整逐命令证据见
[`evidence/task-4-command-ledger.md`](evidence/task-4-command-ledger.md)。

## 最终模块与依赖契约

最终 podspec 保持：

- `s.module_name = 'TLCore'`
- iOS `13.0`
- Swift `4.2`
- Trezor 与 libsecp 的合并 header search paths/C flags
- `trezor-crypto/*.table` 的 `preserve_paths`
- `gRPC/**/*.pbrpc.m` 与 `ABI/ObjectiveC/EthereumCrypto.m` 的 ARC

外部直接依赖恰好 9 个：

| Pod | 固定版本 |
| --- | --- |
| BigInt | 3.1.0 |
| CryptoSwift | 1.8.4 |
| SwiftProtobuf | 1.38.1 |
| gRPC | 1.68.1 |
| Protobuf | 3.29.6 |
| gRPC-Core | 1.68.1 |
| gRPC-ProtoRPC | 1.68.1 |
| gRPC-RxLibrary | 1.68.1 |
| FMDB | 2.7.5 |

Example 与 Main 的最终 fresh resolved graph 对下列旧名字全部为 0：
`TronWalletABI`、`TronWalletKeystore`、`TronWalletWeb3Swift`、
`tron-wallet-secp256k1`、`PromiseKit`、`scrypt.c`；旧 target-support directory 也为 0。
Main resolved lock 与 Manifest 的相同 SHA 是
`1ec58bfc333a9d4175cbaebb5d7035892858e93e9ed2c1b5ddb39e00795648a1`。

## 公共 API 门禁与计划偏差

`SingleTLCorePublicAPITests` 只通过 `@testable import TLCore` 使用以下 API：

- mnemonic 创建 `Wallet` 并派生 index 0 key；
- `KeystoreKey` 生成 Tron `0x41` 前缀地址；
- ABI v1 `ABIEncoder` 编码 `BigUInt(42)` 为 32 bytes；
- Web3 `PrivateKey` 校验与地址有效性；
- `ABIv2Encoder` 编码 `uint256`。

计划样例要求 `Wallet.getKey(at: 0).address.data.first == 0x41`，真实 locked 行为是
20-byte Web3 address，首字节为 `0xc8`。初始测试按计划字面执行并 RED 后，没有改变
生产语义：最终断言 20-byte 地址与固定私钥 golden，同时由 `KeystoreKey` 单独覆盖
Tron 21-byte 地址的 `0x41` 前缀。恢复后的最终工程上 smoke 为 1/1 通过。

## 静态边界结果

| 门禁 | 结果 |
| --- | --- |
| 两仓旧模块 import | 0 |
| checked-in modulemap（排除 generated Pods） | 恰好 1：`tronlink-iOS-core/TLCore.modulemap` |
| module 声明 | `framework module TLCore` |
| `NS_SWIFT_NAME(TronProtoAddress)` | 恰好 1 |
| 迁入 Web3 的 `Data.hex` / `String.hex` 重复 | 0（receiver-aware） |
| public Swift type 重复 | 0 |
| Objective-C class 重复 | 0 |
| C symbol manifest 重复 | 0 |
| public `ecdsa.h` | 恰好 1 |

裸 `rg 'var hex: String' Data+Extension.swift` 会命中 `extension UInt8` 中仍被使用的
`UInt8.hex`；它不是 `Data.hex`，因此 receiver-aware 结果是权威门禁。
同理，生成头中的 `GAPIAnnotationsRoot (DynamicMethods)` 是 category；只提取
`@interface Name : Superclass` 后重复 class 为 0。所有矩阵编译也验证了不存在对应
redeclaration。

## 测试、构建与 lint

| 门禁 | 结果 |
| --- | --- |
| 未过滤 Example 全量 XCTest | exit 65；66 tests，恰好 4 个已裁定失败，0 unexpected |
| 精确跳过上述 4 项 | exit 0；其余 62/62 通过，0 failure |
| 最终 single-module focused smoke | exit 0；1/1 通过，11.447s |
| Example Debug simulator | exit 0 |
| Example Release simulator | exit 0 |
| `pod lib lint --allow-warnings --skip-tests` | exit 0；passed validation |
| Main Debug simulator | exit 0 |
| Main Release generic-device unsigned | exit 0；static-link/arm64 权威门禁 |

四个未在本阶段修复、且与迁移前裁定一致的失败是：

- `EmbeddedKeystoreTests.testKeystoreKeyRejectsAllPrintableASCIIInput()`
- `EmbeddedKeystoreTests.testKeystoreKeyRejectsMnemonicASCIIPayload()`
- `Tests.testBase58CheckRoundTripWithFlickrAlphabet()`
- `Tests.testSignTransaction()`

精确跳过四项后的 62 个测试全部通过，说明没有出现第五项回归。

## 构建产物

Release simulator 的 `TLCore.framework` 只暴露 9 个 TLCore module artifacts，没有旧
Swift module。架构与静态链接结果：

- Release simulator TLCore：`x86_64 arm64`，dynamic framework；
- Device TLCore：`arm64`，static archive；
- Main Debug app：`x86_64 arm64`；
- Main Device app：`arm64`；
- Main Debug/Device 构建树中的 legacy framework：0；
- Device TLCore 中 defined symbols 11,809；`secp256k1_*`/`ecdsa_*` 55，重复 0；
- Main Device app 不导出这些 static dead-stripped/hidden symbols，重复 0。

## 最终 manifest

| 文件 | 行数 | 文件 SHA-256 | 来源 |
| --- | ---: | --- | --- |
| `manifests/final-tlcore-sources.sha256` | 260 | `09bdc096f6b6248b83b257a249d2e7d77fa2d2a8ffdcd9ecd876bd53dc465de3` | 排序后的最终 `tronlink-iOS-core/Classes` 文件 SHA |
| `manifests/final-public-types.txt` | 62 | `e3fd911583ffdc75b74a81f334ae9ec1110a204f6d432744ed348ed52a2b31dc` | 最终源码 public type/extension 扫描 |
| `manifests/final-public-headers.txt` | 71 | `cbe4b814007babc3c4550cbc22a457a2655fa5e2680df4e09c30832f81ca2731` | Release simulator `TLCore.framework/Headers` |
| `manifests/final-c-symbols.txt` | 2,714 | `31c068887aeee90e2679a122b1aa9516b4dc526cd957c732e12775981633404c` | Release simulator final fat `TLCore.framework/TLCore` 的 `nm -gjU` 排序输出 |

首次 C symbol 生成选中同一 Release build 的 arm64 intermediate binary；对最终 fat
binary 重跑得到相同行数和 SHA，`cmp` 为 0，故提交结果与最终 framework 一致。

## 用户改动保护

Main 的 `Podfile.lock` 与 `TronLink.xcodeproj/project.pbxproj` 在 Task 4 开始前已经
dirty。CocoaPods 验证后，两文件均恢复到开始时保存的精确字节：

| 文件 | 开始/最终 SHA-256 | 开始/最终逐文件 diff SHA-256 |
| --- | --- | --- |
| `Podfile.lock` | `3513cae6cf97a837f66a04240feb143fbc826701786f9ceb93652bfcf9921c65` | `af0938ab42ec83a2ec8915d4aeb7c5efbb7a483e842e4070c4b9fa6df3c32896` |
| `TronLink.xcodeproj/project.pbxproj` | `739e70443445ad475c46dc5bfc49a688dbde923657b1221ce29a9a56ae3b12f8` | `d015ab9f57155ccf26b30347515440ac0a7d81a5b8009150a13f0d14ea17fa60` |

两组起始/最终 diff 的 `cmp` 都是 0。Main 提交只含 `Podfile`；用户未跟踪文件与
GasFree project reorder 均未进入提交。

## 主工程复验入口

由于 fresh resolved lock 只是验证产物，用户自有 `Podfile.lock` 按要求恢复而没有
提交。任何新的 Main 验证必须先显式解析当前本地 core：

```bash
cd /Users/viccc/working/4_22_0/TronLink_iOS
TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core pod install
```

随后再执行需要的 workspace test/build。不要用发布版 `tronlink-iOS-core 1.0.7`
代替本地迁移 checkout。

## 风险与限制

- 四个已裁定 Example 失败仍存在；本阶段以“未新增第五个失败”作为一致性门禁。
- lint 使用临时 iOS 13 xcconfig 绕过 Xcode 26 移除旧 libarclite 的工具链差异；该
  文件没有进入产品配置。
- Xcode/CocoaPods 仍输出既有 generated/deprecation/run-script warnings；本阶段没有
  观察到 TLCore 迁移相关 error。
- Main 用户自有 lock 被有意恢复，因此 checkout 后复验必须先执行上面的本地
  `pod install`；fresh graph 的无旧依赖结论来自保存的 resolved lock/Manifest。
- Task 4 只完成模块与依赖边界收口；主工程功能级兼容门禁属于 Task 5，尚未在本
  阶段展开。

## 回滚顺序

从最新到最旧执行：

1. 回滚记录本报告与 Task 4 command ledger 的 docs 提交；
2. 在 Main 回滚 `c641762a5502b28478b555b75c51d14f5cc29f39`；
3. 在 Core 回滚 `9e535d3ca2b70715b0959676eb8b787aa6c12e8f`。

回滚后按主工程复验入口重新执行 `pod install`，不要覆盖用户原有 lock/project 改动。
