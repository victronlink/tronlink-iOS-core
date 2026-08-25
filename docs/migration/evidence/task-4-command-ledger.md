# Task 4 命令与产物台账

本台账记录“单一 TLCore 模块边界收口”的可复核执行证据。时间均为
Singapore time（`+0800`），日期跨越 2026-08-25 至 2026-08-26。

## 证据记录规则

- `mtime` 是现存日志的文件系统修改时间，不反推或冒充命令开始时间。
- Xcode 日志中的 `Command line invocation` 是 Xcode 对实际进程参数的记录。早期
  外层 shell 的 `time`/`tee` 具体书写没有被日志保存时，本台账明确标为不可恢复，
  不重构一个看起来更整洁的 wrapper。
- 退出码来自当时的命令执行结果；测试数来自日志中的 XCTest summary。
- `/tmp` 日志是当前工作机上的审计附件。验收引用文件 SHA-256，不引用会因读取而
  改变的整个 `xcresult` tree hash。
- 两次旧 simulator UDID 复验共用了随后被成功运行覆盖的日志路径。它们的旧内容
  没有存活，故不能提供 SHA；本台账保留可从执行记录确认的命令、退出情况和这一
  证据缺口。

## 固定起点与提交

| 项目 | 值 |
| --- | --- |
| Core 仓库 | `/Users/viccc/source/tronlink-iOS-core` |
| Core Task 4 起点 | `1f2eff7ec420be7d014997aae4c656d2dd65d80a` |
| Core 实现提交 | `9e535d3ca2b70715b0959676eb8b787aa6c12e8f` |
| Core 原始证据提交 | `1ce7c4e2d96fbb5aa40981df0d098235fa2a8104` |
| Main 仓库 | `/Users/viccc/working/4_22_0/TronLink_iOS` |
| Main Task 4 起点 | `45b4b6019de1b93c07eae13ee587d5bd948ecaab` |
| Main 清理提交 | `c641762a5502b28478b555b75c51d14f5cc29f39` |
| Main 本地 core 入口 | `TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core` |

本次文档修订提交不能在自身内容中引用自身 SHA。`05-single-module-cleanup.md` 的
回滚命令以 `1ce7c4e..HEAD` 对两个 Task 4 文档做路径限定发现，因此本次及以后修订
都可确定性找到，无需循环写入提交哈希。

## TDD RED 与 GREEN

### 计划字面断言 RED

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- Xcode 日志自报的实际进程命令：

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-4-red -only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests CODE_SIGNING_ALLOWED=NO
```

- 时间证据：XCTest event `2026-08-25 23:29:15.500`；日志
  `mtime=2026-08-25T23:29:29+0800`。外层 shell 精确开始时间不可恢复。
- exit：65；1 test，1 failure，0 unexpected。实际首字节 `200`，计划值 `65`。
- 输出：`/tmp/task4-smoke-preinstall.log`，35,819 行，11,203,880 bytes，SHA-256
  `99db174eafe0e2099ff19ef43048f6345def7ff4e3464b0092ebebad7ef1efdc`。
- 日志保留 `real 105.35`。历史外层 `time`/redirect/tee wrapper 的字面形式不可恢复。

### 保持生产语义后的 GREEN

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- Xcode 日志自报的实际进程命令：

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-4-red -only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests CODE_SIGNING_ALLOWED=NO
```

- 时间证据：xcresult 名称记录 `2026-08-25_23-29-50_+0800`；日志
  `mtime=2026-08-25T23:30:10+0800`。外层 shell 精确开始时间不可恢复。
- exit：0；1 test，0 failure；日志保留 `real 21.16`。
- 输出：`/tmp/task4-smoke-green.log`，3,964 行，307,787 bytes，SHA-256
  `0d4c2b38a6d8d1a7b3ed32a54b64d665aecf8a82976c19b84ab8f7506bd7c9e2`。

RED 证明计划样例混用了 Web3 20-byte address 和 Tron 21-byte address。最终测试
保持 `Wallet.getKey` 的 20-byte 语义和固定私钥 golden，并用 `KeystoreKey.address`
单独验证 `0x41` 前缀，没有修改生产实现。

## CocoaPods 清理与解析

早期 CocoaPods 命令的可恢复部分是 cwd、可执行命令、退出码、日志、mtime、耗时与
SHA。现存日志没有记录外层 shell wrapper 的字面形式，因此这里不声明 wrapper。

