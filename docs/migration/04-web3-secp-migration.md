# Web3 and secp256k1 migration

## Scope and commits

Task 3 consolidates the wallet's required Web3Swift and libsecp256k1 surface
inside TLCore. The source inputs were fixed to these clean repositories:

- Web3Swift: `2b24ba4e65a3cf2026d697396a95ba7e1937e325`
- secp256k1: `5d74ae264f59f5b98c5832f79cc33c2ed9bad82d`

The implementation moved through these repository boundaries:

| Repository | Before | After | Commit |
| --- | --- | --- | --- |
| TLCore | `c84aa8d5d4315001fe530357a63c3b1bc0ef0af9` | `7f20566c229c82c6abe731f0e941addcbcdaf17a` | `refactor: embed the required Web3 and secp256k1 APIs` |
| Main app | `664a1fcb132c7ab3d2e0e8c5428395a4f73d4a1b` | `b87d495eaac377a9ade9f3b6baa2710696a76b85` | `refactor: consume Web3 APIs from TLCore` |
| Main app local-source gate | `b87d495eaac377a9ade9f3b6baa2710696a76b85` | `45b4b6019de1b93c07eae13ee587d5bd948ecaab` | `fix: require local TLCore checkout for migration` |

The main commit contains `Podfile`, 11 source files, and only the project-file
hunks that remove obsolete PromiseKit, scrypt, and web3swift header/link
settings. The pre-existing GasFree project-file reorder remains unstaged.
`Podfile.lock` also remains unstaged because it had a user-owned pre-task diff
that could not be separated safely. The local validation lock resolved TLCore
through `/Users/viccc/source/tronlink-iOS-core`; it is evidence, not a commit
input.

The 11 final main-app source paths migrated from the old Web3 modules to
`TLCore` are:

```text
TronLink/TRXModule/ColdWallet/TRXColdWalletViewController.swift
TronLink/TRXModule/DAPP/ViewController/TRXDAppViewController+Confirm.swift
TronLink/TRXModule/HomePage/DeepLinkTranscationManager.swift
TronLink/TRXModule/MultiSig/ViewModel/TRXMultiSigAddPermissionViewModel.swift
TronLink/TRXModule/TestFeature/TRXTestFeatureListViewController.swift
TronLink/TRXModule/Transfer/NormalTransfer/TronLinkTransferSecondViewController+GasFreeConfirm.swift
TronLink/WalletHelp/TRXSensitiveConfigManager.swift
TronLink/WalletHelp/TronLinkGRPCApiManager.swift
TronLink/WalletHelp/TronLinkHelper+SignBroadcast.swift
TronLink/WalletHelp/TronLinkWalletCore.swift
TronLink/WalletHelp/route/TronLinkEncoder.swift
```

The main Podfile now has no published-version fallback. It requires a nonempty
`TLCORE_LOCAL_PATH`, verifies that it resolves to a directory, expands it, and
passes it to CocoaPods as a local path:

```bash
TLCORE_LOCAL_PATH=/path/to/tronlink-iOS-core pod install
```

Missing, blank, and nonexistent paths fail while evaluating the Podfile. This
prevents a migration checkout from silently resolving the published TLCore
1.0.7 API instead of the implementation under review.

## Imported source boundary

All 45 paths under `tronlink-iOS-core/Classes/Secp256k1` match the pinned
source byte-for-byte. The only path adaptation is
`secp256k1.c -> wallet_secp256k1.c`, which prevents a basename/object collision
with TrezorCrypto. Both files have SHA-256
`335ffe244402cd7b228a38cad9b43f7c7a3da511c36aecaacb5206996def154d`.

The committed provenance manifest below gives every destination, its path at
the pinned secp256k1 revision, and the SHA-256 shared by both files:

