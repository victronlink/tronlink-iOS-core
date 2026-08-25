# Single TLCore Migration Final Report

总体结论：**迁移兼容性 PASS**。原先由 TLCore、TronWalletABI、
TronWalletKeystore、TronWalletWeb3Swift 与 tron-wallet-secp256k1 共同提供的主工程
可达能力，已经收口为单一 `TLCore` Swift module；主工程最终 Debug simulator 与
Release device arm64 静态链接均通过，没有旧 module/import/pod/framework 或重复
secp/ecdsa definition。

这个结论遵循已裁定的兼容门禁，不等价于“所有历史测试字面全绿”：Example 未过滤
66 项有精确 4 个既有/矛盾失败，跳过这 4 项后其余 62/62 通过；其中 embedded
Keystore 为 25/27，通过之外的 2 项是已裁定的 printable-ASCII 矛盾断言；Main
精选未过滤 49 项有精确 1 个陈旧断言失败，exact skip 后 48/48 通过。

## Final Core Commit

- 最终生产实现提交：`9e535d3ca2b70715b0959676eb8b787aa6c12e8f`
  (`chore: enforce the single TLCore module boundary`)。
- Task 4 最终边界证据基线：`695e0079dc25f7a909f2a1dcd8300114ba87f063`。
- Task 5 主工程验证证据：`03c9916707f980c2a654ab330db17112b265f7b9`
  (`docs: record main-app TLCore validation`)。
- 本报告的 carrier commit 不能在自身内容中循环引用；用下述确定性命令发现：

```bash
git -C /Users/viccc/source/tronlink-iOS-core log --format='%H %s' 03c9916707f980c2a654ab330db17112b265f7b9..HEAD -- docs/migration/07-final-report.md
```

## Final Main-App Commit

- 最终生产消费边界：`c641762a5502b28478b555b75c51d14f5cc29f39`
  (`chore: remove legacy TLCore transitive pods`)。
- 最终 App 类型解析测试：`280145c1e029ccf967453d1e17c4ea506570611a`
  (`test: cover the consolidated TLCore module`)。

Task 5 的 Main 提交只修改 `TronLinkTests/TronLinkTests.swift`，增加一个覆盖四项类型
边界的测试；没有提交用户自有 lock/project 或生产代码。

## Embedded Source Baselines

| Source | 固定 commit | Tag | 基线 manifest 条目 |
| --- | --- | --- | ---: |
| 原 TLCore | `eb25afce43edeaae7a3ba2bfaef2bd83e99756f5` | `1.0.7` | 57 |
| TronWalletABI | `c367023e0e141f414c9319c2ccb382eda396f2a4` | `1.0.2` | 124 |
| TronWalletKeystore | `be7ad15ffa6fcb4c902bc19529f738a79576c881` | `1.0.5` | 13 |
| TronWalletWeb3Swift | `2b24ba4e65a3cf2026d697396a95ba7e1937e325` | `1.1.2` | 117 |
| tron-wallet-secp256k1 | `5d74ae264f59f5b98c5832f79cc33c2ed9bad82d` | `1.0.0` | 45 |

五份基线 manifest 共 356 项，均从对应 pinned Git blob/source root 验证。最终 TLCore
边界的四份 manifest：

| 文件 | 行数 | SHA-256 |
| --- | ---: | --- |
| `final-tlcore-sources.sha256` | 260 | `09bdc096f6b6248b83b257a249d2e7d77fa2d2a8ffdcd9ecd876bd53dc465de3` |
| `final-public-types.txt` | 62 | `e3fd911583ffdc75b74a81f334ae9ec1110a204f6d432744ed348ed52a2b31dc` |
| `final-public-headers.txt` | 71 | `cbe4b814007babc3c4550cbc22a457a2655fa5e2680df4e09c30832f81ca2731` |
| `final-c-symbols.txt` | 2,714 | `31c068887aeee90e2679a122b1aa9516b4dc526cd957c732e12775981633404c` |

