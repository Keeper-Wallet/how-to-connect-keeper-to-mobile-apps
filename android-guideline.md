# Connect Keeper Wallet in your Android app (Kotlin)

**Requirements**
* Android min SDK 23
* Java 11

To interact with Keeper Wallet in your Android app, complete the following steps:

- [Step 1. Register your project in WalletConnect Cloud](#step-1-register-your-project-in-walletconnect-cloud)
- [Step 2. Install WalletConnect](#step-2-install-walletconnect)
- [Step 3. Initialize WalletConnect client](#step-3-initialize-walletconnect-client)
- [Step 4. Create a pairing URI for Keeper Wallet](#step-4-create-a-pairing-uri-for-keeper-wallet)
- [Step 5. Connect Keeper Wallet](#step-5-connect-keeper-wallet)
- [Step 6. Sign a transaction](#step-6-sign-a-transaction)

> For more information on how to use Kotlin implementation of WalletConnect, see their [docs](https://docs.walletconnect.com/2.0/kotlin/web3wallet/installation).

## Step 1. Register your project in WalletConnect Cloud

1. Go to [WalletConnect Cloud](https://cloud.walletconnect.com/app) and sign in or sign up.
2. Click <b>+ New project</b>.
3. Give your project a name and click <b>Create</b>.
4. On the project page, obtain a <b>Project ID</b>.

## Step 2. Install WalletConnect

root/build.gradle.kts:

```swift
allprojects {
 repositories {
      mavenCentral()
      maven { url "https://jitpack.io" }
 }
}
```

app/build.gradle.kts

```swift
implementation("com.walletconnect:android-core:release_version")
implementation("com.walletconnect:sign:release_version")
```

## Step 3. Initialize WalletConnect client

```kotlin
private val projectId = "YOUR_PROJECT_ID"
private val relayUrl = "relay.walletconnect.com"
private val serverUrl = "wss://$relayUrl?projectId=$projectId"

val connectionType = ConnectionType.AUTOMATIC

//Describe your app and define its appearance
val appMetaData = Core.Model.AppMetaData(
    name = "YOUR_APP_NAME",
    description = "YOUR_APP_DESC",
    url = "YOUR_WEBSITE",
    icons = listOf("YOUR_IMAGE_URLS"),
    redirect = null
)

CoreClient.initialize(relayServerUrl = serverUrl, connectionType = connectionType, application = this.application, metaData = appMetaData) { error ->
    // Error will be thrown if there's an issue during initialization
}

SignClient.initialize(Sign.Params.Init(core = CoreClient)) { error ->
    // Error will be thrown if there's an issue during initialization
}
```

## Step 4. Create a pairing URI for Keeper Wallet

```kotlin
val pairing = CoreClient.Pairing.create()

val chains: List<String> = listOf("waves:T")
val methods: List<String> = listOf(
    "waves_signTransaction", 
    "waves_signTransactionPackage", 
    "waves_signMessage", 
    "waves_signTypedData"
)
val events: List<String> = listOf()
val namespaces: Map<String, Sign.Model.Namespace.Proposal> = mapOf(
    namespace to Sign.Model.Namespace.Proposal(chains, methods, events, null)
)

val connectParams = Sign.Params.Connect(namespaces, pairing)
SignClient.connect(connectParams,
    onSuccess = {
        val wcUrl = pairing.uri      // URI for opening Keeper Wallet
    },
    onError = { error ->
        // Error will be thrown if there's an issue during connect
    }
)
```

## Step 5. Connect Keeper Wallet

[Deep Links](https://developer.android.com/training/app-links/deep-linking) are used to interact with Keeper Wallet. As a result of a call, Keeper Wallet is opened and the user is prompted to connect.

After the user confirms or cancels the connection, your app is called back. For the callback, the Deep Link must be configured in your app.

```kotlin
val callback = "YOUR_DEEP_LINK"
val parameters = ArrayList<String>()

parameters.add("wcurl=${URLEncoder.encode(wcUrl, "utf-8")}")
parameters.add("callback=$callback")

val query = parameters.joinToString("&")

val browserIntent = Intent(
    Intent.ACTION_VIEW,
    Uri.parse("https://link.keeper-wallet.app/auth?$query")
)

startActivity(browserIntent)
```

The result of connecting is passed via `onSessionApproved` delegate function. Read more in [SignClient.DappDelegate](https://docs.walletconnect.com/2.0/kotlin/sign/dapp-usage#signclientdappdelegate) section of WalletConnect docs.

```kotlin
override fun onSessionApproved(approvedSession: Sign.Model.ApprovedSession) {
    val sessionTopic = approvedSession.topic
}
```

## Step 6. Sign a transaction

As a result of the request, Keeper Wallet is opened and the user is prompted to sign the transaction.

After the user confirms or cancels the transaction, your app is called back.

Example:

```kotlin
val params = JSONObject( YOUR_TX_PARAMS ).toString()
SignClient.request(Sign.Params.Request(
    sessionTopic = sessionTopic,
    method = "waves_signTransaction",
    params =listOf(params).toString(),
    chainId = currentChainId
)){error->
    // Error will be thrown if there's an issue during request
}
```

Call Keeper Wallet with `topic` specified:

```kotlin
private val callback = "YOUR_DEEP_LINK"
val parameters = ArrayList<String>()

parameters.add("topic=$sessionTopic")
parameters.add("callback=$callback")

val query = parameters.joinToString("&")

val browserIntent = Intent(
    Intent.ACTION_VIEW,
    Uri.parse("https://link.keeper-wallet.app/wakeup?$query")
)

startActivity(browserIntent)
```

The result of signing the transaction comes in the `onSessionRequestResponse` function.