### Example deintegrate

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际可执行命令：

```bash
pod deintegrate
```

- 时间证据：日志 `mtime=2026-08-25T23:30:18+0800`；精确开始时间不可恢复。
- exit：0；日志保留 `real 0.72`。
- 输出：`/tmp/task4-example-pod-deintegrate.log`，760 bytes，SHA-256
  `cf50b7645c6bba65a894ca0c5b3ec10e7c480fcd3d574ceed72648440c6ac0d9`。

### Example install

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际可执行命令：

```bash
pod install
```

- 首次 sandbox 运行因 CocoaPods cache/network 权限非零退出。它使用了下述成功日志
  的同一路径，后来被成功运行覆盖；精确退出值、时间和旧 SHA 已不可恢复。
- 成功运行：exit 0，14 total pods；日志保留 `real 192.76`，
  `mtime=2026-08-25T23:33:43+0800`。
- 成功输出：`/tmp/task4-example-pod-install.log`，964 bytes，SHA-256
  `49079cf51b049043c4bc77feb95a8eddbb569d301c15b2134cf79e9f7fd9631b`。

### Main 第一次 install

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际可执行命令：

```bash
TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core pod install
```

- 时间证据：日志 `mtime=2026-08-25T23:34:09+0800`；精确开始时间不可恢复。
- exit：0；113 total pods；日志保留 `real 12.27`。
- 输出：`/tmp/task4-main-pod-install.log`，2,449 bytes，SHA-256
  `94fa01f7c0ad28788d9075cb655f836f0c80d1433656672a89c91aff362fb36a`。

### 删除旧 target branch 后的 Main install

- cwd：`/Users/viccc/working/4_22_0/TronLink_iOS`
- 实际可执行命令：

```bash
TLCORE_LOCAL_PATH=/Users/viccc/source/tronlink-iOS-core pod install
```

- 时间证据：日志 `mtime=2026-08-25T23:35:07+0800`；精确开始时间不可恢复。
- exit：0；113 total pods；日志保留 `real 11.64`。
- 输出：`/tmp/task4-main-pod-install-final.log`，2,450 bytes，SHA-256
  `8357c9aa687cfaa0bb90ecccd265b8c3bb416eee89cc109faf33464394ed7abf`。

最终解析出的 Main `Podfile.lock` 与 `Pods/Manifest.lock` 被保存到
`/tmp/task4-main-resolved-Podfile.lock` 和
`/tmp/task4-main-resolved-Manifest.lock`；两者各 105,345 bytes，mtime 均为
`2026-08-26T00:35:46+0800`，SHA-256 均为
`1ec58bfc333a9d4175cbaebb5d7035892858e93e9ed2c1b5ddb39e00795648a1`。

## 完整 XCTest、build 与 lint 矩阵

以下每个 Xcode 命令都逐字来自对应日志的 `Command line invocation`。现存日志没有
保存早期外层 shell wrapper 的字面形式；日志末尾的 `real` 值单独记录。

### 未过滤 Example XCTest

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际进程命令：

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-4-tests-full CODE_SIGNING_ALLOWED=NO
```

- 时间证据：日志 `mtime=2026-08-25T23:54:00+0800`；精确开始时间不可恢复。
- exit：65；66 tests，4 failures，0 unexpected，test time 940.803s；`real 1040.86`。
- 输出：`/tmp/task4-example-tests-full.log`，36,009 行，11,438,277 bytes，SHA-256
  `3f9c7999b39b1ddafeb800cff478a2d2a17fa59e8904c910bdd967e5fa8d3e50`。

### 精确跳过四项的 Example XCTest

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际进程命令：

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-4-tests-full -skip-testing:tronlink-iOS-core_Tests/Tests/testBase58CheckRoundTripWithFlickrAlphabet -skip-testing:tronlink-iOS-core_Tests/Tests/testSignTransaction -skip-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests/testKeystoreKeyRejectsMnemonicASCIIPayload -skip-testing:tronlink-iOS-core_Tests/EmbeddedKeystoreTests/testKeystoreKeyRejectsAllPrintableASCIIInput CODE_SIGNING_ALLOWED=NO
```

- 时间证据：日志 `mtime=2026-08-26T00:09:22+0800`；精确开始时间不可恢复。
- exit：0；62 tests，0 failure，test time 887.165s；`real 907.60`。
- 输出：`/tmp/task4-example-tests-skip-four.log`，4,063 行，293,202 bytes，SHA-256
  `cc474eb863fdad22d3c643995e8d617e93c1abbe1faa71be9563b88bbbaa9921`。

