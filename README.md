
# yuno-sdk-android

## Android SDK Requirements

In order to use the Yuno Android SDK, you need to meet the following requirements:

- Yuno Android SDK needs your minSdkVersion to be of 21 or above.
- Your project must have Java 8 enabled and use AndroidX instead of older support libraries.
- The android-gradle-plugin version must be 4.0.0 or above.
- The Proguard version must be 6.2.2 or above.
- The kotlin-gradle-plugin version must be 1.4.0 or above.

## Adding the SDK to the project

First, add the repository source using the following code line:

```Gradle
maven { url "https://yunopayments.jfrog.io/artifactory/snapshots-libs-release" }
```



After that, include the code snippet below in the "build.gradle" file to add the Yuno SDK dependency to the application.

```Gradle
dependencies {
    implementation 'com.yuno.payments:android-sdk:{last_version}'
}
```



#### Permissions

We have already included the INTERNET permission by default as we need it to make network requests.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```



### Initialize Yuno

To initialize the Yuno SDK, first, you need to get your public API keys from the Yuno dashboard. Then, if you have not implemented a custom application yet, you will need to create one and call the initialize function in the onCreate() method of your application class.

The following code snippet includes an example of a custom application:

```kotlin
class CustomApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Yuno.initialize(
        this, 
        "your api key",
        config: YunoConfig, // This is a data class to use custom configs in the SDK.
        )
    }
}
```



Please use the YunoConfig data class presented as follows:

```kotlin
data class YunoConfig(
    val cardFlow: CardFormType = CardFormType.ONE_STEP, // This is optional, CardFormType.ONE_STEP by default, this is to choose Payment and Enrollment Card flow.
    val saveCardEnabled: Boolean = false, // This is to choose if show save card checkbox on cards flows.
)
```



In addition, you need to update your manifest to use your application:

```XML
<application
    android:name=".CustomApplication">
</application>
```



## Functions

### Enroll a new payment method

Call the following method from your activity to start an enrollment flow of a new payment method.

````Kotlin
startEnrollment(
    requestCode: Int, //Optional
    customerSession: String,
    countryCode: String,
    showEnrollmentStatus: Boolean, //Optional - Default true
    callbackEnrollmentState: ((String?) -> Unit)?, //Optional - Default null | To register this callback is a must to call ```initEnrollment``` method on the onCreate method of activity.
)
````



#### Callback Enrollment State

To register a callback to get the final enrollment state, it is necessary call the `initEnrollment` method on the onCreate method of activity.

### Checkout

To start a new payment process, you need to call the following method on the onCreate method of activity that calls the SDK:

```Kotlin
startCheckout(
    checkoutSession: "checkout_session",
    countryCode: "country_code_iso",
    callbackOTT: (String?) -> Unit,
    callbackPaymentState: ((String?) -> Unit)?,
)
```



#### Callback One Time Token (OTT)

The `callbackOTT` parameter is a function that returns an OTT needed to complete the payment back to back. This function is mandatory.

#### Callback Payment State

The `callbackPaymentState` parameter is a function that returns the current payment process. Sending this function is not mandatory if you do not need the result. The possible states are:

```Kotlin
const val PAYMENT_STATE_SUCCEEDED = "SUCCEEDED"
const val PAYMENT_STATE_FAIL = "FAIL"
const val PAYMENT_STATE_PROCESSING = "PROCESSING"
const val PAYMENT_STATE_REJECT = "REJECT"
const val PAYMENT_STATE_INTERNAL_ERROR = "INTERNAL_ERROR"
const val PAYMENT_STATE_STATE_CANCELED_BY_USER = "CANCELED"
```



#### Show Payment Methods

When implementing the Full  SDK version, you need to add the following view on your layout to show the available payment methods:

```XML
<com.yuno.payments.features.payment.ui.views.PaymentMethodListView
        android:id="@+id/list_payment_methods"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```



#### Start Payment

To start a payment process, you have to call the method `startPayment`. However,  if you are using the lite version of the SDK, you must call the method `startPaymentLite`.

###### Full SDK Version

```Kotlin
startPayment(
callbackOTT: (String?) -> Unit, //Optional - Default null
)
```



###### Lite SDK Version

```Kotlin
startPaymentLite(
    paymentSelected: PaymentSelected,
    callbackOTT: (String?) -> Unit, //Optional - Default null
)
```



For the Lite version, you need to send an additional parameter, which is the vaulted token and/or the payment type that the user selected to make the payment.

```Kotlin
PaymentSelected(  
    id: "payment_vaulted_token",  
    type: "payment_type",  
)
```



At the end of this process, you will obtain the OTT, which is a required parameter to create a payment via the Payments API POST/payments. You can obtain this data from to `onActivityResult` explained in the callback section.

#### Complete Payment

If in the create_payment response the sdk_action_required parameter is true you need to call the following method:

```Kotlin
continuePayment(
    showPaymentStatus: Boolean, //Optional - Deault true
    callbackPaymentState: ((String?) -> Unit)?, //Optional - Deault null
)
```



To show your own payment status screens, you should send `false` in the `showPaymentStatus` parameter and then get the payment state by callback.

## Styles

### Button Styles

If you want to use your own button styles, you can override our styles. An example is presented in the code snippet below:

```XML
<style name="Button.Normal.Purple">
        <item name="android:background">your color</item>
    </style>
```



> These are our button styles you can override:  
> -Button.Normal.White  
> -Button.Normal.Green  
> -Button.Normal.Purple  
> -Button.Normal.Purple.Big