```text
tronlink-iOS-core/Classes/Secp256k1/basic-config.h | secp256k1/basic-config.h | ea2634b9bcacadf2ed2ce4192c3fc4d2ba83d565742f1105732e4983f8f3ddfc
tronlink-iOS-core/Classes/Secp256k1/ecdh_impl.h | secp256k1/ecdh_impl.h | d8a01061a17c4d67f9b5a727c10c1bef9ab1cbe8c168454dbeac269d3554f7a8
tronlink-iOS-core/Classes/Secp256k1/ecdsa.h | secp256k1/ecdsa.h | 151f9f1a38c5051f1d06007a772909d356e5ec2e8a3d4b999ec25db9c163f925
tronlink-iOS-core/Classes/Secp256k1/ecdsa_impl.h | secp256k1/ecdsa_impl.h | 4b0e3b37a9f01599bc46deaf6a4b3bb7fd470b20e17e756e965be3f657010375
tronlink-iOS-core/Classes/Secp256k1/eckey.h | secp256k1/eckey.h | 705ca275a9e6534b356c6d9da82e6bf2a2d4f4c79f2841f85be1c0faceea730e
tronlink-iOS-core/Classes/Secp256k1/eckey_impl.h | secp256k1/eckey_impl.h | 46d76e37f92bfe04e4ded8fed44644ded32c4fe9894ff9acb36f4ba6d52d74b1
tronlink-iOS-core/Classes/Secp256k1/ecmult.h | secp256k1/ecmult.h | 0261228190bcdfce2deb16544b8bca5647e8c43dcf4cddd192dcccdcd746feed
tronlink-iOS-core/Classes/Secp256k1/ecmult_const.h | secp256k1/ecmult_const.h | 9d541941482b871a6fa193be1d39bc653ca193cf92346e33637962753f082a6f
tronlink-iOS-core/Classes/Secp256k1/ecmult_const_impl.h | secp256k1/ecmult_const_impl.h | cdb2e0c212f7016634ec6211b1d0a2323ee800637366492e9961b8b899239273
tronlink-iOS-core/Classes/Secp256k1/ecmult_gen.h | secp256k1/ecmult_gen.h | 80074e92391898d0e6d2ae328b673dfeb17502a6b971a3ff1ab6eadd9ead7341
tronlink-iOS-core/Classes/Secp256k1/ecmult_gen_impl.h | secp256k1/ecmult_gen_impl.h | 9b03e298b640ddab0c8a3d761f54843c293a9a2a488c9bd9fe08eb1dd8804aba
tronlink-iOS-core/Classes/Secp256k1/ecmult_impl.h | secp256k1/ecmult_impl.h | e621364350dc81eb94d204c0627d19bcfda536b0678fed073311cac69ff929bb
tronlink-iOS-core/Classes/Secp256k1/field.h | secp256k1/field.h | ea0e2bfeb6c7077ee83a92895d12f5ac4a8c1b90d3bcb0c94a424a15e183a671
tronlink-iOS-core/Classes/Secp256k1/field_10x26.h | secp256k1/field_10x26.h | e34c3daea1e819a8d8b6fa5b62d96163a54c5abc184acce378913b8753ec5c43
tronlink-iOS-core/Classes/Secp256k1/field_10x26_impl.h | secp256k1/field_10x26_impl.h | 05ffa372c3f76c9c7df410faa24d6cba91b75b00f9c0491ceba8e8edfd893e2e
tronlink-iOS-core/Classes/Secp256k1/field_5x52.h | secp256k1/field_5x52.h | 1d0e538ff36e19968b14833472690b863ca64abf3f5763682ad6e59245fcc275
tronlink-iOS-core/Classes/Secp256k1/field_5x52_asm_impl.h | secp256k1/field_5x52_asm_impl.h | 12bc60d30f41be9b186cb6744405434a241abd50f2499d0a46de351212c7cfaf
tronlink-iOS-core/Classes/Secp256k1/field_5x52_impl.h | secp256k1/field_5x52_impl.h | 12aa09cfb6b3dc3e97a4823c7da6a66d1dee02a610851e2c26a601df86aed0f3
tronlink-iOS-core/Classes/Secp256k1/field_5x52_int128_impl.h | secp256k1/field_5x52_int128_impl.h | a604d261b73ea43e0b664fada56b4407e454426b002311a19357aa217fb63058
tronlink-iOS-core/Classes/Secp256k1/field_impl.h | secp256k1/field_impl.h | 51a7bfea38ba587ab2922219d96145a168bc60b9e2ee0e031aa4c0dd34ac9556
tronlink-iOS-core/Classes/Secp256k1/group.h | secp256k1/group.h | 7f445701eaaa65c395184afb8bff198cedf2b375b711fb624190ee25ded6150c
tronlink-iOS-core/Classes/Secp256k1/group_impl.h | secp256k1/group_impl.h | 4f03e7741c6d483118f2b3d8792c8aa85d0c6e8770cc0eb2486c4eb8203ebeb9
tronlink-iOS-core/Classes/Secp256k1/hash.h | secp256k1/hash.h | 4961777743ffc65f2f7dfc6095073a64c53e84f4769aebe7238231d65490d1af
tronlink-iOS-core/Classes/Secp256k1/hash_impl.h | secp256k1/hash_impl.h | 803601d5f82ab52254932086e0af0eef08d80cbb0cc4b227c8af777ff4e6f13b
tronlink-iOS-core/Classes/Secp256k1/include/secp256k1-wallet.h | secp256k1/include/secp256k1-wallet.h | 26f415afc74b2dc1a670f0354eeef6407b86991b220a977a25a163b1ae88b97d
tronlink-iOS-core/Classes/Secp256k1/num.h | secp256k1/num.h | d9d66c21408892c9c1db45548c4a0cf43a0c3499f5daadc9dea52b4f42e6f266
tronlink-iOS-core/Classes/Secp256k1/num_gmp.h | secp256k1/num_gmp.h | af79122e2a2738fb2b044eaf4934e9443265c05672ffdf9d1aafda1b0fdacfb7
tronlink-iOS-core/Classes/Secp256k1/num_gmp_impl.h | secp256k1/num_gmp_impl.h | 41f2d8f30d0e44347be8ec39c80722d6b477c8f93c7cf38bf727570791d8c1c4
tronlink-iOS-core/Classes/Secp256k1/num_impl.h | secp256k1/num_impl.h | 42a5c9cb11d0595d6b5cba9652797f326bb94ff376f7ae7353bfc13d57847653
tronlink-iOS-core/Classes/Secp256k1/recovery_impl.h | secp256k1/recovery_impl.h | 9b39883cc89f9dad24b49efc085fb0ecbfea977ea67be743d00a1ad94b899f92
tronlink-iOS-core/Classes/Secp256k1/scalar.h | secp256k1/scalar.h | c8c3c06f3e18f5b93298cb2395250725be4c312c510dfed72722b77599f6ed0d
tronlink-iOS-core/Classes/Secp256k1/scalar_4x64.h | secp256k1/scalar_4x64.h | 0b54445ba25bc5e0893bc42f65784e5442a28d9a24aa8728134ff45c87629764
tronlink-iOS-core/Classes/Secp256k1/scalar_4x64_impl.h | secp256k1/scalar_4x64_impl.h | 1c7310bad8e65d4842855b62602c975e0c334f4c0edc4b6b9e2757e358209b39
tronlink-iOS-core/Classes/Secp256k1/scalar_8x32.h | secp256k1/scalar_8x32.h | c89ea3b63eac13aa26bf2088914c0f2fd7df1c7eb775ba0ca2fcf854010985eb
tronlink-iOS-core/Classes/Secp256k1/scalar_8x32_impl.h | secp256k1/scalar_8x32_impl.h | 03ec0ca66f41303f40f39332eff7647d18b46b45cc3d23b8b0db531a478b8341
tronlink-iOS-core/Classes/Secp256k1/scalar_impl.h | secp256k1/scalar_impl.h | 8498e8ce9d78d1d823f2772131b63033541f16208d7d1e6a359978e9ee6bded3
tronlink-iOS-core/Classes/Secp256k1/scalar_low.h | secp256k1/scalar_low.h | c037d026f9dbb419a7e9399814e914d8eb9e19980cb23b89e17dc9349d4027e8
tronlink-iOS-core/Classes/Secp256k1/scalar_low_impl.h | secp256k1/scalar_low_impl.h | 11c4ca226674b877d1d56fd1a84eaffe1b8ba0999b9409de169331d0e2612c55
tronlink-iOS-core/Classes/Secp256k1/scratch.h | secp256k1/scratch.h | 1fc228d92d345046d57f5360ac41c9255939dfb253a3ed48d0e409d89f486d87
tronlink-iOS-core/Classes/Secp256k1/scratch_impl.h | secp256k1/scratch_impl.h | 80822f92a0904c768a5b96a17c5f1cf0576a5e89092fa9b19b9eed25409cb446
tronlink-iOS-core/Classes/Secp256k1/secp256k1-config.h | secp256k1/secp256k1-config.h | af1ead439f6ac7c8da11f73cdb0a003e046b65e89be62bdb922d80792c78f9f4
tronlink-iOS-core/Classes/Secp256k1/secp256k1_ec_mult_static_context.h | secp256k1/secp256k1_ec_mult_static_context.h | 0c7a61fbeaa300ae58ecde1d1b170fa78551fe7143cc40a0770de8aff8cc230b
tronlink-iOS-core/Classes/Secp256k1/secp256k1_main.h | secp256k1/secp256k1_main.h | 4361bf120a620f5789b144e3ea93e405362f8b2ce0ad1824b857d7bced9b5fcd
tronlink-iOS-core/Classes/Secp256k1/util.h | secp256k1/util.h | 67a5328a40eeb3158a2e8257d815e49f991d0db0909c6cc99c82d664ace33f00
tronlink-iOS-core/Classes/Secp256k1/wallet_secp256k1.c | secp256k1/secp256k1.c | 335ffe244402cd7b228a38cad9b43f7c7a3da511c36aecaacb5206996def154d
```