### Example Debug simulator build

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际进程命令：

```bash
xcodebuild build -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Example -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tlcore-stage-4-debug CODE_SIGNING_ALLOWED=NO
```

- 时间证据：日志 `mtime=2026-08-26T00:12:19+0800`；精确开始时间不可恢复。
- exit：0，`BUILD SUCCEEDED`；`real 164.47`。
- 输出：`/tmp/task4-example-debug-build.log`，56,599 行，20,138,013 bytes，SHA-256
  `439f7dbd40e1942a941b158d5de32d7b0d1e7207502731546f170fd041450bf8`。

### Example Release simulator build

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际进程命令：

```bash
xcodebuild build -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Example -configuration Release -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tlcore-stage-4 CODE_SIGNING_ALLOWED=NO
```

- 时间证据：日志 `mtime=2026-08-26T00:15:36+0800`；精确开始时间不可恢复。
- exit：0，`BUILD SUCCEEDED`；`real 185.29`。
- 输出：`/tmp/task4-example-release-build.log`，53,713 行，18,958,338 bytes，SHA-256
  `e7a28c317cf953ffbaf1b597d075b3b43ec58121a676a13044c5334b0fc0b559`。

### Pod lint

- cwd：`/Users/viccc/source/tronlink-iOS-core`
- 执行记录保留的实际可执行命令：

```bash
env XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig pod lib lint tronlink-iOS-core.podspec --allow-warnings --skip-tests
```

- 临时 xcconfig：`/tmp/tlcore-lint-xcode26.xcconfig`，内容仅为
  `IPHONEOS_DEPLOYMENT_TARGET = 13.0`，mtime `2026-08-25T18:48:01+0800`，SHA-256
  `e8a40fb39aef577173fd073eb85cdc95a67447360f2453d0ef5a3e7aaa7653ae`。
- 时间证据：日志 `mtime=2026-08-26T00:21:31+0800`；精确开始时间不可恢复。
- exit：0，`tronlink-iOS-core passed validation.`；`real 335.98`。
- 输出：`/tmp/task4-pod-lib-lint.log`，328 行，81,753 bytes，SHA-256
  `da2f0cb51221d5c1116e332733bf978c4d3330c82eb4c902f075013fba4a4f9c`。

### Main Debug simulator build

- cwd：`/Users/viccc/source/tronlink-iOS-core`；workspace 使用绝对路径指向 Main。
- 实际进程命令：

```bash
xcodebuild -workspace /Users/viccc/working/4_22_0/TronLink_iOS/TronLink.xcworkspace -scheme TronLink -configuration Debug -sdk iphonesimulator -destination 'generic/platform=iOS Simulator' -derivedDataPath /tmp/tronlink-tlcore-stage-4-debug CODE_SIGNING_ALLOWED=NO build
```

- 时间证据：日志 `mtime=2026-08-26T00:28:48+0800`；精确开始时间不可恢复。
- exit：0，`BUILD SUCCEEDED`；`real 421.53`。
- 输出：`/tmp/task4-main-debug-build.log`，175,770 行，48,958,875 bytes，SHA-256
  `da0aeaf0eecd67a4e84e9af662dcbdbe576a506f12922f42fd26199bac4c1294`。

### Main Release generic-device unsigned build

- cwd：`/Users/viccc/source/tronlink-iOS-core`；workspace 使用绝对路径指向 Main。
- 实际进程命令：

```bash
xcodebuild -workspace /Users/viccc/working/4_22_0/TronLink_iOS/TronLink.xcworkspace -scheme TronLink -configuration Release -sdk iphoneos -destination generic/platform=iOS -derivedDataPath /tmp/tronlink-tlcore-stage-4-device CODE_SIGNING_ALLOWED=NO build
```

- 时间证据：日志 `mtime=2026-08-26T00:32:52+0800`；精确开始时间不可恢复。
- exit：0，`BUILD SUCCEEDED`；`real 230.90`。这是 static-link/arm64 权威门禁。
- 输出：`/tmp/task4-main-release-device-build.log`，107,149 行，26,030,848 bytes，
  SHA-256 `42a0585cfc0d5fba2c2b3c4205e4239cf2ece18cb1f26372d18a14c748d496fb`。

## 恢复 Example project 后的 focused smoke

