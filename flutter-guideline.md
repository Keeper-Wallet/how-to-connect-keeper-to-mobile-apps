# Connect Keeper Wallet in your app (Flutter)

To interact with Keeper Wallet in your iOS or Android app, complete the following steps:

- [Step 1. Register your project in WalletConnect Cloud](#step-1-register-your-project-in-walletconnect-cloud)
- [Step 2. Install WalletConnect](#step-2-install-walletconnect)
- [Step 3. Initialize WalletConnect client](#step-3-initialize-walletconnect-client)
- [Step 4. Get a pairing URI for Keeper Wallet](#step-4-get-a-pairing-uri-for-keeper-wallet)
- [Step 5. Connect Keeper Wallet](#step-5-connect-keeper-wallet)
- [Step 6. Sign a transaction/order/message](#step-6-sign-a-transactionordermessage)

> For more information on using Flutter implementation of WalletConnect, see their [docs](https://docs.walletconnect.com/2.0/flutter/installation).

## Step 1. Register your project in WalletConnect Cloud

1. Go to [WalletConnect Cloud](https://cloud.walletconnect.com/app) and sign in or sign up.
2. Click <b>+ New project</b>.
3. Give your project a name and click <b>Create</b>.
4. On the project page, obtain a <b>Project ID</b>.

## Step 2. Install WalletConnect

pubspec.yaml:

```
walletconnect_flutter_v2: ^1.2.3
```

## Step 3. Initialize WalletConnect client

Describe your app and define its appearance in Keeper Wallet when a user is prompted to connect or sign a transaction/order/message.

```dart
static const projectId = 'YOUR_PROJECT_ID';

_wcClient = await Web3App.createInstance(
  relayUrl: 'wss://relay.walletconnect.com',
  projectId: projectId,
  metadata: const PairingMetadata(
    name: 'YOUR_APP_NAME',
    description: 'YOUR_APP_DESC',
    url: 'YOUR_WEBSITE',
    icons: ["YOUR_IMAGE_URLS"],
  ),
);
```

## Step 4. Get a pairing URI for Keeper Wallet

Send a request to WalletConnect to get a pairing URI for Keeper Wallet.

```dart
// Set the chain ID: 'W' for Mainnet or 'T' for Testnet
static const testChainId = 'waves:T';
static const mainChainId = 'waves:W';

final resp = await _wcClient?.connect(requiredNamespaces: {
  namespace: const RequiredNamespace(
    chains: [testChainId],
    methods: [
      'waves_signTransaction',
      'waves_signTransactionPackage',
      'waves_signOrder',
      'waves_signMessage',
      'waves_signTypedData',
    ],
    events: [],
  ),
});

final wcUrl = resp?.uri.toString();
```

## Step 5. Connect Keeper Wallet

Here's how it works:

1. Your app calls Keeper, specifying the callback URL in the request.
2. The Keeper Wallet app opens and prompts the user to connect.
3. Once the user confirms or cancels the connection, your app receives a callback.

[Deep Links](https://docs.flutter.dev/ui/navigation/deep-linking) are used to interact with Keeper Wallet. You can use [url_launcher](https://pub.dev/packages/url_launcher) plugin to call Keeper. For the callback, the Deep Link must be configured in your app.

```dart
static const callback = 'YOUR_LINK_OR_SCHEME';

final parameters = <String>[];
parameters.add('wcurl=${Uri.encodeComponent(wcUrl)}');
parameters.add('callback=$callback');

final query = parameters.join('&');
final url = Uri.parse('https://link.keeper-wallet.app/auth?$query');

//url_launcher
launchUrl(url, mode: LaunchMode.externalApplication);
```

The result of connecting comes to the listener:

```dart
_wcClient?.onSessionConnect.subscribe((SessionConnect? connect) {
  print('onSessionConnect: $connect');
  _topic = connect?.session.topic;
});
```

## Step 6. Sign a transaction/order/message

Here is how it works:

1. Your app sends a signing request via WalletConnect.
2. Your app calls Keeper, specifying the callback URL in the request.
3. The Keeper Wallet app opens and prompts the user to sign the transaction, order, or custom message.
4. Once the user confirms or cancels the request, your app receives a callback.

### Transaction

Example:

```dart
static const testChainId = 'waves:T';
static const mainChainId = 'waves:W';
static const Map<String, Object> invoke = { YOUR_TX_PARAMS };

_wcClient?.request(
  topic: _topic,
  chainId: testChainId,
  request: SessionRequestParams(
    method: 'waves_signTransaction',
    params: [invoke],
  ),
)
.then((value) => print('request result: $value'))
.onError((error, _) => print('request error: $error'));
```

### Order

```dart
static const testChainId = 'waves:T';
static const mainChainId = 'waves:W';
static const Map<String, Object> order = { YOUR_ORDER_PARAMS };

_wcClient?.request(
  topic: _topic,
  chainId: testChainId,
  request: SessionRequestParams(
    method: 'waves_signOrder',
    params: [order],
  ),
)
.then((value) => print('request result: $value'))
.onError((error, _) => print('request error: $error'));
```

### Custom message

```dart
static const testChainId = 'waves:T';
static const mainChainId = 'waves:W';
static const message = 'YOUR_MESSAGE';

_wcClient?.request(
  topic: _topic,
  chainId: testChainId,
  request: SessionRequestParams(
    method: 'waves_signMessage',
    params: [message],
  ),
)
.then((value) => print('request result: $value'))
.onError((error, _) => print('request error: $error'));
```

### Call Keeper Wallet

Specify `topic` and `callback` in the request.

```dart
static const callback = 'YOUR_LINK_OR_SCHEME';

final parameters = <String>[];
parameters.add('topic=$_topic');
parameters.add('callback=$callback');

final query = parameters.join('&');
final url = Uri.parse('https://link.keeper-wallet.app/wakeup?$query');

//url_launcher
launchUrl(url, mode: LaunchMode.externalApplication);
```
