# Outpayce Checkout SDK - Release Notes


### 3.2.2
#### Supported until 08 Nov 2024
- fix bad import on previous version
### 3.2.1
#### Supported until 08 Nov 2024
- fix bad import on previous version
### 3.2.0
#### Supported until 08 Nov 2024
- Introduced new option in SDK `emitcontactformdetails`.If it set to `true`, the onReady and onChange callbacks will be enriched with contact form details information. 
- Added new `contactForm`.Only present if `emitContactFormDetails` set to true (and contactDetails set to true in the profile manager mop-config) with the following properties (or Null if not filled or irrelevant):
  -  `email`: String.
  -  `phone`: String.
- Based on the emailconfig and phoneconfig the contact Detail fields should appear in the contactForm. Email and phone configuration should be done in the profile manager phone config.
- new urls for checkout sdk static files
- Introduced a new action `prepare`. This action streamlines the prepayment process by fetching obfees and installment plans (if applicable) in a single call.
- The card form component has been updated to utilize the new `prepare` action. As a result, it no longer uses the separate `obfees`and `installments` actions.
- Adding logo for Amop Blik, Przelewy24, StripeRedirect
- Added new error messages for wallet

### 3.1.0
#### Supported until 08 Nov 2024
- perform 3DS for highest amount card in multi-card payments
- Added new field `walletForm` to `PaymentMethodStatus`
- Added `maxOccurences` field for each mop in the onReady callback
- Wallet component is changed into an Input instead of a Slider
- Added a new component `miles-form` which provides the slider functionality to the end user. Using this functionality allows the user to slide to a certain miles rank and pay the equivalent amount of cash.
- Added a new field `rewardForm`, which contains two optional fields `rewardPointsUsed` and `equivalentAmount`, to the `PaymentMethodStatus`.
- Added `lastDigits` field in the `CardFormDetails`

### 3.0.5
#### Supported until 08 Nov 2024
- Fix konbini7-11 and konbini-conv amops type name
### 3.0.4
#### Supported until 08 Nov 2024
- Trustly logo updated
### 3.0.3
#### Supported until 08 Nov 2024
- bug fix on AMOP vnPay logo name
### 3.0.2
#### Supported until 08 Nov 2024
- add konbini-all and konbini-7-11 logos for VN
### 3.0.1
#### Supported until 08 Nov 2024
- Only pass necessary actionTokens to `delegateServerCall` in `ValidateFop` flavor

### 3.0.0
#### Supported until 08 Nov 2024
- Added split payment options when `mopSelectionInSdk` is false. Restrictions apply, please check with your implementation point of contact.
- `onChange` callback is now deprecated. Please migrate to `onPaymentMethodsChange` to support latest capabilities.
- `setLoyalty` added new helper function to pass the QBRPoints informations for payment that includes QBRPoints.
- Multi Currency Pricing offers are now available in `onReady` callback via `paymentDetails.mcpOffers`.
- Added translation for Polish, Turkish, Danish, Ukrainian, Bulgarian, Slovenian and Portuguese-Brazilian language support.
- Added `maskCVV` option to mask/hide user's input in CVV field.
- Added `isTokenInvalid` error when prefilled card does not have bin information.

### Migration from version 2.6.0 to 3.0.0
In SDK version 3.0.0, if you are are handling payment methods selection in your page, the process has been updated.
`checkout.create(containerId, mopId)` now returns a string representing the `componentId` of the payment payment method. 

```Javascript
// - `container` can be either a String containing a HTML id or a HTMLElement
// - `mop` can be any element of the `mopsList` sent in `onReady` callback
const componentId =  await checkout.create(container, mop.id);
```
The returned componentId is crucial for further interactions with the component. When a payment method is selected, you should ensure its component is displayed. Instead of using `AmadeusCheckout.selectMop(mopId)`, you now call `checkout.selectMopComponent(selectedComponentId)` to get the right status updates. If another payment method is chosen, you can simply call `AmadeusCheckout.selectMopComponent(componentId)` again.

Version 3.0.0 also introduce support for multiple Methods of Payment via the helper function `AmadeusCheckout.selectMopComponents(mopSelections)` described below:

Parameters:
- `mopSelections`: Array. Array of objects, containing the following properties:
    - `componentId`: String.  Id of the payment method component.
    - `amount`: String. Amount used for the selected payment method. This property is optional (full amount considered if not present). We strongly advise to always provide it in case of multiple method of payments.

It is important to note that a debounce timer of 300ms has been added to the `selectMopComponents` function to reduce server calls.

Lastly, the `onChange` callback is now obsolete and should be replaced by `onPaymentMethodsChange`.

### 2.7.6
#### Supported until 08 Nov 2024
- Trustly logo updated
### 2.7.5
#### Supported until 08 Nov 2024
- Deliver stable 2.7.5 with userguide hyperlink update
### 2.7.4
#### Supported until 08 Nov 2024
- Adding logo for AMOP konbini7-11 and konbini-conv
### 2.7.3
- Only pass necessary actionTokens to `delegateServerCall` in `ValidateFop` flavor
### 2.7.2
#### Supported until 08 Nov 2024
- Add an option `forceChangeDetection` to force change detection on CVV and PAN field.
### 2.7.1
#### Supported until 08 Nov 2024
- Optimize SEPA logo
### 2.7.0
#### Supported until 08 Nov 2024
- Adding logo for Amop SEPA
### 2.6.0
#### Supported until 08 Nov 2024
- Upgrade to Angular 16
### 2.5.7 - Deprecated
- Fix to add catalan to supported langage
### 2.5.6 - Deprecated
- Added translation for Catalan language support
### 2.5.5 - Deprecated
- Added selected installment plan details in onChange callback
### 2.5.4 - Deprecated
- Added definition of Google Pay and Multibanco payment methods
### 2.5.3 - Deprecated
- Added definition of Napas Pay Now, KCP and Shopee Pay payment methods
### 2.5.2 - Deprecated
- Added definition of Napas payment methods
### 2.5.1 - Deprecated
- Added definition of Momo, VNPay, and VietQr payment methods