### Sandbox 诊断，非验收结果

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际 shell 命令：

```bash
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=B9032202-E5EC-4093-90E1-309A44C030E0' -derivedDataPath /tmp/tlcore-stage-4-tests-full CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests 2>&1 | tee /tmp/task4-smoke-final.log
```

- 时间证据：Xcode 输出 `2026-08-26 00:37:12.049`；精确 shell 开始时间不可恢复。
- shell pipeline exit：0，因为该次命令没有设置 `pipefail`；底层 xcodebuild 在 sandbox
  中未取得 CoreSimulator/home 权限并报 workspace 诊断错误，未编译、未测试。
- 输出路径当时是 `/tmp/task4-smoke-final.log`，随后被两次命令覆盖；旧内容和 SHA
  不可恢复，不作为验收证据。

### 旧 UDID，exit 70

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际 shell 命令：

```bash
set -o pipefail
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=B9032202-E5EC-4093-90E1-309A44C030E0' -derivedDataPath /tmp/tlcore-stage-4-tests-full CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests 2>&1 | tee /tmp/task4-smoke-final.log
```

- 时间证据：Xcode 写 result bundle 的时间为 `2026-08-26 00:38:20.699`；精确 shell
  开始时间不可恢复。
- exit：70；当前 simulator 列表没有该 UDID，未执行测试。
- 输出路径仍是 `/tmp/task4-smoke-final.log`，随后被成功命令覆盖；旧内容和 SHA
  不可恢复，不作为验收证据。

### 当前可用 UDID，最终 GREEN

- cwd：`/Users/viccc/source/tronlink-iOS-core/Example`
- 实际 shell 命令：

```bash
set -o pipefail
xcodebuild test -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=8E7ED5C7-0414-4767-BEEA-F849C083AA2A' -derivedDataPath /tmp/tlcore-stage-4-tests-final CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/SingleTLCorePublicAPITests 2>&1 | tee /tmp/task4-smoke-final.log
```

- 时间证据：xcresult 名称记录 `2026.08.26_00-38-32-+0800`；XCTest event 从
  `00:39:53.294` 到 `00:40:04.742`；日志 `mtime=2026-08-26T00:40:10+0800`。
- exit：0；1 test，0 failure，11.447s。
- 最终输出：`/tmp/task4-smoke-final.log`，35,811 行，11,445,901 bytes，SHA-256
  `92de94dd6a334b027fc3e8150086c4d09de1b0cdd502e53ddff3d3a0827f93f0`。

## 静态边界精确复核

原始汇总 `/tmp/task4-source-module-scans.log` 的 mtime 为
`2026-08-25T23:35:22+0800`，SHA-256
`2c41c7b14dc36b616ec8b8337674801953839a15527bea4991bd7a5f6a13600e`；它保留了
结果但没有保留完整 shell invocation。`/tmp/task4-podspec-contract-scan.log` 同样只
保留结果（mtime `2026-08-25T23:36:23+0800`，SHA-256
`9647048b5851488df7e0f08dcfbc9734295fc8ba5c1e32724c2d9f8fe629f481`），故不伪造
两者的历史 wrapper。

为关闭这一证据缺口，文档修订时从 Core cwd 执行了以下实际只读命令：