Task 5 最终复核 `final-tlcore-sources.sha256` 的 260 个路径全部 `OK`。

## Final Directory and Module Boundary

所有合并源码都留在 core 仓库并由同一个 pod target 编译：

```text
tronlink-iOS-core/Classes/
├── ABI/                 Tron ABI v1、TrezorCrypto、Objective-C bridge
├── Keystore/            TronWalletKeystore 实现
├── Web3Subset/          主工程实际可达的 ABIv2/sign/recover 子集
├── Secp256k1/           libsecp256k1；实现文件改名 wallet_secp256k1.c
├── gRPC/                原 TLCore protobuf/gRPC
└── existing TLCore APIs
```

最终 podspec 保持 `s.module_name = 'TLCore'`、Swift 4.2、iOS 13；
`gRPC/**/*.pbrpc.m` 与 `ABI/ObjectiveC/EthereumCrypto.m` 显式启用 ARC，避免原 ABI
代码落入全局 MRC。Trezor 与 libsecp 各自保留 include hierarchy；两个原同名
`secp256k1.c` 不再产生相同 object basename。Main 不承载复制源码，只消费
`TLCore`。

## Public Type Mapping

| 语义 | 最终类型/入口 | 约束 |
| --- | --- | --- |
| TRON 地址 | `TLCore.Address` | 21-byte，含 `0x41` network prefix |
| EVM/Web3 地址 | `TLCore.Web3Address` | 20-byte raw address |
| 主工程钱包模型 | unqualified `Wallet` | 解析为 TronLink App 类型 |
| HD/keystore wallet | `TLCore.Wallet` | `defaultPath = m/44'/195'/0'/0/0` 保持可用 |
| protobuf 地址桥 | `TronProtoAddress` | 避免与 Swift `Address` 冲突 |
| Web3 私钥/签名 | `TLCore.PrivateKey`、`TLCore.Signature` | 使用内嵌 secp256k1 |
| ABI v1/v2 | `ABIEncoder`/`ERC20Encoder`、`ABIv2*` | 同一 TLCore namespace |

Main 的 `testTLCoreTypeNamesRemainUnambiguousInTheApp` 对前四行逐项断言，isolated
1/1 通过。

## Final External Dependencies

TLCore 最终只直接依赖以下 9 个外部 Pod：

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

Main fresh resolved lock/Manifest 对 `TronWalletABI`、`TronWalletKeystore`、
`TronWalletWeb3Swift`、`tron-wallet-secp256k1`、`PromiseKit`、`scrypt.c` 均为
0 matches。

## Web3 Whitelist and Exclusions

从 pinned TronWalletWeb3Swift 迁入的是主工程可达闭包，而非整库：7 个 ABIv2
文件、`PrivateKey`/`keccak`/libsecp bridge、9 个 required support 文件，再加两个
为闭包补齐的 focused compatibility 文件，共 21 个最终文件。公共地址类型改名为
`Web3Address`/`Web3AddressError`，以保留 `TLCore.Address` 的 TRON 语义。

明确排除：Web3 provider/network 层、PromiseKit、Web3 keystore、scrypt/PBKDF 尾部、
RLP/transaction implementations、contracts、EIP67、ENS/BIP67、ERC721/ERC777、
完整 browser/provider conveniences。`scrypt.c` 与 PromiseKit 没有因子集迁移重新成为
依赖；keystore 采用已迁入的 TronWalletKeystore，而不是 Web3Swift keystore。

## Keystore Compatibility Evidence

- embedded Keystore 上游兼容套件：27 项中 25 项通过。
- 两个未通过且已裁定为与批准产品行为相矛盾的 printable-ASCII 断言是：
  - `EmbeddedKeystoreTests.testKeystoreKeyRejectsMnemonicASCIIPayload()`
  - `EmbeddedKeystoreTests.testKeystoreKeyRejectsAllPrintableASCIIInput()`