The exact Web3 subset is 21 files: 19 pinned whitelist inputs plus two focused
compatibility files.

```text
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Decoding.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Elements.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Encoding.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2ParameterTypes.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Parsing.swift
tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2TypeParser.swift
tronlink-iOS-core/Classes/Web3Subset/Crypto/LibSecp256k1Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Crypto/PrivateKey.swift
tronlink-iOS-core/Classes/Web3Subset/Crypto/keccak.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Array+Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/CryptoExtensions.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Data+Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Int+Sequence.swift
tronlink-iOS-core/Classes/Web3Subset/Support/NSRegularExpressionExtension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/NativeTypesEncoding+Extensions.swift
tronlink-iOS-core/Classes/Web3Subset/Support/String+Extension.swift
tronlink-iOS-core/Classes/Web3Subset/Support/UInt256.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Web3Address.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Web3CompatibilityTypes.swift
tronlink-iOS-core/Classes/Web3Subset/Support/Web3Utils.swift
```

The two licenses are exact copies. Their SHA-256 values are
`9e886690e7511028efff4d42b2f4bf1efa4730252dbfb9a422fb8f311b845bfd`
for Web3Swift and
`ea3c96f6e9b90d277a8577868e8063eed03340b05d725cad61b353a16acbbe5f`
for secp256k1.