```bash
set -o pipefail
{
  printf 'started=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')"
  printf 'podspec_syntax='; ruby -c tronlink-iOS-core.podspec | tr -d '\n'; printf '\n'
  printf 'podspec_dependencies='; rg -n "s\.dependency" tronlink-iOS-core.podspec | wc -l | tr -d ' '
  printf 'legacy_imports='; rg -n '^[[:space:]]*(@testable[[:space:]]+)?import[[:space:]]+(TronCore|TronKeystore|web3swift|tron_wallet_secp256k1|secp256k1)([.]|[[:space:]]|$)' tronlink-iOS-core Example /Users/viccc/working/4_22_0/TronLink_iOS/TronLink /Users/viccc/working/4_22_0/TronLink_iOS/TronLinkTests | wc -l | tr -d ' '
  printf 'checked_in_modulemaps='; find . -name '*.modulemap' -not -path './Example/Pods/*' -print | tee /tmp/task4-review-modulemaps.txt | wc -l | tr -d ' '
  printf 'protobuf_annotations='; rg -n 'NS_SWIFT_NAME\(TronProtoAddress\)' tronlink-iOS-core/Classes/gRPC/api/Api_Tron.pbobjc.h | wc -l | tr -d ' '
  printf 'web3_receiver_hex_collisions='; ruby -e 'ARGV.each { |path| receiver = nil; depth = 0; File.readlines(path).each_with_index { |line, index| if receiver.nil? && (match = line.match(/^\s*(?:public\s+)?extension\s+(Data|String)\b/)); receiver = match[1]; depth = line.count("{") - line.count("}"); next; end; if receiver; puts "#{path}:#{index + 1}:#{receiver}:#{line.strip}" if line.match?(/\bvar\s+hex\s*:\s*(?:Data|String)\b/); depth += line.count("{") - line.count("}"); receiver = nil if depth == 0; end } }' tronlink-iOS-core/Classes/Web3Subset/Support/Data+Extension.swift tronlink-iOS-core/Classes/Web3Subset/Support/String+Extension.swift | wc -l | tr -d ' '
  printf 'swift_public_type_duplicates='; rg -n '^public (final )?(class|struct|enum|protocol|typealias)' tronlink-iOS-core/Classes | sed -E 's/.*(class|struct|enum|protocol|typealias) ([A-Za-z_][A-Za-z0-9_]*).*/\2/' | sort | uniq -d | wc -l | tr -d ' '
  printf 'objc_class_duplicates='; rg -n '^@interface [A-Za-z_][A-Za-z0-9_]*[[:space:]]*:' tronlink-iOS-core/Classes | sed -E 's/.*@interface ([A-Za-z_][A-Za-z0-9_]*)[[:space:]]*:.*/\1/' | sort | uniq -d | wc -l | tr -d ' '
  printf 'gapi_dynamic_categories='; rg -n '^@interface GAPIAnnotationsRoot[[:space:]]*\(DynamicMethods\)' tronlink-iOS-core/Classes | wc -l | tr -d ' '
  printf 'c_symbol_duplicates='; uniq -d docs/migration/manifests/final-c-symbols.txt | wc -l | tr -d ' '
  printf 'public_ecdsa_headers='; rg -x 'ecdsa\.h' docs/migration/manifests/final-public-headers.txt | wc -l | tr -d ' '
  printf 'legacy_main_resolved_lock_matches='; rg -n 'TronWalletABI|TronWalletKeystore|TronWalletWeb3Swift|tron-wallet-secp256k1|PromiseKit|scrypt\.c' /tmp/task4-main-resolved-Podfile.lock /tmp/task4-main-resolved-Manifest.lock | wc -l | tr -d ' '
  printf 'legacy_example_lock_matches='; rg -n 'TronWalletABI|TronWalletKeystore|TronWalletWeb3Swift|tron-wallet-secp256k1|PromiseKit|scrypt\.c' Example/Podfile.lock Example/Pods/Manifest.lock | wc -l | tr -d ' '
  printf 'legacy_main_target_support_dirs='; find /Users/viccc/working/4_22_0/TronLink_iOS/Pods/Target\ Support\ Files -maxdepth 1 -type d | rg 'TronWallet|secp256k1|web3swift' | wc -l | tr -d ' '
  printf 'legacy_example_target_support_dirs='; find Example/Pods/Target\ Support\ Files -maxdepth 1 -type d | rg 'TronWallet|secp256k1|web3swift' | wc -l | tr -d ' '
  printf 'finished=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')"
} 2>&1 | tee /tmp/task4-review-boundary-gates.log
review_gate_exit=${pipestatus[1]}
printf 'exit=%s\n' "$review_gate_exit" | tee -a /tmp/task4-review-boundary-gates.log
exit "$review_gate_exit"
```

- cwd：`/Users/viccc/source/tronlink-iOS-core`
- 日志自带时间：`2026-08-26T00:53:07+0800`；exit 0。
- 输出：`/tmp/task4-review-boundary-gates.log`，480 bytes，SHA-256
  `fd84ee1dc378c8233e4e30a3e1778a81c0f24b0f64cdd35dcfa3141d4417fd6e`。
- 结果：podspec syntax OK；dependencies 9；旧 import 0；checked-in modulemap 1；
  protobuf annotation 1；receiver-aware hex collision 0；Swift type duplicate 0；真实
  Objective-C class duplicate 0；合法 `GAPIAnnotationsRoot (DynamicMethods)` category 1；
  C symbol duplicate 0；public `ecdsa.h` 1；两仓旧 resolved dependency 和 target dir 0。

