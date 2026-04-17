# tronlink-iOS-core

TronLink Wallet is a decentralized non-custodial wallet.TronLink-Core is the core module of TronLink Wallet, which provides core functions such as Create Wallet, Get Address and Sign Transaction.

## Analytics Module (Privacy-First Design)

The `Metrics` module collects aggregated usage data only. **Wallet addresses, keys, mnemonics, transaction hashes, counter-parties, and device identifiers are never transmitted.** The backend receives only opaque UUIDs and same-day aggregated buckets; individual transactions are not reconstructible.

### Address → UUID Anonymization

Each address is mapped to a random UUID stored **locally only** (FMDB, with migration from legacy `UserDefaults`). The backend never sees the address, and the same user's two wallets appear as two independent UUIDs (no server-side linkage).

```swift
// TRXAddressMapManager.swift — local-only address → uuid mapping
public func id(for address: String) -> String {
    let normalized = Self.normalizeAddress(address)
    var existing: String?
    queue.sync { existing = mapping[normalized] }
    if let v = existing { return v }

    var result = ""
    var needsSave = false
    queue.sync(flags: .barrier) {
        if let v = self.mapping[normalized] { result = v; return }
        var candidate = Self.generateUUIDFull()            // UUID().uuidString
        while self.usedIds.contains(candidate) { candidate = Self.generateUUIDFull() }
        self.mapping[normalized] = candidate
        self.usedIds.insert(candidate)
        result = candidate
        needsSave = true
    }
    if needsSave {
        DispatchQueue.global(qos: .utility).async {
            TRXMetricsDBManager.shared.saveAddressMappings(self.allMappings()) // on-device only
        }
    }
    return result
}
```

### What Is Uploaded

Records are merged locally per `(uId, actionType, tokenAddress, date)` before upload, and raw amounts are distributed into 9 logarithmic buckets (`A1`..`A9`, e.g. `A1=(0,1]`, ..., `A9=(10m,+∞)`). Only these two payloads are sent (encrypted):

```swift
// TRXAssetSyncModel — daily asset snapshot (one per uId per UTC day)
public var uId: String?           // anonymous UUID, never the address
public var idType: Int?           // wallet provenance enum; watch wallets skipped
public var trxBalance, usdtBalance, usdBalance: String?
public var date: String?
public var chain: String?

// TRXTransactionSyncModel — aggregated daily action counts
public var uId: String?
public var actionType: Int?       // closed enum; unknown contract calls discarded
public var tokenAddress: String?  // TRX / TRC10 / TRC20 id
public var count: Int?
public var tokenAmount, energy, bandwidth, burn: String?
public var A1, A2, A3, A4, A5, A6, A7, A8, A9: Int?  // bucketed amount distribution
public var date: String?
```

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