### 2.5.0 - Deprecated
- Improved accessibility

### 2.4.0 - Deprecated
- Added BIN information for prefilled cards in `onReady()` callback
- Added merchant vendor code mapping information in `onReady()` callback
- Added the possibility to configure the display of cardholderName through vendor configuration
- Added `collectBillingAddressForCountries` option

### 2.3.0 - Deprecated
- Added support of new Accertify Data Collector

### 2.2.0 - Deprecated
- Added support of alternative payment methods with parameters
- Added definition of LINE Pay payment method
- Updated billing address fields length
- Improved shadow DOM support
- Fixed minor bug on zipcode verification
- Fixed a bug preventing 2 card forms to work in a same session

### 2.1.1 - Deprecated
- Fixed a bug preventing the proper error handling of `delegateServerCall`

### 2.1.0 - Deprecated
- Added zipcode verification with a regular expression for each country
- Added definition of valU payment method

### 2.0.0 - Deprecated
- Breaking change: Updated callbacks to support payments with several methods
- Added currency conversion support
- Added support for Apple Pay on the Web
- Added translations for HU and AR languages
- Fixed minor bugs

### 1.10.9 - Deprecated
- Added `deniedCardIssuingCountries` option

### 1.10.8 - Deprecated
- Added `externalId` and `payMerchantReference` properties in `onSuccess()` callback

### 1.10.7 - Deprecated
- Added translations for CS language

### 1.10.6 - Deprecated
- Added definitions of PostFinance payment methods
- Added translations for NL language
- Fixed minor translation issues

### 1.10.5 - Deprecated
- Added a help tooltip on cardholder name field
- Updated cardholder name field to prevent special characters, and set its maximum length from 50 to 26 characters 

### 1.10.4 - Deprecated
- Fixed a bug related to 3DSv2

### 1.10.3 - Deprecated
- Added macrolanguages support (`no` language is interpreted as `nb`, `zh` as `zh-Hans`)

### 1.10.2 - Deprecated
- Fixed a bug preventing the usage of `collectCardholderName` and `collectCardExpiryDate` options

### 1.10.1 - Deprecated
- Added definition of ShopBack payment method

### 1.10.0 - Deprecated
- Added support of prefilled payment cards
- Added support of cobadged payment cards
- Added the possibility to configure the display of CVV (mandatory or not, displayed or not, length) through configuration
- Added `showSaveCardToggle` and `preferredCountryCodes` options
- Added `hasInstallments` and `needUserInput` flags to payment method objects
- Added `details` inside `cardBin` object to provide detailed card information
- Added definitions of IataPay and Vipps payment methods
- Added logo for Carte Bancaire vendor
- Updated Klarna logo
- Fixed minor bugs

### 1.9.0 - Deprecated
- Updated installment panel to be displayed before billing address
- Added textAlign option to form fields iframes
- Added states support
- Added definitions of EMI, InternetBanking, Wallet, UPI payment methods
- Bug fix: Fixed a bug that may occur when third party cookies are deactivated

### 1.8.0 - Deprecated
- Breaking change: Dropped compatibility with IE11 and Edge Legacy <= 44
- Added installments support on payment card form component
- Added definitions of Giropay, Hoolah, Konbini, RuPay payment methods
- Added translations for EL, RU, RO languages

### 1.7.0 - Deprecated
- Added contact details
- Added support of vendors with no CVV or optional CVV
- Updated BIN check to use 8 digits instead of 6

### 1.6.0 - Deprecated
- Improved Accessibility
- Added definitions of EPS, Trustly payment methods
- Added logo for Mada card vendor
- Added `displayPlaceholder` parameter
- Improved handling of user cancellation
- Fixed a bug with web redirections in delegation flow

### 1.5.0 - Deprecated
- Added translations for PT language
- Updated `container` parameter to also accept HTMLElements
- Fixed bug related to shadow dom usage in host application
- Fixed various bugs

### 1.4.0 - Deprecated
- Added definitions of BillEase, GCash, Interac, Mada, PayMaya and Online Banking payment methods
- Added an alternative CSS file (`sdk-no-material.css`) for Angular Material based applications
- Improved CSS encapsulation

### 1.3.0 - Deprecated
- Added `delegateServerCall()` callback
- Added definitions of Humm and FNPL payment methods
- Added support of extra environements
- Fixed various bugs

### 1.2.0 - Deprecated
- Added extra information to `onSuccess()` callback: fees and payment references
- Added a new option to deactivate cardholder name collection
- Added `displayableName` property to alternative payment methods
- Added translations for ID, JA, KO, MS, TH and VI languages
- Updated default responsiveness of payment card form component
- Updated card expiry field type to display digital keyboard on mobile
- Fixed various bugs

### 1.1.0 - Deprecated
- `mount()` function renamed to `start()`
- Added support of fingerprints and 3DSv2
- Added `isLoading` flag in `onChange()` callback
- Added `create()` function to dynamically create components for a given payment method
- Added `getIconUrl()` function to retrieve payment method logos
- Fixed various bugs

### 1.0.0 - Deprecated
- Initial stable version