The reproducible literal bulk-import command is below. It copies the complete
45-file secp tree, renames only the implementation basename, copies exactly the
19 Web3 whitelist inputs, and copies both licenses. Subsequent source
adaptations are described separately below.

```bash
TLCORE_ROOT=/path/to/tronlink-iOS-core
WEB3_PIN=/path/to/tron-wallet-web3swift
SECP_PIN=/path/to/tron-wallet-secp256k1

mkdir -p "$TLCORE_ROOT/tronlink-iOS-core/Classes/Secp256k1"
mkdir -p "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI"
mkdir -p "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Crypto"
mkdir -p "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support"
mkdir -p "$TLCORE_ROOT/ThirdPartyLicenses"
cp -R "$SECP_PIN/secp256k1/." "$TLCORE_ROOT/tronlink-iOS-core/Classes/Secp256k1/"
mv "$TLCORE_ROOT/tronlink-iOS-core/Classes/Secp256k1/secp256k1.c" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Secp256k1/wallet_secp256k1.c"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2Decoding.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Decoding.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2Elements.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Elements.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2Encoding.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Encoding.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2ParameterTypes.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2ParameterTypes.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2Parsing.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2Parsing.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/ABIv2/ABIv2TypeParser.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/ABI/ABIv2TypeParser.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/LibSecp256k1Extension.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Crypto/LibSecp256k1Extension.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Encryption/PrivateKey.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Crypto/PrivateKey.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Encryption/keccak.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Crypto/keccak.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/Array+Extension.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/Array+Extension.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/CryptoExtensions.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/CryptoExtensions.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/Data+Extension.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/Data+Extension.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/Int+Sequence.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/Int+Sequence.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/NSRegularExpressionExtension.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/NSRegularExpressionExtension.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/NativeTypesEncoding+Extensions.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/NativeTypesEncoding+Extensions.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/String+Extension.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/String+Extension.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/Convenience/UInt256.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/UInt256.swift"
cp "$WEB3_PIN/TronWalletWeb3Swift/Classes/KeystoreManager/EthereumAddress.swift" "$TLCORE_ROOT/tronlink-iOS-core/Classes/Web3Subset/Support/Web3Address.swift"
cp "$WEB3_PIN/LICENSE" "$TLCORE_ROOT/ThirdPartyLicenses/TronWalletWeb3Swift-LICENSE"
cp "$SECP_PIN/License.md" "$TLCORE_ROOT/ThirdPartyLicenses/tron-wallet-secp256k1-LICENSE"
```

### Explicit exclusions

The subset does not import Base58, RIPEMD160, AES/Cryptor, DerivedKey, legacy
scrypt, Web3 keystores, RLP/transaction implementations, contracts, providers,
networking, Objective-C bridges, Ethereum APIs, EIPs, ERC implementations, or
PromiseKit. TLCore already owns the required address, encoding, hashing, and
keystore primitives; importing those families would duplicate behavior or
reintroduce the dependencies this stage removes.