- Main 的批准行为由
  `testKeystoreKeyInitAcceptsPrintableASCII32BytePrivateKey`、有效 32-byte 私钥、legacy
  HD JSON/地址保持、classify import、KDF 串行化、balanced/legacy light scrypt、密码
  更新与新旧密码导出断言覆盖。
- Main 未过滤精选矩阵中的另一个历史陈旧断言
  `testUntrustedLegacyRiskMarkerCannotBypassPrivateKeyImportValidation()` 被单独保留并
  裁定；精确 skip 后其余 48/48 通过。

因此 Keystore **迁移兼容门禁 PASS**，但“所有上游 Keystore 断言字面通过”这一更强
表述为 false，不能打勾或用于发布声明。

## ABI and Signing Differential Evidence

- Stage 0 固定向量覆盖 ABI `uint256(42)`、Web3 private key `00…01` 对 32 bytes
  `0x11` 的 sign/recover、地址
  `0x7e5f4552091a69125d5dfcb7b8c2659029395bdf`，以及 TRON 标准 abandon mnemonic
  派生私钥 `b5a4cea271ff424d7c31dc12a3e43e401df7a40d7412a15750f3f0b6b5449a28`。
- Stage 3 在旧 Web3 module 尚可并行加载时执行 3 个 dynamic differential：
  private-key sign/recover、ABIv2 parser/uint、static/dynamic/address encode/decode 均一致。
- 移除旧 module 后的 5 个 Web3 golden 全绿，保留 sign/recover、parser、uint、
  static/dynamic/address 行为。
- Main 精选 7 个 exchange ABI route 与 NFT normal/multisig 两条 route 通过；非法地址
  返回 nil，有效 transferFrom calldata 为 100 bytes。

这些证据覆盖 ABI 与 sign/recover 的功能一致性。Main 既有 transaction/multisig
方法大多依赖 live network 或将真正 sign 调用注释掉，未把它们虚报为确定性测试。

## Build and Test Matrix

| 门禁 | 结果 | 判定 |
| --- | --- | --- |
| Example 未过滤全量 | 66 tests，4 failures，0 unexpected | 精确四项已裁定；未新增第五项 |
| Example exact skip 四项 | 62/62，0 failure | PASS |
| Embedded Keystore 子集 | 25/27；两个 printable-ASCII 矛盾断言 | 兼容 PASS；非“全部断言通过” |
| Main 类型边界 isolated | 1/1，0 failure | PASS |
| Main 精选未过滤 | 49 tests，精确 1 个陈旧断言失败 | 已裁定；无第二项失败 |
| Main exact skip 陈旧断言 | 48/48，0 failure | PASS |
| Example Debug simulator | `BUILD SUCCEEDED` | PASS |
| Example Release simulator | `BUILD SUCCEEDED` | PASS |
| Main Debug generic simulator | `BUILD SUCCEEDED`，427.49s | PASS |
| Main Release generic device | `BUILD SUCCEEDED`，arm64 static link，209.25s | PASS |
| Core pod lint | `tronlink-iOS-core passed validation.` | PASS |
| Main old import/qualified scan | 0 | PASS |
| resolved old pod scan | 0 | PASS |
| device old framework scan | 0 | PASS |
| device TLCore secp/ecdsa | 55 unique，0 duplicate | PASS |

Example 未过滤的精确四项是：

- `EmbeddedKeystoreTests.testKeystoreKeyRejectsAllPrintableASCIIInput()`
- `EmbeddedKeystoreTests.testKeystoreKeyRejectsMnemonicASCIIPayload()`
- `Tests.testBase58CheckRoundTripWithFlickrAlphabet()`
- `Tests.testSignTransaction()`