## 产物、架构与符号精确复核

原始产物路径日志 `/tmp/task4-artifact-path-scan.log` 的 mtime 为
`2026-08-26T00:33:11+0800`，SHA-256
`228e6bfe79a46c94eb7ff16d475a2bc4d7232222b6864fbb791585b3273f8a98`；原始架构
日志 `/tmp/task4-artifact-architectures.log` 的 mtime 为
`2026-08-26T00:33:23+0800`，SHA-256
`6ebaff065ad1ce382412a5bad11cdd29df3474f7efc35d1ed444f9803e0b0052`。两者未保存
完整历史 wrapper，因此以下列修订时实际执行的完整只读命令补足：

```bash
set -o pipefail
release_tlcore=/tmp/tlcore-stage-4/Build/Products/Release-iphonesimulator/tronlink-iOS-core/TLCore.framework/TLCore
device_tlcore=/tmp/tronlink-tlcore-stage-4-device/Build/Products/Release-iphoneos/tronlink-iOS-core/TLCore.framework/TLCore
debug_app=/tmp/tronlink-tlcore-stage-4-debug/Build/Products/Debug-iphonesimulator/TronLink.app/TronLink
device_app=/tmp/tronlink-tlcore-stage-4-device/Build/Products/Release-iphoneos/TronLink.app/TronLink
{
  printf 'started=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')"
  printf 'tlcore_module_artifacts='; find /tmp/tlcore-stage-4 -type f -path '*/TLCore.framework/Modules/*' -print | sort | tee /tmp/task4-review-module-artifacts.txt | wc -l | tr -d ' '
  printf 'legacy_frameworks='; find /tmp/tronlink-tlcore-stage-4-debug /tmp/tronlink-tlcore-stage-4-device -type d \( -name 'TronCore.framework' -o -name 'TronKeystore.framework' -o -name 'web3swift.framework' -o -name 'tron_wallet_secp256k1.framework' \) -print | tee /tmp/task4-review-legacy-frameworks.txt | wc -l | tr -d ' '
  printf 'release_tlcore_archs='; lipo -archs "$release_tlcore"
  printf 'device_tlcore_archs='; lipo -archs "$device_tlcore"
  printf 'debug_app_archs='; lipo -archs "$debug_app"
  printf 'device_app_archs='; lipo -archs "$device_app"
  file "$release_tlcore" "$device_tlcore"
  printf 'device_tlcore_defined_symbols='; nm -gjU "$device_tlcore" | sort | wc -l | tr -d ' '
  printf 'device_tlcore_secp_ecdsa_symbols='; nm -gjU "$device_tlcore" | sort | rg '^_(secp256k1_|ecdsa_)' | wc -l | tr -d ' '
  printf 'device_tlcore_secp_ecdsa_duplicates='; nm -gjU "$device_tlcore" | sort | rg '^_(secp256k1_|ecdsa_)' | uniq -d | wc -l | tr -d ' '
  printf 'device_app_secp_ecdsa_symbols='; nm -gjU "$device_app" | sort | rg '^_(secp256k1_|ecdsa_)' | wc -l | tr -d ' '
  shasum -a 256 "$release_tlcore" "$device_tlcore" "$debug_app" "$device_app"
  printf 'finished=%s\n' "$(date '+%Y-%m-%dT%H:%M:%S%z')"
} 2>&1 | tee /tmp/task4-review-artifact-gates.log
review_artifact_exit=${pipestatus[1]}
printf 'exit=%s\n' "$review_artifact_exit" | tee -a /tmp/task4-review-artifact-gates.log
exit "$review_artifact_exit"
```

- cwd：`/Users/viccc/source/tronlink-iOS-core`
- 日志自带时间：`2026-08-26T00:53:53+0800` 至 `00:53:54+0800`；exit 0。
- 输出：`/tmp/task4-review-artifact-gates.log`，1,756 bytes，SHA-256
  `14775a98bf5d1fcf66ebcd6c27b82b6dc7af3a89c8bc6ed6f64f8f7e8b6d6fae`。
- module artifact 明细：`/tmp/task4-review-module-artifacts.txt`，9 行，SHA-256
  `501f735ca2fea39459d707ec691ae7b8b7a14cebb1b0ebd71ebd92160ea3fa4c`。
- legacy framework 明细为空：`/tmp/task4-review-legacy-frameworks.txt`，SHA-256
  `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855`。