The exclusion scan has no executable-code matches. Three textual matches are
upstream comments/examples only: two ERC20 documentation lines in
`Web3Address.swift` and one ERC20 compatibility comment in
`ABIv2Decoding.swift`.

## Source adaptations

- Removed `import tron_wallet_secp256k1`; the TLCore umbrella exposes
  `secp256k1-wallet.h` from the embedded tree.
- Renamed Web3Swift's `Address`/`AddressError` surface to
  `Web3Address`/`Web3AddressError`, leaving TLCore's existing `Address` intact.
- Removed the copied `Data.hex: String` and `String.hex: Data` declarations;
  callers use the existing TLCore helpers with the same required lowercase and
  odd-nibble behavior.
- Adapted `Data.fromHex` and `String.dataFromHex()` to the existing TLCore hex
  initializer while preserving legacy empty-data rejection.
- Added only the ABIv2-reachable `Web3Error`, `EventLog`, Codable
  `Web3Address`, approved `Web3Utils` methods, and exact upstream `Web3Units`.
- Pruned unused legacy scrypt/PBKDF and Cryptor-backed digest tails.
  `Web3Utils.sha256` delegates to the existing CryptoSwift-backed
  `Data.sha256T()`.
- Removed copied `Data.bytes` after the main test target proved it collided
  with `CryptoSwift.Data.bytes`. Four internal Web3 call sites now use explicit
  `Array(data)` conversions. A whole copied-Data/String member scan has no
  remaining CryptoSwift intersection.
- `TLWalletCore` now calls `Web3Utils.hashECRecover` directly.

The main migration applies only the approved module/type mappings. The initial
module-import scan found 10 files. A subsequent test-target compile exposed one
additional indirect dependency on Web3Swift's global Objective-C
`NSData.hexString()` bridge. That 11th file now imports TLCore and uses
`Data(zeroArray).hexString`, which preserves lowercase output.

## Dynamic differential fixture and golden inputs

The pre-removal dynamic differential was this complete three-test fixture. It
was intentionally temporary because the old module had to be present for both
implementations to execute in one process. The invalid tuple spelling is
asserted as a rejection in both modules; the valid nested array is compared by
structure rather than module-qualified debug descriptions.

