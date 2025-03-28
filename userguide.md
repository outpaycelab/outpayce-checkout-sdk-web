# Outpayce Checkout Web SDK

Checkout Web SDK is a library that makes payment experience easy. It helps you to create a payment form and to include it in any web application seamlessly.

You can then accept payment using our UI components inside your web app. The payment is handled by the SDK and done through Amadeus checkout servers. Notifications will be sent back to your web app through callbacks to notify you with the status.

## Table of Content

*  [Prerequisites](#prerequisites)
    *  [Payment Platform](#payment-platform)
    *  [Content-Security-Policy](#content-security-policy)
*  [How does it work?](#how-does-it-work)
*  [Step 1: Integration](#step-1-integration)
*  [Step 2: Initialize and configure the SDK](#step-2-initialize-and-configure-the-sdk)
*  [Step 3: Handle callbacks and get available methods of payment](#step-3-handle-callbacks-and-get-available-methods-of-payment)
    * [Handle Option 2 : methods selection is done in your page](#handle-option-2-methods-selection-is-done-in-your-page)
*  [Step 4: Currency conversion (if applicable)](#step-4-currency-conversion-if-applicable)
*  [Step 5: Trigger payment](#step-5-trigger-payment)
*  [Step 6: After redirection](#step-6-after-redirection)
*  [Step 7: Check payment status](#step-7-check-payment-status)
*  [Customization](#customization)
*  [Access to assets](#access-to-assets)
*  [Device fingerprint](#device-fingerprint)

## Prerequisites

### Payment Platform

You need to be registered as a customer of the [Amadeus Payment Platform](https://amadeus.com/en/business-function/payments/xchange-payment-platform) in order to use this service and accept payments.

Before starting the checkout process, you need to initialize it from your server. To do so, your server has to call the POST `payment/checkout-forms` rest webservice, with your merchant identifier, the currency and the amount to be paid (if applicable). In response you will get a PPID, that is a unique identifier of the transaction.

### Content-Security-Policy

Checkout Web SDK relies on iframes to perform card payments, and may also relies on third party services for 3DS2 authentication and fraud data collection.
If applicable, ensure your website security policies ( `Content-Security-Policy` ) are permitive enough.

**Required policies for any usage:**
 - allow scripts from *https://\*.amadeus.com/*  : `script-src https://*.amadeus.com/`
 - allow inline styles and styles from *https://\*.amadeus.com/* : `style-src 'unsafe-inline' https://*.amadeus.com/`
 - allow frames from *https://\*.amadeus.com/* : `frame-src https://*.amadeus.com/`

**Required policies for 3DS2 authentication:**
 - allow scripts from *https://\*.cardinalcommerce.com/* : `script-src https://*.cardinalcommerce.com/`
 - allow frames from any domain : `frame-src *`

**For fraud data collection**, required policies vary depending on the fraud provider and the way it's implemented.
The required policies are usually the following ones – for detailed information, contact your fraud provider:
 - allow scripts from data collector domain
 - allow inline scripts and string evaluation may be required : `script-src 'unsafe-eval' 'unsafe-inline'`
 - allow frames from data collector domain may be required

## How does it work?

With these very few steps you can collect payments in your web application:
 1. Include Checkout SDK in your page.
 1. Initialize the SDK and specify your configuration.
 1. The SDK will then fetch the payment methods available, then you can:
     - Option 1: Integrates the payment method selection component, to let the SDK handle the payment method selection and the payment.
     - Option 2: Integrates directly the component of the selected payment method, to handle the payment method selection on your side, and let the SDK handle the payment.
 1. Once user enters the information needed for payment, you can trigger payment collection step.
 1. The SDK performs the payment and notify your page with the result. You can then handle the result.

### Step 1: Integration

Include the following scripts and CSS in your web page.

```HTML
<script src="https://prd.payment.amadeus.com/checkout/sdk/5.0.0/sdk-es2015.js" integrity="sha384-cuqVh/0A75K/ZscZnqQx5o3z0YVfNFP9NJ/ksLBJP1QZjtbEyzMCNCat6H9pv1GZ" crossorigin="anonymous"></script>
<link rel="stylesheet"  href="https://prd.payment.amadeus.com/checkout/sdk/5.0.0/sdk.css" integrity="sha384-xlGmvFOWqf14lJgvyxcMcf+Ljs4mehLYI4+ekJ6hWAhzyKZjzypBDXSnq/gbpewC" crossorigin="anonymous">
```

If your website is based on Angular Material, you can use this alternate CSS file, that is lighter as it doesn't contain Material rules:
```HTML
<link rel="stylesheet"  href="https://prd.payment.amadeus.com/checkout/sdk/5.0.0/sdk-no-material.css" integrity=" sha384-vY2CCgIirbneo+gzFqW6xtfhljUKyTq94D7uZKpSlvG3JLuFVadMr4a/G3G+bz0F" crossorigin="anonymous">
```

If the Checkout SDK components are inserted inside a shadow DOM, the above CSS file will have to be inserted in the corresponding ShadowRoots instead.

Use the following link to get the sha384 checksum of each file: https://prd.payment.amadeus.com/checkout/sdk/manifest.json .



### Step 2: Initialize and configure the SDK

Create a section in your page that will contain :
- Option 1: the list of methods of payment available and their UI component.
- Option 2: the pre-selected method of payment's UI component.

For example:

```HTML
<div  id="payment-form-container"></div>
```

Then initialize the SDK like follow :

```Javascript
let SDKConfiguration = {
    container: 'payment-form-container',  // Mandatory
    ppid:'XXXXXXXXXXXXXXXX',              // Mandatory: returned by PreparePaymentPage service
    environment :  'prd',                 // Mandatory
    locale:        'en-GB',               // Mandatory
    options : {                           // Optional
        // See options below
        // If you want the SDK to display only one selected method, set mopSelectionInSdk to true
    }
}
let checkout = new AmadeusCheckout(SDKConfiguration);
```

__SDKConfiguration properties:__

|Name|Type|Default|Description|
|---|---|---|---|
|`container`| String or HTMLElement | *Mandatory* | HTML id of the DOM element, or directly the DOM element, that will contain the checkout component. |
|`ppid`| String | *Mandatory* | Unique id returned by the PreparePaymentPage service. |
|`environment`| Enumeration: `uat`, `pdt`, `mig`, `frt`, `skl`, `brd`, `prd` | *Mandatory* | Environment where the PPID was generated from. Not case sensitive|
|`locale`| String | *Mandatory* | The shopper's locale. This is used to set the language and to format amounts. |
|`options`| `AmadeusCheckoutOptions` | `{}` | Object containing SDK options, cf. below table. |

__`AmadeusCheckoutOptions` properties:__

|Name|Type|Default|Description|
|---|---|---|---|
|`mopSelectionInSdk`|Boolean|`true`|`true`: corresponds to option 1: the SDK displays the list of available methods of payment and their components. `false`: corresponds to option 2: the mop selection is done before and the SDK displays only the selected mop's component.|
|`showSecurityHelper`|Boolean|`true`| Show the PCI compliancy icon and text in the Payment Card component. |
|`collectCardholderName`|Boolean|`true`| Set to `false` to deactivate cardholder name collection by Payment Card component. |
|`showVendor`|Boolean|`true`|If set to `true`, user will have a manual selection of the card vendor. Otherwise, the detection of the vendor will be done dynamically after the user types the 8 first digits of the card number.|
|`matAppearance`|Enum: `fill`,`outline` |`outline`|The appearance of the form fields with material design: Look at images below.|
|`matFloatLabels`|Enum: `auto`, `always`|`auto`|`auto` : Make the label float when the user click in the field. `always`: Make the label fixed on top of the field.|
|`cardNumberSeparator`|Enum: ` `(space character), `-`|space character|The separator between digit blocks in card number formatting. For example for a card number with: `space` : 4444 3333 2222 1111. with `-`: 4444-3333-2222-1111.|
|`overridenLabels`| Object | `{}` |SDK provides translations for several languages. Use this option to override any label from the default translations. Eg. `{"label_cvv":"Security Code", "label_city":"City"}` . |
|`displayPlaceholders`|Boolean|`false`|If set to `true`, the card details form fields will have placeholders. Otherwise, the only card details form field having placeholder will be the expiry date.|
|`collectCardExpiryDate`|Boolean|`true`| If set to `false`, the expiry date selector will not be shown. |
|`showSaveCardToggle`|Boolean|`false`| If set to `true`, a toggle is added after the card information that user can turn on to have its card saved. This information can then be retrieved via the GET checkout-forms. |
|`preferredCountryCodes`|Array\[String\]|`[]`| List of country codes to be displayed on top of the country list in billing address form and contact form.|
| `cardholderNameRegex` | String | `""` | A regex pattern for validating the cardholder name input. If empty, no validation is applied. |
|`billingAddressOrder`|Enum:`default`,`US_BILLING_ADDRESS_ORDER` |`default`| Specifies the template for billing address form as a single string.
|`deniedCardIssuingCountries`|Array\[String\]|`[]`| List of country codes which issuing cards are blocked.|
|`collectBillingAddressForCountries`|Array\[String\]|`[]`| List of country codes for which billing address is collected. If the option is not provided, billing address can be collected for any country.|
|`forceChangeDetection`| `Boolean` |`false`| /!\ Not to be used in a standard use case /!\ In case your framework is messing up with application lifecycle, use this option to force change detection on CVV and PAN field. |
|`emitCardFormDetails`|Boolean|`false`|If set to `true`, the onReady and onChange callbacks will be enriched with card form details information. See `cardForm` key in below documentation|
|`emitContactFormDetails`|Boolean|`false`|If set to `true`, the onReady and onChange callbacks will be enriched with contact form details information. See `contactForm` key in below documentation|
|`paymentMethodSpecificOptions.applePay`| `AmadeusCheckoutApplePayOptions` |`undefined`| Apple Pay specific options. See below for more details. |
|`maskCVV`|Boolean|`false`|If set to `true`, the CVV value will be masked when it is being typed by the user, otherwise it will be visible. |
|`iframeStyle`| {[key: string]: string} | `{}` | Allows for the application of custom CSS styles directly to payment form fields within iframes. This object accepts key-value pairs where the key is a CSS property (in camelCase format) and the value is the style setting. You can customize aspects like font size, text color, placeholder color, and text alignment, ensuring the payment form aligns seamlessly with the visual aesthetics of your website or application.
|`showWalletAccountDetails`|Boolean|`true`| If set to `false`, after the user login, the wallet information section should not be displayed |


__`AmadeusCheckoutApplePayOptions` properties:__

|Name|Type|Description|
|---|---|---|
|`sheetDisplayName`|String|Set this property to override the canonical name for your store. Apple recommands not to localize it. [More details](https://developer.apple.com/documentation/apple_pay_on_the_web/apple_pay_js_api/requesting_an_apple_pay_payment_session).
|`paymentRequest`|`ApplePayPaymentRequest`|Set this property to enrich the `ApplePayPaymentRequest`. The provided object is automatically merged with the one generated by Checkout. [More details](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest).|

Example:

```Javascript
// All keys are optional:
options.paymentMethodSpecificOptions.applePay = {
    sheetDisplayName:'My Store',         // This label is displayed on MacBook Pro Touch Bar.
    paymentRequest: {
        total: {label:'Total Amount'},   // Label for the total amount on the Apple sheet
        lineItems: [                     // Amount breakdown on the Apple sheet
            {label:'Flight 1', amount:'100.00'},
            {label:'Flight 2', amount:'120.00'}
        ]
    }
};
```

__MatAppearance options:__

![matappearance](doc/img/matappearance.png)

### Step 3: Handle callbacks and get available methods of payment:

The SDK communicates with your page through callbacks that you need to handle depending on your use case before running the SDK, you can refer to the example below.

Once you have implemented all the callbacks listed below, call `AmadeusCheckout.start()` to start the SDK.

```Javascript
checkout.onReady = function(mopsList, sessionTimeout, paymentDetails) {
    // Called as soon as the SDK as retrieved the list of available method of payment
};
checkout.onChange = function(change) {
    // Called when there is a change on: the selected method, the fees, the card bin or the validity of the form
    //This method is deprecated and will be replaced by onPaymentMethodsChange
};
checkout.onPaymentMethodsChange = function(change) {
    // Called when there is a change on: the selected methods of payment, the fees, the cards bin or the validity of the form
    // replace the deprecated onChange method
};
checkout.onRedirectionNeeded = function(mopId, redirectionRequest) {
    // Called before the SDK redirects the user to an external page
}
checkout.onSuccess = function(mainMopId, successData) {
    // Called when the payment is successful
};
checkout.onError = function(mopId, errorDetails) {
    // Called on failure or on error
};
checkout.delegateServerCall = function(actionTokenList, delegationType) {
    // Called when there is a payment transaction that is delegated to calling application.
    // For standard integrations this callback can be ignored.
};
checkout.onApplePaySessionCreation = function() {
    // Called when the user has decided to pay via apple pay, to pass an ApplePayPaymentRequest, which would override some ApplePay data [Currently this only supports "supportedNetworks" field in ApplePayPaymentRequest]
};

// Starts the SDK, and insert payment component in the container:
checkout.start();
```

__*AmadeusCheckout.onReady(mopList, sessionTimeout, paymentDetails)*__

*onReady* is called once the SDK is fully initialized and ready to be used.

It will provide you the list of methods of payments available per your configuration.

If you set session timeout in the prepare payment page message sent from your server, this callback will provide you the time left in seconds before the `ppid` expires, otherwise this value will be `null`. `sessionTimeout` is only sent for informative reasons, the SDK will handle it accordingly and will inform you with an error once expired and it will close, no more actions will be possible.

Parameters:
- `mopsList`: Array of Objects `[{id, type, name, displayableName}]`, having the following properties:
  - `id`: String. Id of the payment method. Used to indentify this method in the different callbacks and functions.
  - `type`: String. Type of payment method (eg. `AmopPaypal`). Used as a unique identifier for each payment method type (see table in appendix).
  - `displayableName`: String. Displayable name, only defined for alternative payment methods (eg. `PayPal`).
  - `hasFees`: Boolean. Indicate whether OB fee is applicable or not.
  - `fees`: Object[] (Optional), representing the surcharge fees information (for now, only available for applepay). It contains the following properties.
    - `cardType` : string. Indicate the type of card for which the fees is to be applied on.
    - `feeAmount` : string. Indicate the surcharge fee amount.
    - `feePercent` : string. Indicate the surcharge fee percent.
    - `network` : string. Indicate the card network for which the fees is to be applied on.
  - `isConfigValid`:Boolean. Indicate wether the payment method configuration is valid or not.
  - `hasInstallments`: Boolean. Indicate whether installments is applicable or not.
  - `name`: String.
  - `needUserInput`: Boolean. Indicate whether some user input is needed for that mop. Can be used to determine if the creation of a specific component ( using checkout.create() ) is needed for this payment method.
  - `vendors`: String[]. List of vendor/coBadge ids allowed for the card. For prefilled cards, contains only the prefilled vendor.
  - `merchantVendorCodes`: {code: String; id: String}[]. List of vendors allowed for the card associated to the vendor code defined in the merchant configuration (eg. `{code: 'VI', id:'visa'}`).
  - `cardBinDetails`: Object representing the bin information of the prefilled card. Please refer to onChange parameter `cardBin/details` for content information.
  - `cardFormDetails`: Object representing the card form information. Only present if `emitCardFormDetails` set to true. Please refer to onChange parameter `cardForm/details` for content information.
  - `maxOccurences`: Number. Specifies the maximum number of times a payment method can be used.
- `sessionTimeout`: Number or Null
- `paymentDetails`: Object (optional) representing the initial payment information
  - `amountDetails`: Object (optional), with the following properties:
    - `precision`: Number of decimal of the currency, e.g. 2
    - `currency`: String. Currency ISO code, e.g. 'EUR'
    - `amount`: string. Amount as string with decimals, e.g. '10.00'
  - `mcpOffers `: Object[] (Optional). Array of McpOffer, containing the following properties:
    -  `currency`: String. Target currency.
    -  `amount`: String. Amount converted in target currency.
    -  `margin`: String. Conversion margin.
    -  `rate`: String. Conversion rate.
  - `maxMops`: String (optional) representing the number of maximum mops authorized
    
__*AmadeusCheckout.onChange(status)*__
The *onChange* method is deprecated and will be replaced by *onPaymentMethodsChange*. It was previously called in the following cases:
- A new method of payment is selected
- The form validity has changed, `AmadeusCheckout.pay()` will effectively trigger the payment only if the form is valid
- The BIN of the entered Payment Card has changed
- The applicable fees for the selected payment method has changed
- The loading state has changed

Parameter:
- `status`: Object, with the following properties:
  -  `mopId`: String. Id of the selected payment method, Null if no method is selected.
  -  `isValid`: Boolean. `true` if form is properly filled, `false` otherwise.
  -  `isLoading`: Boolean. `true` if a server query is ongoing, `false` otherwise. You can use this value to display a loading spinner on the page.
  -  `cardBin`: Object, with the following properties (or Null if not filled or irrelevant):
     - `vendor`: String. Vendor/CoBadge of the card that will be used for the payment (eg. `visa`).
     - `bin`: String. 6 first digits of the card (eg. `444433`).
     - `details`: Object. with the following properties:
        - `issuingCountry`: String. Issuing country of the card (e.g. `US`).
        - `cardType`: String. Credit card type (e.g. `CR` stands for credit card, `DB` stands for debit card, `CH` stands for charge card).
        - `isCorporate`: Boolean. Is the card a corporate card or not.
        - `currency`: String. Currency of the bin range (e.g. `USD`).
        - `coBadges`: String[]. Array containing all supported vendors for the card (e.g. [`visa`, `mastercard`]).
        - `bin`: String. Bank identification number which is only first 6 digits of the card (eg. `444433`).
        - `extendedBin`: String. Bank identification number in case of 7 or 8 digits.
  -  `cardForm`: Object representing the card form information. Only present if `emitCardFormDetails` set to true. with the following properties (or Null if not filled or irrelevant):
     - `focusedField`: String. Id of the card form field currently receiveing the focus (or null if no field focus). Possible values: `pan`, `expiry`, `cvv`, `name`.
     - `details`: Object. with the following properties:
        - `concealedCardNumber`: String. Concealed Card Number: 6 first digits in clear for manually entered card; 4 last digits in clear for prefilled cards.
        - `concealedCvv`: String. Concealed CVV: All digits will be replaced by 'X'.
        - `expiryDate`: String. Expiry date of the card - format MM/YY.
        - `cardholderName`: String. Cardholder Name.
        - `vendor`: String. Vendor/CoBadge id selected by the user (eg. `visa`) - may be invalid.
        - `tdsProvider`: String. The 3ds provider corresponding to the vendor id (e.g. `visa`).
        - `lastDigits`: String. The last 4 digits of the card (e.g. `1111`).
        - `errors`: Object representing the form errors. Only present if at least one error. with the following properties:
            - `isExpired`: Boolean. `true` if the expiry date in the past, `false` otherwise.
            - `isExpiryTooFar`: Boolean. `true` if the expiry date too far in the future, `false` otherwise.
  -  `fees`: Object, with the following properties (or Null if no fees apply):
     -  `feesAmount`: String.
     -  `status`: Enum `succes`, `failure`, `pending`.
     -  `errorMessage`: String, filled only if SDK was not able to compute fees.
  - `mcpOffers`: Object[]. Array of objects, containing the following properties:
     -  `currency`: String. Target currency.
     -  `amount`: String. Amount converted in target currency.
     -  `margin`: String. Conversion margin.
     -  `rate`: String. Conversion rate.

__*AmadeusCheckout.onPaymentMethodsChange(status)*__

*onPaymentMethodsChange* should be implemented in place of the now deprecated *onChange* . It is called in the following cases:
- A new method of payment is selected
- The form validity has changed, `AmadeusCheckout.pay()` will effectively trigger the payment only if the form is valid
- The BIN of the entered Payment Card has changed
- The applicable fees for the selected payment method has changed
- The loading state has changed

Parameter:
- `status`: Object, with the following properties:
  -  `isValid`: Boolean. `true` if the form for each payment method is properly filled, `false` otherwise.
  -  `isLoading`: Boolean. `true` if a server query is ongoing, `false` otherwise. You can use this value to display a loading spinner on the page.
  -  `paymentMethods`: Object, with the following properties:
        -  `isValid`: Boolean. `true` if form is properly filled, `false` otherwise.
        -  `cardBin`: Object, with the following properties (or Null if not filled or irrelevant):
            - `vendor`: String. Vendor/CoBadge of the card that will be used for the payment (eg. `visa`).
            - `bin`: String. 6 or 7 or 8 first digits of the card representing bank identification number (eg. `444433`).
            - `details`: Object. with the following properties:
            - `issuingCountry`: String. Issuing country of the card (e.g. `US`).
            - `cardType`: String. Credit card type (e.g. `CR` stands for credit card, `DB` stands for debit card, `CH` stands for charge card).
            - `isCorporate`: Boolean. Is the card a corporate card or not.
            - `currency`: String. Currency of the bin range (e.g. `USD`).
            - `coBadges`: String[]. Array containing all supported vendors for the card (e.g. [`visa`, `mastercard`]).
            - `extendedBin`: String. Bank identification number in case of 7 or 8 digits.
        -  `cardForm`: Object representing the card form information. Only present if `emitCardFormDetails` set to true. with the following properties (or Null if not filled or irrelevant):
            - `focusedField`: String. Id of the card form field currently receiveing the focus (or null if no field focus). Possible values: `pan`, `expiry`, `cvv`, `name`.
            - `details`: Object. with the following properties:
                - `concealedCardNumber`: String. Concealed Card Number: 6 first digits in clear for manually entered card; 4 last digits in clear for prefilled cards.
                - `concealedCvv`: String. Concealed CVV: All digits will be replaced by 'X'.
                - `expiryDate`: String. Expiry date of the card - format MM/YY.
                - `cardholderName`: String. Cardholder Name.
                - `vendor`: String. Vendor/CoBadge id selected by the user (eg. `visa`) - may be invalid.
                - `tdsProvider`: String. The 3ds provider corresponding to the vendor id (e.g. `visa`).
                - `lastDigits`: String. The last 4 digits of the card (e.g. `1111`).
                - `errors`: Object representing the form errors. Only present if at least one error. with the following properties:
                    - `isExpired`: Boolean. `true` if the expiry date in the past, `false` otherwise.
                    - `isExpiryTooFar`: Boolean. `true` if the expiry date too far in the future, `false` otherwise.
                    - `isTokenInvalid`: Boolean. `true` if prefilled card does not have bin information , false otherwise.
        -  `contactForm`: Object representing the contact form information. Only present if `emitContactFormDetails` set to true (and contactDetails set to true in the profile manager mop-config) with the following properties (or Null if not filled or irrelevant):
            -  `email`: String.
            -  `phone`: String.
        -  `fees`: Object, with the following properties (or Null if no fees apply):
            -  `feesAmount`: String.
            -  `status`: Enum `succes`, `failure`, `pending`.
            -  `errorMessage`: String, filled only if SDK was not able to compute fees.
        - `mcpOffers`: Object[]. Array of objects, containing the following properties:
            -  `currency`: String. Target currency.
            -  `amount`: String. Amount converted in target currency.
            -  `margin`: String. Conversion margin.
            -  `rate`: String. Conversion rate.
        - `installment`: Object with the following properties:
            -  `id`: String. Id.
            -  `count`: Number. Count.
            -  `first`: AmountInformation. First Amount.
            -  `subsequent`: AmountInformation. Subsequent Amount.
            -  `total`: AmountInformation. Total Amount.
            -  `interest`: AmountInformation. interest Amount.
        - `walletForm`: Object with the following properties:
            -  `accountBalance`: Number. Balance of the wallet account.
            -  `selectedAmount`: Number. Amount selected by the user.
        - `rewardForm`: Object, with the following properties:
            - `rewardPointsUsed`: String. Number of miles used.
            - `equivalentAmount`: String. The equivalent amount for the miles used.

__*AmadeusCheckout.onRedirectionNeeded(mopId)*__

*onRedirectionNeeded*  is called before the SDK redirects the user to an external page.

You can use this callback to store your context, or to display a redirection message to user.

If the returned value of this callback is a `Promise`, the SDK will wait until it is settled before redirecting the user.

Otherwise, user will be redirected right after.

Parameter:
- `mopId`: String. Id of the payment method.

__*AmadeusCheckout.onSuccess(mainMopId, successData)*__

*onSuccess* is called once the authorization has been successfully completed.

The `mainMopId` parameter is the Id of the main payment method that was used for the transaction. In a payment with miles and another payment method, it will be it Id of the other payment method.

You can proceed with a server to server call to retrieve information about the payment. The SDK will close and no more action will be possible.

Parameter:
- `mopId`: String. Id of the main payment method.
- `successData`: Object, with the following properties (or undefined if not relevant)
    - `redirectUrl`: String
    - `recordLocator`: String
    - `appliedFees`: Number
    - `orderId`: String
    - `paymentMethods`: Array of objects, containing the following properties:
        - `mopId`: String. Id of the payment method.
        - `paymentRecordID`: String
        - `approvalCode`: String
        - `externalId`: String
        - `paymentMerchantReference`: String
        - `pspReconciliationReference`: String

__*AmadeusCheckout.onError(mopId, errorDetails)*__

*onError* is called when there is an error and depending on the details your action will be required. `mopId` is the Id of the method of payment that was used when the error occurred.

It can be an `empty string` if this information couldn't be retrieved or if no method was being used.

`errorDetails` gives you the information you need about what happened and the action to be done on your side.

Parameters:
- `mopId`: String.  Id of the payment method.
- `errorDetails`: Object, with the following properties:
  - `nextAction`: `ActionType` enum. Cf. below list.
  - `errorType`: Enum `ErrorType`. Cf. below list.
  - `paymentStatus`: Enum `StatusType`. Cf. below list.

Types:
- Enum `ActionType`:
    - `Retry` : the user can retry the payment and you could display an error in your page depending on `errorType`.
    - `CheckStatus`: the SDK will close, no more action can be done with the same context. You need to check the status with a server to server call using the `ppid` to define what can be done. In most cases you need to regenerate a new `ppid` if you want to start a new payment and restart the initialization of the SDK.
    - `Reset` : The ppid timeout was reached while no payment was done or payment failed and user could retry but they didn't. In this case the `ppid` can't be used anymore and you need to generate a new `ppid` and restart the initialization of the SDK if you want the user to retry.
 - Enum `ErrorType`:
    - `Network` : When the SDK reaches a timeout waiting for a response from the server.
    - `SessionTimeout`: When the `ppid` has expired.
    - `PaymentError`: When authorization has failed.
    - `Canceled`: Informs you that the user aborted the payment after a payment request was sent. Nothing needs to be done on your side.
    - `Unexpected`: An unexpected error has occurred.
- Enum `StatusType`:
    - `Success`: payment was successful.
    - `Failure`: payment has failed.
    - `Unknown`: payment status is unknown and needs to be checked with a server to server call.
    - `Cancel`: payment was canceled.

__*AmadeusCheckout.delegateServerCall(actionTokenList, delegationType)*__

*delegateServerCall* is called when a server transaction has to be processed by the host application, instead of the Checkout server.
In a standard implementation, this callback does not need to be implemented and can be ignored.

*Return value:* a `Promise` has to be returned, it should be resolved as soon as the transaction is successfully processed, or be rejected if it failed.

Parameters:
- `actionTokenList`: Array. Array of objects, containing the following properties:
    - `mopId`: String.  Id of the payment method. It may be `undefined` when the SDK is reloaded after a redirection.
    - `actionToken`: String. Transaction ID that has to be transmitted, alonside with the PPID, in the payment transaction request.
- `delegationTye`: Enum `DelegationType`. Type of delegation transaction. Cf. below list.

Types:
- Enum `DelegationType`:
    - `CreateFop`: a FOP creation request is expected to be sent.
    - `ValidateFop`: a FOP validate request is expected to be sent.

__*AmadeusCheckout.onApplePaySessionCreation()*__

*onApplePaySessionCreation*  is called when the user has decided to pay via apple pay, to pass an ApplePayPaymentRequest, which would override some ApplePay data [Currently this only supports "supportedNetworks" field in ApplePayPaymentRequest].
For regular ApplePay Integration(Integration in which Network Selection option is not given to the user) this callback can be ignored.

This is a callback that can be used to override some parameters of ApplePayPaymentRequest.
Only the list of supported vendors can be overriden for now and the overriden list must be a subset of the Checkout configuration.
This is particularly useful in the use-case where addtional fees is applicable on a card payment, and hence the vendor selection must be done before triggering the applepay payment sheet. In this case, the information of the selected vendor can be passed here.
[More details on the ApplePayPaymentRequest](https://developer.apple.com/documentation/apple_pay_on_the_web/applepaypaymentrequest)

The return value of this callback is expected to be a valid ApplePayPaymentRequest with supportedNetwork fields. If the supportedNetwork field value is not valid, SDK will initiate Apple Pay Session with the default networks which are configured.
```Javascript
// Once user has Selected a Network and Click on Pay button, calls onApplePaySessionCreation() to return the selected network via an ApplePayPaymentRequest with supportedNetwork fields:
checkout.onApplePaySessionCreation = function() {
  var newApplePayPaymentRequest  = {
    supportedNetworks: ["visa"], //User selected Network
  }
  return newApplePayPaymentRequest;
};
```


##### Handle Option 2 : methods selection is done in your page

In case the payment method selection is handled on your side (`mopSelectionInSdk` set to `false`), use the payment method list returned in the `onReady` callback.

Create the component for a specific payment method with `checkout.create(containerId, mopId)` function.This method returns a string representing the componentId associated with the payment method, which can be used for further interactions with the component:
```Javascript
// - `container` can be either a String containing a HTML id or a HTMLElement
// - `mop` can be any element of the `mopsList` sent in `onReady` callback
const componentId =  await checkout.create(container, mop.id);
```


Once the payment method is selected, ensure its component is displayed, and use `AmadeusCheckout.selectMopComponent(componentId)` to receive appropriate status updates.
The componentId passed to selectMopComponent() is the one returned by the create() function.
```Javascript
// Once user has selected a method of payment, calls selectMopComponent() with its componentId:
checkout.selectMopComponent(selectedComponentId);
```
If the user selects another payment method, you can call again `AmadeusCheckout.selectMopComponent(componentId)`.

For a payment with multiple Methods of Payment, you can use the following helper function:
__*AmadeusCheckout.selectMopComponents(mopSelections)*__

The `selectMopComponents` function is used to select the Methods of Payment to be used for a payment with multiple payment methods.

Parameters:
- `mopSelections`: Array. Array of objects, containing the following properties:
    - `componentId`: String.  Id of the payment method component.
    - `amount`: String. Amount used for the selected payment method. This property is optional (full amount considered if not present). We strongly advise to always provide it in case of multiple method of payments.

Please note that to avoid too many server calls (for obfees), a debounce timer of 300ms is present on the selectMopComponent(s) functions.
Extra focus should still be put on the frequency this method is called to avoid unecessary computations.

For a payment that includes QBRPoints or miles without slider UI, you can call this helper function to pass the loyalty informations:
Please note that before using `setLoyalty` the `componentId` should be created using `create` and pass that componentId as input parameter to the `setLoyalty`.
__*AmadeusCheckout.setLoyalty(componentId, loyaltyData)*__

Parameters:
- `componentId`: String.  Id of the payment method component.
- `loyaltyData`: Object containing either `qbrLoyalty` or `milesLoyalty`
    - `qbrLoyalty`: Object containing the following properties:
        - `accountNumber`: String.  QBRLoyalty Account Number.
        - `program`: String. QBRLoyalty Program.
        - `numberOfPoints`: String. Number of QBRPoints used for payment.
        - `convertedAmount`: String. Equivalent amount of selected QBRPoints.
        - `currency`: String. Type of currency for payment.
        - `membershipId`: String. It is optional field and contains the frequent flyer number or the membership Identifier.
        - `sliderRanks`: Number. Contains the maximum rank.
        - `sliderSelection`: Number. Contains the specific rank for slider.

    - `milesLoyalty`: Object containing the following properties:
        - `membershipId`: String. It is optional field and contains the frequent flyer number or the membership Identifier.
        - `membershipStatus`: string. Contains the miles membership status.
        - `routingVendor`: string. Contains the miles routing vendor.
        - `sliderRanks`: Number. Contains the maximum rank.
        - `sliderSelection`: Number. Contains the specific rank for slider.

Please note that for some alternative payment methods, the payment component is empty. As soon as they are selected, the status ot the form will be set to valid, and you will be able to call `AmadeusCheckout.pay()` to trigger the payment.

### Step 4: Currency conversion (if applicable)

If a currency conversion offer is available (refer to `mcpOffers` field in `onChange` callback), the following function can be called to set the target currency:
```Javascript
checkout.selectCurrency(currencyCode);
```

### Step 5: Trigger payment

Once the form is valid (`onChange` called with `isValid` set to `true`) you can trigger payment in the checkout SDK by calling the function `pay`.

If you call this function on an invalid form, it will display an error below the incorrect fields.

```Javascript
checkout.pay(); // Trigger payment
```

### Step 6: After redirection

After a 3DS authentication or an alternative payment method redirection, Amadeus Checkout will redirect the user back to your page.

It will use the `postredirection` url you have provided in the message `POST payment/checkout-forms`, with the following format:
```Javascript
`${PostRedirection_URL}?PPID=${PPID}`
```

To finalize the payment, and get its status, you have to repeat the Step 2: reinitialize the SDK using the same PPID.


### Step 7: Check payment status

When payment is completed, perform a server to server call with the message `GET payment/checkout-forms/{PPID}` to get the status of the authorisation and other information about it.

## Customization

For possible customizations through SDK, the list of options is provided in the section [Step 2](#step-2-initialize-and-configure-the-sdk).

For additional CSS customizations, you can override some CSS attributes of the SDK in your web page CSS file.

## Access to assets

The SDK provides you a function to retrieve the url(as a `string`) of available icons for vendors, secure vendors and alternative methods of payment, or `null` if no icon was found corresponding to the input. You can then use the url provided to display some icons in your page if needed.

```Javascript
const myIconUrl = checkout.getIconUrl(iconType, iconName); //string or null
```

`getIconUrl` takes as arguments:
- Enum `IconType`:
    - `vendor` for vendor ids (such as `visa`, `diners`, `mastercard`...)
    - `secure-vendor` for schemas that supports 3DS (such as `visa`, `mastercard`, `amex`...)
    - `amop` for alternative methods of payment (such as `paypal`, `bancontact`)
- String `IconName` : can be retrieved for each mop in the list of payment methods provided in the callback `onReady`, by accessing `mop.name` for amops, and by using available vendors for credit card mops.

## Device fingerprint

You can enable device fingerprint collection through our SDK if you have one of the three fraud partners that we support (Cybersource, Accertify and RED). A setup needs to be done before you can use this service where you need to provide us parameters such as your `provider`, your `merchantId` and `thirdPartyHostname` in case of Cybersource or Accertify.

Please be aware that if you enable this feature, the SDK will add the `Javascript` files of the provider in your page according to their implementation guides. We cannot control what these scripts do, same with any security issues that they might introduce. The files that are added will be in your page until a reload or change of the page url. Some browsers may block them since they collect information about the user behavior etc. For more information about this, please check the documentation of your fraud provider.

These scripts are limited to one call per page until you reload the page.
If your provider is Cybersource or Accertify, the SDK will provide a device fingerprint id to the scripts of the provider and then send it to their backend at payment time. In case you restart the SDK with another `ppid` without reloading the page, the same device fingerprint id will be used, which may cause a problem in case of succesfull payment. This needs to be checked with your fraud partner.
If you restart the SDK with a different `merchantId` or `thirdpartyhostname` without reloading the page, you will receive an error through the callback `onError` with `errorType`: `FingerprintParamsChanged`. This will not block the user but no device fingerprint will be used.
If your provider is RED, we retrieve the id through their script and forward it to them.

If the provider is different from the three currently supported by the SDK, you will receive an error through the callback `onError` with `errorType`: `FingerprintUnknownProvider`. The user will not be blocked but no device fingerprint will be used.

## Appendix

### Table of Payment Methods

|type                    |displayableName             | Remark                                                                                                                                                |
|------------------------|----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| Amop                   | `undefined`                | This type is used for alternative payment methods that are not yet known by Checkout SDK. The `name` attribute can be used as alternative identifier. |
| Amop2C2PInstallment       | 2C2P Installment Payment         | Installment Payment Processed by PSP 2C2P                                                                                                                                                      |
| AmopAirtel             | Airtel                     |                                                                                                                                                       |

| AmopAlipay             | Alipay                     |                                                                                                                                                       |
| AmopAlma               | Alma                       |                                                                                                                                                       |
| AmopAmazonPay          | Amazon Pay                 |                                                                                                                                                       |
| AmopApplePay           | Apple Pay                  | Apple Pay method works without a full page web redirection                                                                                            |
| AmopBancontact         | Bancontact                 |                                                                                                                                                       |
| AmopBillease           | BillEase                   |                                                                                                                                                       |
| AmopBizum              | Bizum                      |                                                                                                                                                       |
| AmopBlik               | Blik                       |                                                                                                                                                       |
| AmopBoost              | Boost                      |                                                                                                                                                       |
| AmopCartesBancaires    | Cartes Bancaires           |                                                                                                                                                       |
| AmopChequeVacance      | Cheque Vacance             |                                                                                                                                                       |
| AmopCimbclicks         | CIMB Clicks                |                                                                                                                                                       |
| AmopCimbniaga          | CIMB Niaga                 |                                                                                                                                                       |
| AmopClearpay           | Clearpay                   |                                                                                                                                                       |
| AmopCliQ               |  CliQ                      |                                                                                                                                                       |
| AmopCup                | UnionPay                   |                                                                                                                                                       |
| AmopDuitNow            | DuitNow                    |                                                                                                                                                       |
| AmopEmi                | Equated Monthly Instalment |                                                                                                                                                       |
| AmopEmis               | Multicaixa                 |                                                                                                                                                       |
| AmopEnets              | eNets                      |                                                                                                                                                       |
| AmopEps                | EPS                        |                                                                                                                                                       |
| AmopEwallet            | E-wallet                   |                                                                                                                                                       |
| AmopFlexpay            | Flexpay                    |                                                                                                                                                       |

| AmopFlynowpaylater     | FlyNow PayLater            |                                                                                                                                                       |
| AmopFps                |  FPS                       |                                                                                                                                                       |
| AmopFpx                | FPX                        |                                                                                                                                                       |
| AmopGcash              | GCash                      |                                                                                                                                                       |
| AmopGip                |  GIP                       |                                                                                                                                                       |
| AmopGiropay            | Giropay                    |                                                                                                                                                       |
| AmopGooglePay          | Google Pay                 |                                                                                                                                                       |
| AmopGrabpay            | GrabPay                    |                                                                                                                                                       |
| AmopHoolah             | ShopBack                   | Hoolah has been renamed to ShopBack, this AMOP definition is kept for backward compatibility                                                          |
| AmopHumm               | Humm                       |                                                                                                                                                       |
| AmopInternetBanking    | Internet Banking           |                                                                                                                                                       |
| AmopIataPay            | *Merchant dependant*       | The logo, and the displyable name of this AMOP may vary depending on the merchant.                                                                    |
| AmopIdeal              | Ideal                      |                                                                                                                                                       |
| AmopInstallmentpayment | Instalment payment         |                                                                                                                                                       |
| AmopInterac            | Interac                    |                                                                                                                                                       |
| AmopKcp                | KCP                        |                                                                                                                                                       |
| AmopKlarna             | Klarna                     |                                                                                                                                                       |
| AmopKnet               | KNET                       |                                                                                                                                                       |
| AmopKonbini            | Konbini                    |                                                                                                                                                       |
| AmopKonbini7_11        | Konbini7-11                |                                                                                                                                                       |
| AmopKonbiniConv        | Konbini-conv               |                                                                                                                                                       |
| AmopKorean            | South Korean Payment Methods |                                                                                                                                                       |
| AmopLinepay            | LINE Pay                   |                                                                                                                                                       |
| AmopMada               | Mada                       |                                                                                                                                                       |
| AmopMaybank            | Maybank2u                  |                                                                                                                                                       |
| AmopMobileBanking      | Mobile Banking             |                                                                                                                                                       |
| AmopMomo               | Momo                       |                                                                                                                                                       |
| AmopMultibanco         | Multicaixa |                                                                                                                                                       |
| AmopMtn                | MTN                       |                                                                                                                                                       |
| AmopNapas              | Napas                      |                                                                                                                                                       |
| AmopNapasPayNow        | Napas Pay Now              |                                                                                                                                                       |
| AmopNordea             | Nordea                     |                                                                                                                                                       |
| AmopOnlinebanking      | Online Banking             |                                                                                                                                                       |
| AmopOrange             | Orange Money               |                                                                                                                                                       |
| AmopOsbPayzen          |  OSB                       |                                                                                                                                                       |
| AmopPaga               |  Paga                      |                                                                                                                                                       |
| AmopPaymaya            | PayMaya                    |                                                                                                                                                       |
| AmopPaypal             | PayPal                     |                                                                                                                                                       |
| AmopPix                | Pix                       |                                                                                                                                                       |
| AmopPoli               | Poli                       |                                                                                                                                                       |
| AmopPostFinance        | PostFinance                |                                                                                                                                                       |
| AmopPromptpay          | Prompt Pay                 |                                                                                                                                                       |
| AmopPrzelewy24         | Przelewy24                 |                                                                                                                                                       |
| AmopQR                 | QR Payment                 |                                                                                                                                                       |
| AmopQrph               | QRPh                       |                                                                                                                                                       |
| AmopRiverty            | Riverty                    |                                                                                                                                                       |
| AmopRupay              | RuPay                      |                                                                                                                                                       |
| AmopSeb                | SEB                        |                                                                                                                                                       |
| AmopSemis              | Multicaixa Express         |                                                                                                                                                       |
| AmopSepa               | Sepa                       |                                                                                                                                                       |
| AmopShopback           | ShopBack                   |                                                                                                                                                       |
| AmopShopeepay          | Shopee Pay                 |                                                                                                                                                       |
| AmopSofort             | Sofort                     |                                                                                                                                                       |
| AmopStripeRedirect     | Stripe Redirect Payments   |                                                                                                                                                       |
| AmopSwish              | Swish                      |                                                                                                                                                       |
| AmopTMoney             | T Money                   |                                                                                                                                                       |
| AmopTigoCash           | Tigo Cash                 |                                                                                                                                                       |
| AmopTouchngo           | TouchnGO                   |                                                                                                                                                       |
| AmopTrustly            | Trustly                    |                                                                                                                                                       |
| AmopUba                | UBA                       |                                                                                                                                                       |
| AmopUpi                | UPI                        |                                                                                                                                                       |
| AmopUplift             | Uplift                     |                                                                                                                                                       |
| AmopValu               | valU                       |                                                                                                                                                       |
| AmopVietqr             | VietQr                     |                                                                                                                                                       |
| AmopVipps              | Vipps                      |                                                                                                                                                       |
| AmopVnpay              | VNPay                      |                                                                                                                                                       |
| AmopWallet             | Wallet                     |                                                                                                                                                       |
| AmopWechat             | WeChat                     |                                                                                                                                                       |
| PayByLink              | PayByLink                  |                                                                                                                                                       |
| PaymentCard            | `undefined`                |                                                                                                                                                       |
| PaymentCardToken       | `undefined`                | Some extra information about the token are available in `cardFormDetails` property                                                                    |
