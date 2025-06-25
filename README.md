# kaspa_flutter_core

A **pure-Dart** core library for building Kaspa wallets and applications in Flutter.  
Provides all on-chain wallet logic, BIP-standards, address/transaction encoding, protocol buffers/gRPC bindings, and utilities—**without any UI or platform-specific code**.

Forked from [Kaspium](https://github.com/azbuky/kaspium_wallet

---

## 📦 Features

- **BIP-Standards**  
  - BIP-32 hierarchical deterministic wallets (key derivation)  
  - BIP-39 mnemonic generation & seed derivation  
  - BIP-340 Schnorr signature utilities  

- **Address Encoding**  
  - Kaspa Bech32 address codec (`kaspa:` HRP)  
  - Base58Check support  

- **Transaction Construction & Validation**  
  - UTXO mass calculation (KIP-9)  
  - Raw-tx builder & fee estimation  
  - Script / TxScript helper functions  

- **Protocol Bindings**  
  - Protobuf messages & JSON/gRPC stubs generated from `proto/`  
  - High-level async `KaspaClient` for both gRPC and JSON-RPC  

- **Utilities**  
  - Currency conversion (KAS ↔ SOMPI)  
  - URI parsing (`kaspa:` URI scheme)  
  - Big-number formatting & parsing  

- **Pure Dart**  
  No native or platform channels—ideal for **both** Flutter and standalone Dart VM use.

---

## 🚀 Installation

Add to your app or package’s `pubspec.yaml`:

```yaml
dependencies:
  kaspa_flutter_core: ^0.1.0
```

Then run:

```bash
flutter pub get
# or, for pure Dart:
dart pub get
```

> **Note:** There are no Flutter widget dependencies—only Dart SDK and pure-Dart libraries.

---

## ⚡ Quick Start

```dart
import 'dart:convert';
import 'package:hex/hex.dart';
import 'package:kaspa_flutter_core/kaspa/bip39/bip39.dart';
import 'package:kaspa_flutter_core/kaspa/bip32/bip32.dart';
import 'package:kaspa_flutter_core/kaspa/types/address.dart';
import 'package:kaspa_flutter_core/client/kaspa_client.dart';

void main() async {
  // 1. Generate a new 12-word mnemonic
  final mnemonic = generateMnemonic(strength: 128);
  print('Mnemonic: \$mnemonic');

  // 2. Derive seed & master key
  final seedHex = mnemonicToSeedHex(mnemonic, passphrase: '');
  final seedBytes = HEX.decode(seedHex);
  final masterKey = BIP32.fromSeed(Uint8List.fromList(seedBytes));

  // 3. Derive an address
  final child = masterKey.derivePath("m/44'/1234'/0'/0/0");
  final address = Address.publicKey(child.publicKey).toString();
  print('Kaspa address: \$address');

  // 4. Query balance via JSON-RPC
  final client = KaspaClient.jsonRpc('http://localhost:16210');
  final balance = await client.getBalance(address);
  print('Balance: \${balance.kaspa} KAS (~\${balance.sompi} sompi)');
}
```

---

## 📚 API Overview

### Wallet & Key Management

- **BIP-39**  
  - `generateMnemonic({int strength = 128})`  
  - `validateMnemonic(String phrase)`  
  - `mnemonicToSeedHex(String phrase, {String passphrase = ''})`

- **BIP-32**  
  - `BIP32.fromSeed(Uint8List seed)`  
  - `derivePath(String path)`  
  - `toBase58()` / `BIP32.fromBase58(String)`  

- **BIP-340**  
  - `signSchnorr(Uint8List msg, Uint8List privKey)`  
  - `verifySchnorr(Uint8List msg, Uint8List signature, Uint8List pubKey)`

### Address & Encoding

- **Bech32**  
  - `Bech32.encode(String hrp, Uint8List data)`  
  - `Bech32.decode(String address) → Bech32Data(hrp, data)`

- **Base58Check**  
  - `bs58check.encode(Uint8List payload)`  
  - `bs58check.decode(String address) → Uint8List`

- **URI Parsing**  
  - `KaspaUri.tryParse(String uri, {AddressPrefix prefix})`

### Transactions

- **Mass Calculation**  
  - `MassCalculator.compute(List<Uint8List> inputs, List<BigInt> outputs, int version)`

- **Builder & Scripts**  
  - `TransactionBuilder.build(...)`  
  - `TxScript` helper functions

### RPC & gRPC Clients

- **gRPC** (`grpc` package)  
  - `KaspaGrpcClient` with methods like `getBlockCount()`, `getRecentBlocks()`, etc.

- **JSON-RPC** (`http` package)  
  - `KaspaJsonRpcClient` for HTTP-based JSON-RPC calls

Both clients share Protobuf models under `lib/kaspa/grpc/` and `lib/kaspa/rpc/`.

---

## 🧪 Testing

All core logic is covered by **pure-Dart** unit tests:

```bash
dart test
```

Test suites under `test/` cover:

- BIP-standards (bip32, bip39, bip340)  
- Address encoding (bech32, bs58check)  
- Transaction mass & builder  
- URI parsing, serialization, utilities  

---

## 🔧 Code Generation

1. **Protobuf / gRPC**  
   Install [buf](https://buf.build/) and the Dart plugin:

   ```bash
   brew install bufbuild/buf/buf  
   dart pub global activate protoc_plugin
   ```

   Then:

   ```bash
   buf generate
   ```

2. **Freezed & JSON Serialization**  
   (Only if you modify `.freezed.dart` or `.g.dart` models)

   ```bash
   dart run build_runner build --delete-conflicting-outputs
   ```

---

## 🤝 Contributing

1. **Fork** the repo  
2. Create a **feature branch**  
3. Write code & **unit tests**  
4. Open a **PR** and ensure `dart test` passes

Please follow code style and add tests for new logic.

---

Happy hacking! 🚀  
