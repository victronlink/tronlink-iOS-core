# tronlink-iOS-core

TronLink Wallet is a decentralized non-custodial wallet. TronLink-Core is the core module of TronLink Wallet, which provides core functions such as Create Wallet, Get Address, Sign Transaction, and on-device statistical Metrics collection.

## Example

To run the example project, clone the repo, and run `pod install` from the Example directory first.

## Requirements

- iOS 13.0+
- Swift 4.2

## Installation

tronlink-iOS-core is available through [CocoaPods](https://cocoapods.org). To install it, simply add the following line to your Podfile:

```ruby
pod 'tronlink-iOS-core'
```

## Demo

- [Create wallet](./Example/Tests/Tests.swift)
- [Sign transaction](./Example/Tests/Tests.swift)
- [Sign message](./Example/Tests/Tests.swift)
- [Export PrivateKey](./Example/Tests/Tests.swift)
- [Export Mnemonic](./Example/Tests/Tests.swift)
- [Metrics – asset & transaction reporting](./Example/Tests/MetricsTests.swift)

---

## Metrics Module

The **Metrics** module provides anonymous, on-device collection and upload of wallet asset and transaction statistics. All data is encrypted with AES-256-CBC before transmission.

### Architecture

| Layer | Class / Protocol | Responsibility |
|---|---|---|
| **Model** | `TRXAssetSyncModel`, `TRXTransactionSyncModel` | Data models for asset & transaction records |
| **DataBase** | `TRXMetricsDBManager` | Standalone SQLite manager (FMDB), independent of the host app database |
| **Reporter** | `TRXStatisticalUploadManager` | Orchestrates data collection, DB writes, and upload scheduling |
| **Repository** | `TRXStatisticalUploadViewModel` | Builds serialised upload parameters and delegates network requests |
| **Utils** | `TRXMetricsEncryptTool`, `TRXMetricsExtensionTool` | AES-CBC encryption/decryption, date/string helpers |

The module is fully decoupled from the host app through the `TRXMetricsDataSource` protocol.

### Quick Start

**1. Implement `TRXMetricsDataSource`**

```swift
class MyMetricsConfig: TRXMetricsDataSource {
    var environmentKey: String      { "MainNet" }
    var isShastaEnvironment: Bool   { false }
    var isWatchWallet: Bool         { false }
    var isBasicFunctionOpen: Bool   { false }
    var isTokenCloudSyncClose: Bool { false }
    var walletAddress: String       { currentWalletAddress }
    var uploadWalletType: Int       { 1 }
    var usdtContractAddress: String { "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t" }
    var isOnlineEnvironment: Bool   { true }
    var isPreReleaseEnvironment: Bool { false }

    func uploadStatisticalData(parameters: [String: Any],
                                visible: Bool,
                                success: @escaping (Bool, Bool) -> Void,
                                failure: @escaping () -> Void) {
        // Delegate to your network layer
        MyNetworkManager.post(parameters: parameters) { ok in
            success(ok, visible)
        } failure: { failure() }
    }
}
```

**2. Configure at app launch**

```swift
TRXStatisticalUploadManager.shared.dataConfig = MyMetricsConfig()
```

**3. Report asset balances**

```swift
let model = TRXStatisticalUploadManager.shared.makeCurrentAddressAssetSyncModel(
    trxBalance: "1000.5",
    usdtBalance: "200",
    totalUsdBalance: "1200"
)
TRXStatisticalUploadManager.shared.upsertAssetData(model: model)
```

**4. Report a transaction**

```swift
TRXStatisticalUploadManager.shared.upsertTransactionsData(
    actionType: 1,
    tokenAddress: "_",         // "_" for TRX native, contract address for TRC-20
    tokenAmount: "50",
    energy: "1000",
    bandwidth: "300",
    burn: "5"
)
```

**5. Trigger upload** (e.g. on app foreground)

```swift
TRXStatisticalUploadManager.shared.uploadStatisticalData()
```

### Encryption

Upload parameters are encrypted with AES-256-CBC. The key is derived via PBKDF2-SHA256 from the request signature and timestamp. Each encryption uses a fresh random 16-byte IV prepended to the ciphertext.

```swift
// Encrypt
let cipher = TRXMetricsEncryptTool.encryptActionData(secretKey: sig, ts: ts, plaintext: json)

// Decrypt
let plain = TRXMetricsEncryptTool.decryptActionData(secretKey: sig, ts: ts, encryptedText: cipher)
```

### Address Privacy

Wallet addresses are never transmitted in plain text. `TRXAddressMapManager` maintains a persistent, deterministic mapping from each address to a UUID that is reported instead.

```swift
let anonymousId = TRXAddressMapManager.shared.id(for: walletAddress)
```

### Legacy Data Migration

If you are upgrading from an older version that stored metrics in a different database, implement the optional migration methods in `TRXMetricsDataSource`:

```swift
func legacyUpdatedAssetSyncModels(forChain chain: String) -> [TRXAssetSyncModel]? {
    // Return records from your old database for the given chain
}

func legacyUpdatedTransactionSyncModels(forChain chain: String) -> [TRXTransactionSyncModel]? {
    // Return records from your old database for the given chain
}
```

Migration runs automatically once before the first upload cycle for each chain and is not repeated after success.