完整 Stage 4 Example 日志 SHA 分别为
`3f9c7999b39b1ddafeb800cff478a2d2a17fa59e8904c910bdd967e5fa8d3e50` 与
`cc474eb863fdad22d3c643995e8d617e93c1abbe1faa71be9563b88bbbaa9921`；
完整 Stage 5 命令/日志/xcresult/SHA 见 Task 5 ledger。

## Rollback Sequence

建议按最新到最旧回滚，且每个命令都限定仓库。先发现并回滚本报告 carrier，再回滚
Task 5 证据与测试：

```bash
CORE_REPO=/Users/viccc/source/tronlink-iOS-core
MAIN_REPO=/Users/viccc/working/4_22_0/TronLink_iOS

git -C "$CORE_REPO" log --format='%H' 03c9916707f980c2a654ab330db17112b265f7b9..HEAD -- docs/migration/07-final-report.md |
while IFS= read -r final_report_commit; do
  test -n "$final_report_commit" || continue
  git -C "$CORE_REPO" revert --no-edit "$final_report_commit"
done

git -C "$CORE_REPO" revert --no-edit 03c9916707f980c2a654ab330db17112b265f7b9
git -C "$MAIN_REPO" revert --no-edit 280145c1e029ccf967453d1e17c4ea506570611a
```

继续撤销生产迁移时，按各阶段报告的 repo-qualified 命令逆序执行。生产边界核心顺序
是 Main `c641762a…` / Core `9e535d3c…`，Main `45b4b601…`、`b87d495e…` /
Core `7f20566c…`，Main `664a1fcb…` / Core `85fd3aac…`，最后 Core
`53da320…`。先回滚消费方，再回滚其提供方；每阶段 docs amendment 由对应报告的
path-limited discovery 命令先行回滚。

用户自有 Main `Podfile.lock`、project 与未跟踪文件不属于任何 rollback commit。
回滚/复验前应确认没有冲突的 staged change。

## Acceptance Checklist

| 验收项 | 状态 | 证据/限制 |
| --- | --- | --- |
| 单一 `TLCore` target/module | PASS | podspec module、smoke、两类 App build |
| 无旧 module import/qualified name | PASS | Main source/tests 0 matches |
| 无旧 standalone pods | PASS | fresh resolved lock/Manifest 0 matches |
| 所有上游 Keystore 断言字面通过 | **不成立** | 25/27；精确两个 printable-ASCII 矛盾断言，未虚报 |
| Keystore 迁移兼容 | PASS | 25 个上游断言 + Main import/scrypt/password gates；两项已裁定 |
| ABI vectors/differential | PASS | Stage 0 golden、Stage 3 dynamic differential、Main routes |
| sign/recover vectors/differential | PASS | 固定 key/hash/signature/address 与 legacy 比对 |
| simulator tests | PASS（按已裁定比较规则） | Example 62/62 others；Main 48/48 others；type 1/1 |
| Main Debug build | PASS | generic simulator |
| Main Release device build | PASS | arm64 static link |
| pod lint | PASS | Xcode 26/iOS 13 temporary xcconfig |
| duplicate C-symbol scan | PASS | secp/ecdsa 55 unique、0 duplicate |
| 旧 framework 产物 | PASS | device build tree 0 |
| 忽略的设计/计划文件未进入 Git history/index | PASS | 最终 `git log --all -- <3 paths>` 与 `git ls-files -- <3 paths>` 均无输出 |
| Main 用户 dirty 文件保持 | PASS | live file SHA、full diff SHA、`cmp` 全部与起始快照一致 |

保留风险：Main 缺少可离线的完整 cold-wallet/multisig、transaction/message signing
端到端 XCTest；现有对应方法依赖 live network、为空或真正 sign 调用被注释。当前
接受依据是 Core 确定性 sign/recover/ABI differential、Main 编译/静态链接与本地 ABI
route 测试。后续若要提高门禁强度，应新增离线 fixture，而不是把这些 live 方法纳入
CI 后宣称稳定。