- 结果：Release TLCore `x86_64 arm64` dynamic；device TLCore `arm64` static archive；
  Main Debug `x86_64 arm64`；Main Device `arm64`；defined symbols 11,809；匹配
  `^_(secp256k1_|ecdsa_)` 的 symbols 55、重复 0；Main Device app 匹配导出 0。

在最终复核前有一次分类诊断使用了实际命令
`nm -gjU "$device_tlcore" | sort | rg '^_(secp256k1|ecdsa_)' | wc -l | tr -d ' '`，
它把不属于 `secp256k1_*` wildcard 的 bare `_secp256k1` 也计入，结果为 56。该次
`/tmp/task4-review-artifact-gates.log` 随后被最终复核覆盖；覆盖前 SHA-256 为
`d900a97ac0fbf913dbdf7f1cd8f7f17a021150b13eace58dd8b526b0c2db09b9`，执行记录时间
为 `2026-08-26T00:53:35+0800`，exit 0。最终门禁使用上文完整命令中的
`^_(secp256k1_|ecdsa_)`，与 `secp256k1_*`/`ecdsa_*` 约束一致。

## 其他存活的原始辅助证据

下列文件来自原始 Task 4 执行。其历史完整 wrapper 没有写入文件；表中只记录可直接
复核的 mtime、内容和 SHA，不以它们替代上文带完整命令的修订复核。

| 文件 | mtime | 内容/行数 | SHA-256 |
| --- | --- | --- | --- |
| `/tmp/task4-final-resolved-graph-gates.log` | `2026-08-26T00:36:27+0800` | 四项 resolved graph/target dir count 均为 0；132 bytes | `d9f82af57fd75fb3c4a8da01b7ed2cdf7e178a13efbc71b7ee563dd2e5e6604f` |
| `/tmp/task4-example-old-deps-scan.log` | `2026-08-25T23:34:22+0800` | 空结果，0 bytes | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `/tmp/task4-main-old-deps-scan.log` | `2026-08-25T23:34:22+0800` | 空结果，0 bytes | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `/tmp/task4-receiver-aware-web3-hex-scan.log` | `2026-08-25T23:36:03+0800` | 空结果，0 bytes | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `/tmp/task4-device-tlcore-defined-symbols.txt` | `2026-08-26T00:33:24+0800` | 11,809 行 | `83dc32ad1ef2f594d2dc0ab449d8d436677d4ae0810ad9060acda56b3606a66a` |
| `/tmp/task4-device-tlcore-secp-ecdsa-symbols.txt` | `2026-08-26T00:33:24+0800` | 55 行 | `2ffb61709a9739f5648464caa5ddd6fcf66ce3a1b3b480a5f99e676b065dd3cd` |
| `/tmp/task4-device-tlcore-secp-ecdsa-duplicates.txt` | `2026-08-26T00:33:24+0800` | 空结果 | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `/tmp/task4-objc-class-duplicates.log` | `2026-08-26T00:36:27+0800` | 空结果 | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `/tmp/task4-objc-category-declarations.log` | `2026-08-26T00:36:27+0800` | 1 行合法 category | `93835580f4d89d54878f252aad73343196c602cad8b357a4e33ec30501b6f7a9` |
| `/tmp/task4-duplicate-swift-types-final.log` | `2026-08-26T00:37:04+0800` | 空结果 | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |
| `/tmp/task4-duplicate-c-symbols-final.log` | `2026-08-26T00:37:04+0800` | 空结果 | `e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855` |

## Manifest 的实际生成与交叉验证

Core cwd 中实际使用的生成命令是：

```bash
find tronlink-iOS-core/Classes -type f -print0 | sort -z | xargs -0 shasum -a 256 > docs/migration/manifests/final-tlcore-sources.sha256
rg -n '^public (final )?(class|struct|enum|protocol|typealias)|^public extension ' tronlink-iOS-core/Classes | sort > docs/migration/manifests/final-public-types.txt
TLCORE_HEADERS_DIR="$(find /tmp/tlcore-stage-4 -type d -path '*/TLCore.framework/Headers' -print -quit)"
test -n "${TLCORE_HEADERS_DIR}"
find "${TLCORE_HEADERS_DIR}" -type f -exec basename {} \; | sort > docs/migration/manifests/final-public-headers.txt
TLCORE_BINARY_PATH="$(find /tmp/tlcore-stage-4 -type f -name TLCore -perm -111 -print -quit)"
test -n "${TLCORE_BINARY_PATH}"
nm -gjU "${TLCORE_BINARY_PATH}" | sort > docs/migration/manifests/final-c-symbols.txt
```