```swift
import BigInt
import TLCore
import XCTest
import web3swift

final class EmbeddedWeb3DifferentialTests: XCTestCase {
    private let privateKeyData = Data(repeating: 0, count: 31) + Data([1])
    private let messageHash = Data(repeating: 0x11, count: 32)

    func testPrivateKeySignAndRecoverMatchesLegacyWeb3() throws {
        let legacy = web3swift.PrivateKey(privateKeyData)
        let embedded = TLCore.PrivateKey(privateKeyData)
        try legacy.verify()
        try embedded.verify()
        XCTAssertEqual(embedded.publicKey, legacy.publicKey)
        XCTAssertEqual(embedded.address.address, legacy.address.address)

        let legacySignature = try legacy.sign(hash: messageHash)
        let embeddedSignature = try embedded.sign(hash: messageHash)
        XCTAssertEqual(embeddedSignature.data, legacySignature.data)
        XCTAssertEqual(
            try TLCore.Web3Utils.hashECRecover(hash: messageHash, signature: embeddedSignature.data).address,
            try web3swift.Web3Utils.hashECRecover(hash: messageHash, signature: legacySignature.data).address
        )
    }

    func testABIv2EncodingAndDecodingMatchesLegacyWeb3() throws {
        XCTAssertThrowsError(try web3swift.ABIv2TypeParser.parseTypeString("(address,uint256[])[]"))
        XCTAssertThrowsError(try TLCore.ABIv2TypeParser.parseTypeString("(address,uint256[])[]"))

        let legacyType = try web3swift.ABIv2TypeParser.parseTypeString("uint256[][2]")
        let embeddedType = try TLCore.ABIv2TypeParser.parseTypeString("uint256[][2]")
        guard case let .array(type: legacyInner, length: legacyOuterLength) = legacyType,
              case let .array(type: legacyLeaf, length: legacyInnerLength) = legacyInner,
              case let .uint(bits: legacyBits) = legacyLeaf,
              case let .array(type: embeddedInner, length: embeddedOuterLength) = embeddedType,
              case let .array(type: embeddedLeaf, length: embeddedInnerLength) = embeddedInner,
              case let .uint(bits: embeddedBits) = embeddedLeaf else {
            return XCTFail("Parsers did not produce the expected nested array structure")
        }
        XCTAssertEqual(legacyOuterLength, embeddedOuterLength)
        XCTAssertEqual(legacyInnerLength, embeddedInnerLength)
        XCTAssertEqual(legacyBits, embeddedBits)
        XCTAssertEqual(embeddedOuterLength, 2)
        XCTAssertEqual(embeddedInnerLength, 0)
        XCTAssertEqual(embeddedBits, 256)

        let values = [BigUInt(42) as AnyObject]
        let legacyEncoded = web3swift.ABIv2Encoder.encode(types: [.uint(bits: 256)], values: values)
        let embeddedEncoded = TLCore.ABIv2Encoder.encode(types: [.uint(bits: 256)], values: values)
        XCTAssertEqual(embeddedEncoded, legacyEncoded)
        XCTAssertEqual(
            String(describing: TLCore.ABIv2Decoder.decode(
                types: [.uint(bits: 256)], data: embeddedEncoded!
            )),
            String(describing: web3swift.ABIv2Decoder.decode(
                types: [.uint(bits: 256)], data: legacyEncoded!
            ))
        )
    }

    func testABIv2StaticDynamicAndAddressValuesMatchLegacyWeb3() throws {
        let legacyTypes: [web3swift.ABIv2.Element.ParameterType] = [
            .bool, .uint(bits: 256), .int(bits: 256), .bytes(length: 4),
            .string, .dynamicBytes, .array(type: .uint(bits: 256), length: 0), .address
        ]
        let embeddedTypes: [TLCore.ABIv2.Element.ParameterType] = [
            .bool, .uint(bits: 256), .int(bits: 256), .bytes(length: 4),
            .string, .dynamicBytes, .array(type: .uint(bits: 256), length: 0), .address
        ]
        let values: [AnyObject] = [
            true as AnyObject,
            BigUInt(42) as AnyObject,
            BigInt(-7) as AnyObject,
            Data([1, 2, 3, 4]) as AnyObject,
            "tron" as AnyObject,
            Data([5, 6, 7]) as AnyObject,
            [BigUInt(1) as AnyObject, BigUInt(2) as AnyObject] as AnyObject,
            "0x7e5f4552091a69125d5dfcb7b8c2659029395bdf" as AnyObject
        ]
        let legacyEncoded = try XCTUnwrap(web3swift.ABIv2Encoder.encode(
            types: legacyTypes, values: values
        ))
        let embeddedEncoded = try XCTUnwrap(TLCore.ABIv2Encoder.encode(
            types: embeddedTypes, values: values
        ))
        XCTAssertEqual(embeddedEncoded, legacyEncoded)
        XCTAssertEqual(
            String(describing: TLCore.ABIv2Decoder.decode(types: embeddedTypes, data: embeddedEncoded)),
            String(describing: web3swift.ABIv2Decoder.decode(types: legacyTypes, data: legacyEncoded))
        )
    }
}
```

The literal final differential command was:

```bash
set -o pipefail
xcodebuild test -quiet -workspace tronlink-iOS-core.xcworkspace -scheme tronlink-iOS-core_Tests -destination 'platform=iOS Simulator,id=017E8DDA-425E-420F-9644-82B896E4907C' -derivedDataPath /tmp/tlcore-stage-3-differential-pruned CODE_SIGNING_ALLOWED=NO -only-testing:tronlink-iOS-core_Tests/EmbeddedWeb3DifferentialTests 2>&1 | tee /tmp/tlcore-stage-3-differential-structural-green.log
```

The actual differential inputs and comparisons were:

| Differential surface | Concrete input | Comparison outcome |
| --- | --- | --- |
| Private key/signing | Private key `0000000000000000000000000000000000000000000000000000000000000001`; message hash `1111111111111111111111111111111111111111111111111111111111111111` | Public keys, checksum addresses, deterministic signature bytes, and recovered-address strings were equal between modules |
| ABI parser/uint | Invalid `(address,uint256[])[]`; valid `uint256[][2]`; `uint256(42)` | Both rejected the invalid spelling; both produced fixed array 2 -> dynamic array -> `uint256`; encoded optionals and decoded descriptions were equal |
| ABI static/dynamic/address | Types `bool,uint256,int256,bytes4,string,bytes,uint256[],address`; values `true`, `42`, `-7`, `01020304`, `"tron"`, `050607`, `[1,2]`, `0x7e5f4552091a69125d5dfcb7b8c2659029395bdf` | Encoded bytes and decoded descriptions were equal between modules |

After removing the legacy module, the committed golden tests retained these
independent concrete inputs and outputs:

| Surface | Concrete input | Golden output |
| --- | --- | --- |
| Private key | `0000000000000000000000000000000000000000000000000000000000000001` | Public key `0479be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798483ada7726a3c4655da4fbfc0e1108a8fd17b448a68554199c47d08ffb10d4b8`; address `0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf` |
| Signing/recovery | Message hash `1111111111111111111111111111111111111111111111111111111111111111` | 65-byte recoverable signature, recovery id below 4, and recovered address `0x7E5F4552091A69125d5DfCb7b8C2659029395Bdf` |
| ABI parser | Invalid `(address,uint256[])[]`; valid `uint256[][2]` | Invalid spelling rejected; valid structure is fixed array 2 -> dynamic array -> `uint256` |
| ABI uint | Type `uint256`, value `42` | `000000000000000000000000000000000000000000000000000000000000002a`; decode returns `42` |
| Hex helpers | bytes `00 ff`; string `TRON`; odd hex `0xabc`; empty `0x` | `00ff`; `54524f4e`; bytes `ab 0c`; empty input rejected |
| Event log | Address `0x53066cddbc0099eb6c96785d9b3df2aaeede5da3`, block `0x4f58f8`, index `0x84`, three 32-byte topics | 20-byte address, block `5200120`, index `132`, three decoded topics |

## Build configuration

The podspec keeps libsecp implementation headers private, exposes only the
umbrella-referenced public header, and adds these C flags:

```text
$(inherited) -pedantic -Wall -Wextra -Wcast-align -Wnested-externs
-Wshadow -Wstrict-prototypes -Wno-shorten-64-to-32
-Wno-conditional-uninitialized -Wno-unused-function -Wno-long-long
-Wno-overlength-strings -O3
```

No field/scalar configuration macro is supplied by the build. The pinned
`secp256k1-config.h` remains authoritative.

After dependency removal, both Example locks and both main validation locks
contain zero occurrences of `TronWalletWeb3Swift`,
`tron-wallet-secp256k1`, `PromiseKit`, or `scrypt.c`. Example `pod install`
removed all four old pod targets. The main project also needed its pre-existing
hard-coded PromiseKit header/link settings and scrypt/web3swift linker entries
removed; generated Pods xcconfigs already omitted them.

## Validation evidence

| Gate | Result | Artifact |
| --- | --- | --- |
| Accepted RED | Exit 65; only the planned missing TLCore Web3 symbols | `/tmp/tlcore-stage-3-red-valid.log`, SHA-256 `b7846de227a3043208d241b256c07f1116067ecd5bf748f498181d4a9b275bd1` |
| Dynamic legacy differential | 3 passed, 0 failed | `/tmp/tlcore-stage-3-differential-structural-green.log`, SHA-256 `1eb769d05ce5a29dce4e5509fbe15827f33f71b19927b392845be6a613369523` |
| Post-removal Web3 golden | 5 passed, 0 failed | `/tmp/tlcore-stage-3-collision-fix-golden.log`, SHA-256 `1391b1210271bf3dba46b2a76a9157ec07fca8874dd3de2ec01d5a2139af4caf` |
| Accepted full Example suite | 61 passed, 0 failed, 0 skipped in the collected result, with exactly four known tests excluded by selector | `/tmp/tlcore-stage-3-complete-example.log`, SHA-256 `596bd9821661a503fc5fc643534cf88d034b9ee2e00325d4817c4f3149aa5fd1` |
| CocoaPods lint | Passed with warnings under the established Xcode 26 iOS 13 xcconfig override | `/tmp/tlcore-stage-3-pod-lib-lint-ios13-xcconfig.log`, SHA-256 `86da45acda070a1e809855f0a548c00697787b062b6839672b6326bcf6fac300` |
| Main Debug simulator build | Exit 0 | `/tmp/tlcore-stage-3-main-build-final.log`, SHA-256 `3f7ee59a7958e2b8bb9060d853f99afe9429aa3df97d3037f75fe9935eca48f7` |
| Main focused tests | 8 passed, 0 failed | `/tmp/tlcore-stage-3-main-focused-eight-tests.log`, SHA-256 `dacbe7dbe27e4777ddf5ee9620bbdbe4611525ecbd36bc18295ab668392317ad` |
| Required local-path pod install | Exit 0; zero standalone Web3/secp dependency names | `/tmp/tlcore-stage-3-review-main-pod-install.log`, SHA-256 `74378b50678184cb09877e12152d5ca9c39f5cc7ecc12fe2fe838e43515453bd` |
| Required local-path Debug build | Exit 0 | `/tmp/tlcore-stage-3-review-main-build-confirm.log`, SHA-256 `460f5e99a6575baab1064d537a44cebb48aa9817ccd06fa20bcf11c2b2621366` |

