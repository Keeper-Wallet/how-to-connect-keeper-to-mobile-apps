# Connect Keeper Wallet in your app (Swift)

To interact with Keeper Wallet in your iOS app, complete the following steps:

- [Step 1. Register your project in WalletConnect Cloud](#step-1-register-your-project-in-walletconnect-cloud)
- [Step 2. Install WalletConnect](#step-2-install-walletconnect)
- [Step 3. Set up the networking configuration](#step-3-set-up-the-networking-configuration)
- [Step 4. Configure the Pair instance](#step-4-configure-the-pair-instance)
- [Step 5. Create a pairing URI for Keeper Wallet](#step-5-create-a-pairing-uri-for-keeper-wallet)
- [Step 6. Connect Keeper Wallet](#step-6-connect-keeper-wallet)
- [Step 7. Sign a transaction](#step-7-sign-a-transaction)

> For more information on using Swift implementation of WalletConnect, see their [docs](https://docs.walletconnect.com/2.0/swift/web3wallet/installation).

## Step 1. Register your project in WalletConnect Cloud

1. Go to [WalletConnect Cloud](https://cloud.walletconnect.com/app) and sign in or sign up.
2. Click <b>+ New project</b>.
3. Give your project a name and click <b>Create</b>.
4. On the project page, obtain a <b>Project ID</b>.

## Step 2. Install WalletConnect

### Cocoapods

1. Update Cocoapods spec repos. Type in terminal  `pod repo update`
2. Initialize Podfile if needed with `pod init`
3. Add pod to your Podfile:

   ```kotlin
   pod 'WalletConnectSwiftV2'
   ```
4. Install pods with `pod install`.

### SwiftPackageManager

1. Open XCode.
2. Go to **File → Add Packages**.
3. Paste the repo GitHub url:
    
    `https://github.com/WalletConnect/WalletConnectSwiftV2`

4. Click **Add Package**.
5. Select **WalletConnect** check mark.

## Step 3. Set up the networking configuration

```swift
let RELAY_URL = "relay.walletconnect.com"
let PROJECT_ID = "YOUR_PROJECT_ID"

Networking.configure(
      relayHost: RELAY_URL,
      projectId: PROJECT_ID,
      socketFactory: SocketFactory()
)
```

## Step 4. Configure the Pair instance

Describe your app and define its appearance in Keeper Wallet  when a user is prompted to connect or sign a transaction/order/message.

```swift
let metadata = AppMetadata(name: "YOUR_APP_NAME",
      description: "YOUR_APP_DESC",
      url: "YOUR_WEBSITE",
      icons: ["YOUR_IMAGE_URLS"]
)

Pair.configure(metadata: metadata)
```

## Step 5. Get a pairing URI for Keeper Wallet

Send a request to WalletConnect to get a pairing URI for Keeper Wallet.

```swift
let methods: Set<String> = [
      "waves_signTransaction", 
      "waves_signTransactionPackage", 
      "waves_signOrder",
      "waves_signMessage",
      "waves_signTypedData"
]
// Set the chain ID: 'W' for Mainnet or 'T' for Testnet
let blockchains: Set<Blockchain> = [Blockchain("waves:T")!]
let namespaces: [String: ProposalNamespace] = [
      "waves": ProposalNamespace(chains: blockchains, methods: methods, events: [])
]
let uri = try! await Pair.instance.create()

try! await Sign.instance.connect(
      requiredNamespaces: namespaces, 
      topic: uri.topic
)
```

## Step 6. Connect Keeper Wallet

Here's how it works:

1. Your app calls Keeper, specifying the callback URL in the request.
2. The Keeper Wallet app opens and prompts the user to connect.
3. Once the user confirms or cancels the connection, your app receives a callback.

To call Keeper Wallet, use the universal link. For the callback, a universal link or a [custom URL scheme](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app) must be defined for your app.

```swift
let callback = "YOUR_LINK_OR_SCHEME"
var params: [String] = [];

let urlEncode = uri.absoluteString.addingPercentEncoding(withAllowedCharacters: .alphanumerics)
params.append("wcurl=\(urlEncode!)")

params.append("callback=\(callback)")

let query = params.joined(separator: "&")
let url = URL(string: "https://link.keeper-wallet.app/auth?\(query)")
UIApplication.shared.open(url)
```

The result of connecting comes to the listener:

```swift
Sign.instance.sessionSettlePublisher
      .receive(on: DispatchQueue.main)
      .sink { [unowned self] (session: Session) in
            // Result handler 
            // print(session) to display the result in the console
      }.store(in: &publishers)
```

## Step 7. Sign a transaction/order/message

Here is how it works:

1. Your app sends a signing request via WalletConnect.
2. Your app calls Keeper, specifying the callback URL in the request.
3. The Keeper Wallet app opens and prompts the user to sign the transaction, order, or custom message.
4. Once the user confirms or cancels the request, your app receives a callback.

### Transaction

Example:

```swift
let method = "waves_signTransaction"
let invokeJson = { YOUR_TX_PARAMS }
let requestParams = AnyCodable([invokeJson])

let request = Request(topic: currentSession!.topic, method: method, params: requestParams, chainId: blockchains.first!)
try! await Sign.instance.request(params: request)
```

### Order

```swift
let method = "waves_signOrder"
let orderJson = { YOUR_ORDER_PARAMS }
let requestParams = AnyCodable([orderJson])

let request = Request(topic: currentSession!.topic, method: method, params: requestParams, chainId: blockchains.first!)
try! await Sign.instance.request(params: request)
```

### Custom message

```swift
let method = "waves_signMessage"
let message = "YOUR_MESSAGE"
let requestParams = AnyCodable([message])

let request = Request(topic: currentSession!.topic, method: method, params: requestParams, chainId: blockchains.first!)
try! await Sign.instance.request(params: request)
```

### Call Keeper Wallet

Specify `topic` and `callback` in the request.

```swift
let callback = "YOUR_LINK_OR_SCHEME"
var params: [String] = [];

params.append("topic=\(topic)")
params.append("callback=\(callback)")

let query = params.joined(separator: "&")
let url = URL(string: "https://link.keeper-wallet.app/wakeup?\(query)")
UIApplication.shared.open(url)
```

The result of signing the request comes to the listener:

```swift
Sign.instance.sessionResponsePublisher
      .receive(on: DispatchQueue.main)
      .sink { [unowned self] (response: Response) in
            // Result handler 
            // print(response) to display the result in the console
      }.store(in: &publishers)
```