历史 shell 开始时间未写入输出；整组命令 exit 0。实际 `find -print -quit` 选中同一
Release build 的 arm64 intermediate binary：
`/tmp/tlcore-stage-4/Build/Intermediates.noindex/Pods.build/Release-iphonesimulator/tronlink-iOS-core.build/Objects-normal/arm64/Binary/TLCore`。

随后实际执行：

```bash
nm -gjU /tmp/tlcore-stage-4/Build/Products/Release-iphonesimulator/tronlink-iOS-core/TLCore.framework/TLCore | sort > /tmp/task4-release-fat-symbols.raw
cmp docs/migration/manifests/final-c-symbols.txt /tmp/task4-release-fat-symbols.raw
```

`cmp` exit 0。交叉验证文件 mtime `2026-08-26T00:33:49+0800`，2,714 行、111,337
bytes，SHA-256 `31c068887aeee90e2679a122b1aa9516b4dc526cd957c732e12775981633404c`。

| Manifest | 行数 | SHA-256 |
| --- | ---: | --- |
| `docs/migration/manifests/final-tlcore-sources.sha256` | 260 | `09bdc096f6b6248b83b257a249d2e7d77fa2d2a8ffdcd9ecd876bd53dc465de3` |
| `docs/migration/manifests/final-public-types.txt` | 62 | `e3fd911583ffdc75b74a81f334ae9ec1110a204f6d432744ed348ed52a2b31dc` |
| `docs/migration/manifests/final-public-headers.txt` | 71 | `cbe4b814007babc3c4550cbc22a457a2655fa5e2680df4e09c30832f81ca2731` |
| `docs/migration/manifests/final-c-symbols.txt` | 2,714 | `31c068887aeee90e2679a122b1aa9516b4dc526cd957c732e12775981633404c` |

## 用户文件恢复与提交

Main 用户文件的实际恢复命令：

```bash
cp /tmp/task4-main-start-Podfile.lock /Users/viccc/working/4_22_0/TronLink_iOS/Podfile.lock
cp /tmp/task4-main-start-project.pbxproj /Users/viccc/working/4_22_0/TronLink_iOS/TronLink.xcodeproj/project.pbxproj
```

随后实际执行逐文件 diff 和 `cmp -s`。两项 `cmp` 均为 0：

| 文件 | 起始/最终文件 SHA-256 | 起始/最终 diff SHA-256 |
| --- | --- | --- |
| `Podfile.lock` | `3513cae6cf97a837f66a04240feb143fbc826701786f9ceb93652bfcf9921c65` | `af0938ab42ec83a2ec8915d4aeb7c5efbb7a483e842e4070c4b9fa6df3c32896` |
| `TronLink.xcodeproj/project.pbxproj` | `739e70443445ad475c46dc5bfc49a688dbde923657b1221ce29a9a56ae3b12f8` | `d015ab9f57155ccf26b30347515440ac0a7d81a5b8009150a13f0d14ea17fa60` |

Example project 的随机 CocoaPods reference/build-phase ID churn 使用下列实际命令恢复：

```bash
git restore -- Example/tronlink-iOS-core.xcodeproj/project.pbxproj
```

恢复后执行的最终 focused smoke 已在前文完整记录并通过。

实际提交命令：

```bash
git add Example/Tests/Tests.swift docs/migration/manifests/final-tlcore-sources.sha256 docs/migration/manifests/final-public-types.txt docs/migration/manifests/final-public-headers.txt docs/migration/manifests/final-c-symbols.txt
git diff --cached --stat
git diff --cached --check
git diff --cached --name-only
git commit -m "chore: enforce the single TLCore module boundary"
```

Core 实现提交为 `9e535d3ca2b70715b0959676eb8b787aa6c12e8f`，只含 5 个列出的
白名单文件。

Main cwd 中实际执行：

```bash
git add Podfile
git diff --cached --stat
git diff --cached --check
git diff --cached --name-only
git diff --cached -- Podfile
git commit -m "chore: remove legacy TLCore transitive pods"
```

Main 提交为 `c641762a5502b28478b555b75c51d14f5cc29f39`，只含 `Podfile` 的 9 行
删除。用户的 lock/project、GasFree reorder 和既有 untracked 文件均未暂存。