The accepted full-suite xcresult is
`/tmp/tlcore-stage-3-complete-example/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_18-57-13-+0800.xcresult`.
Its 61 tests include all five Web3 golden tests. The four command-level
exclusions are the two previously accepted legacy tests and two previously
accepted ASCII-keystore incompatibility tests; none is a Task 3 test.

The previously recorded whole-tree xcresult hashes (`09edbe05...` for the
accepted suite, `8ffa57bd...` for the post-`Data.bytes` focus, and `019493b8...`
for the main eight-test focus) are point-in-time diagnostic snapshots only.
They are not reproducible acceptance gates: `xcresulttool` creates or updates
mutable `TestReport` cache files inside an xcresult bundle while querying it,
which changes a subsequent whole-tree hash without changing the test result.
Repeatable evidence is the committed literal test command, the immutable tee
log and its SHA-256 above, and a result-node query for the counts:

```bash
xcrun xcresulttool get test-results summary --path /tmp/tlcore-stage-3-complete-example/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_18-57-13-+0800.xcresult
xcrun xcresulttool get test-results tests --path /tmp/tlcore-stage-3-complete-example/Logs/Test/Test-tronlink-iOS-core_Tests-2026.08.25_18-57-13-+0800.xcresult
```

Those queries establish the result nodes and counts; no later hash of the
mutable xcresult directory is used to accept or reject Task 3.

Plain `pod lib lint` is expected to fail under Xcode 26 because FMDB's
historical deployment target requests the removed `libarclite`; the accepted
run uses the Task 2 workaround
`XCODE_XCCONFIG_FILE=/tmp/tlcore-lint-xcode26.xcconfig`, containing
`IPHONEOS_DEPLOYMENT_TARGET = 13.0`. It changes no repository file.

## Header and symbol evidence

The built TLCore public Headers directory contains exactly one `ecdsa.h`, the
Trezor header. Libsecp's root `ecdsa.h` remains private.

The literal brief command `nm -gU TLCore | sort | uniq -d` returns 18 rows on
the statically archived framework product, rather than an empty result. These
rows are archive-level repeated Swift compiler/profile records and include
address-qualified entries; this command is not a valid duplicate-definition
oracle for an archive. The authoritative per-architecture name-only scan
(`nm -gjU`, then duplicate names) reports zero duplicate names matching
`secp256k1_*` or `ecdsa_*`. The final main static link also succeeded without a
duplicate-symbol diagnostic. The exact 18-row artifact has SHA-256
`6c0b185cd0978175232cb4c64a1844fe1887fae93f7896b06af60d66276932a5`.

The public header artifact is
`/tmp/tlcore-stage-3-public-ecdsa-headers.txt`, SHA-256
`22579a98d2e39886c9b68e149b56ad5487def0fc62316f52bc809a1fc9848891`.

## Source-fidelity whitespace exception

`git diff --cached --check` reported 70 whitespace diagnostics in 14 imported
files: two diagnostics in byte-identical secp files and 68 in mapped pinned
Web3 files. Every warned Web3 line/EOF was found byte-for-byte in its pinned
source file. Custom compatibility files, tests, podspec, umbrella,
`TLWalletCore`, licenses, and the Example project had zero diagnostics.
Normalizing the pinned whitespace would reduce source fidelity, so the
inherited diagnostics are deliberately retained.

## Risks, rollback, and Task 4 entry gate

Remaining risks are limited to the intentionally preserved upstream Swift 4.2
surface and static-archive tooling ambiguity described above. Functional,
dependency, public-header, lint, full-suite, main-build, and main-test gates are
green.

Rollback the consumer first, then the provider:

```bash
git -C /Users/viccc/working/4_22_0/TronLink_iOS revert 45b4b6019de1b93c07eae13ee587d5bd948ecaab
git -C /Users/viccc/working/4_22_0/TronLink_iOS revert b87d495eaac377a9ade9f3b6baa2710696a76b85
git -C /Users/viccc/source/tronlink-iOS-core revert 7f20566c229c82c6abe731f0e941addcbcdaf17a
```

Task 4 may begin only from the two implementation commits and the main
local-source-gate fix above plus this documentation commit, with no old
Web3/secp imports or dependency names, the 61-test accepted Example suite
green, the 8-test main gate green, and the user-owned main lock/project changes
still preserved outside Task 3 commits.
