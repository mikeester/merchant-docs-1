# Bitinvestor Merchant Checkout Documentation

This documentation is for merchants who want to integrate Bitinvestor Checkout into their website. Bitinvestor Checkout is a widget that allows your customers to deposit cryptocurrency on your website, paid for by the customer in fiat. The widget is embedded in an iFrame and is fully customizable. The widget is designed to be simple to use and easy to integrate into your website.


- [Bitinvestor Merchant Checkout Documentation](#bitinvestor-merchant-checkout-documentation)
  - [Query Parameters](#query-parameters)
    - [Required Parameters:](#required-parameters)
    - [Optional Parameters:](#optional-parameters)
  - [Server-Side URL Signature:](#server-side-url-signature)
    - [Example with NodeJS:](#example-with-nodejs)
    - [Example iFrame URL:](#example-iframe-url)
    - [Example iFrame:](#example-iframe)
  - [Order Notifications:](#order-notifications)
    - [Responses](#responses)
      - [Payment Pending:](#payment-pending)
      - [Order Cancelled:](#order-cancelled)
      - [Order Completed:](#order-completed)
      - [Order Broadcasted:](#order-broadcasted)
    - [Response Definition:](#response-definition)
    - [The signature:](#the-signature)
      - [Example with NodeJS:](#example-with-nodejs-1)
  - [Supported Cryptocurrencies:](#supported-cryptocurrencies)
  - [Supported Fiat Currencies:](#supported-fiat-currencies)
  - [Test Credentials](#test-credentials)
    - [Credit Card:](#credit-card)
  - [Payment Methods](#payment-methods)
    - [Response Definition:](#response-definition-1)
  - [Get Payment Status](#get-payment-status)
    - [Response Definition:](#response-definition-2)

## Query Parameters

### Required Parameters:

- `apiKey`: Your publishable API key for customer and transaction assignment. Also known as the public key.
- `currencyCode`: The cryptocurrency code (e.g., BTC, ETH, BCH) for purchase. See [Supported Cryptocurrencies](https://github.com/BitinvestorNet/merchant-docs?tab=readme-ov-file#supported-cryptocurrencies) for more options. If multiple walletAddress are set, this will be the default cryptocurrency.
- `walletAddress`: The destination cryptocurrency wallet address for purchased funds. For instance, if you want to accept Bitcoin, Litecoin, and Dogecoin, format the walletAddress parameter as follows: `walletAddress=BTC:yourBitcoinAddress,LTC:yourLitecoinAddress,DOGE:yourDogecoinAddress`.
  **Note** that the code is case sensitive so ensure the code you use for your default cryptocurrency matches what you used in `currencyCode` field.

### Optional Parameters:

- `language`: Specify the widget's language using an [ISO 639 Set 1 code](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes). The language will default to EN (English) if the parsed language is not supported. Currently supported languages: EN, DA, TR, ES, PT-BR
- `baseCurrencyCode`: The fiat currency code for transactions (e.g., USD, EUR, GBP). See [Supported Fiat currencies](https://github.com/BitinvestorNet/merchant-docs?tab=readme-ov-file#supported-fiat-currencies) for more options.
- `lockBaseCurrency`: (True/False) Locks the currency selection to the specified `baseCurrencyCode`
- `baseCurrencyAmount`: The fiat currency amount to be spent, in integer form. 2 decimals max. Cannot be zero if set.
- `quoteCurrencyAmount`: The cryptocurrency amount to be purchased, in integer form. 3 decimals max. 
- `email`: The email address of the customer.
- `externalTransactionId`: Your unique identifier for the transaction.
- `externalCustomerId`: Your unique identifier for the customer.
- `redirectUrl`: The URL for redirection after purchase completion. Must be URLencoded, e.g. https%3A%2F%2Fwww.myurl.com.
- `responseUrl`: The URL for receiving order notifications. See more under the section Order Notifications. Must be URLencoded, e.g. https%3A%2F%2Fwww.myurl.com.
- `customerKYC`: The level of Know Your Customer (KYC) verification. customerKYC = 0 means the customer has not completed KYC. customerKYC = 1 means the customer has completed Proof of ID + Liveness Check. customerKYC = 2 means the customer has completed Proof of ID + Liveness Check + Proof of Address.
- `style`: The styling ID provided by Bitinvestor for customization.
- `destinationTag`: Adds a destination tag to the transaction. Only supported for XRP and TON.
- `coverFees`: (True/False) Sets processing and handling fees to zero for the customer. Fees will be included through the spread.
- `minAmount`: Sets the minimum order amount in EUR.
- `lockAmount`: (True/False) Locks the fiat and crypto fields for the user.


## Server-Side URL Signature:

- Compute an HMAC with SHA-256 using your secret API key and the original query string.

### Example with NodeJS:

```javascript
import crypto from 'crypto';

const publicKey = 'your_public_key';
const secretKey = 'your_secret_key';
const currencyCode = 'currency_code';
const walletAddress = 'your_wallet_address';

const originalUrl = `https://checkout.bitinvestor.net?apiKey=${publicKey}&currencyCode=${currencyCode}&walletAddress=${walletAddress}`;
const signature = crypto.createHmac('sha256', secretKey).update(new URL(originalUrl).search).digest('base64');
const urlWithSignature = `${originalUrl}&signature=${encodeURIComponent(signature)}`;

console.log(urlWithSignature);
```

### Example iFrame URL:

The iFrame has been optimized for height: 585px; width: 445px;.

https://checkout.bitinvestor.net?apiKey={pk_live_key}&currencyCode=btc&signature=loremipsum 


### Example iFrame:

```html
<iframe allow="accelerometer; autoplay; camera; encrypted-media; gyroscope; payment; clipboard-read; clipboard-write" src="https://checkout.bitinvestor.net?apiKey={pk_live_key}&currencyCode=btc&signature=lorem" title="Buy crypto with Bitinvestor" style="height: 585px; width: 445px; border-radius: 
0.75rem; margin: auto;"></iframe>
```

## Order Notifications:
To receive order notifications you must provide a responseUrl. You will receive the notification on the provided url. In the header of the HTTP request, there’s a signature to make sure the data comes from Bitinvestor. 
Order notifications are sent to the responseUrl you specify. These notifications inform you of the status of each order. Below are the types of notifications you can receive.

### Responses


#### Payment Pending:
Indicates that the transaction has been created but the customer hasn't completed payment.

```json
{
  "order_status": "payment_pending",
  "external_transaction_id": "1234567",
  "external_customer_id": "1234567",
  "order_crypto": "LTC",
  "order_amount_usd": 25,
  "order_amount_usd_plus_fees": 25.5,
  "order_amount_eur": 21,
  "order_amount_eur_plus_fees": 22.5,
  "order_id": "9fcc45a5-4def-4953-9bd8-9ff75d9aaa9c"
}
```

#### Order Cancelled:
Indicates that the order is cancelled for any reason, such as payment failure, or user cancellation.

```json
{
  "order_status": "order_cancelled",
  "external_transaction_id": "1234567",
  "external_customer_id": "1234567",
  "order_id": "9fcc45a5-4def-4953-9bd8-9ff75d9aaa9c"
}
```

#### Order Completed:
Indicates that the order has been processed, and the cryptocurrency purchase was successful.

```json
{
  "order_id": "9fcc45a5-4def-4953-9bd8-9ff75d9aaa9c",
  "order_crypto_amount": 0.347764,
  "order_crypto": "LTC",
  "order_status": "order_completed",
  "order_crypto_address": "ltc1qlec2yfpkdvn4lr0vpf27qggrxtu34zeu5l6g2u",
  "external_transaction_id": "1234567",
  "external_customer_id": "1234567",
  "order_amount_usd": 25,
  "order_amount_usd_plus_fees": 25.5,
  "order_amount_eur": 21,
  "order_amount_eur_plus_fees": 22.5,
  "network": "litecoin"
}
```

#### Order Broadcasted:
Indicates that the cryptocurrency transaction has been broadcasted to the blockchain.

```json
{
  "order_id": "9fcc45a5-4def-4953-9bd8-9ff75d9aaa9c",
  "order_crypto_amount": 0.347764,
  "order_crypto": "LTC",
  "order_status": "order_broadcasted",
  "transaction_id": "ea458dda0ff8583199bdd4d9b9a69a2813694764a633fd40b27de22a868cebec"
  "order_crypto_address": "ltc1qlec2yfpkdvn4lr0vpf27qggrxtu34zeu5l6g2u",
  "external_transaction_id": "1234567",
  "external_customer_id": "1234567",
  "order_amount_usd": 25,
  "order_amount_usd_plus_fees": 25.5,
  "order_amount_eur": 21,
  "order_amount_eur_plus_fees": 22.5,
  "network": "litecoin"
}
```

### Response Definition:

- `order_id`: The order id on Bitinvestor.
- `order_crypto_amount`: The exact cryptocurrency amount you will receive.
- `order_crypto`: The cryptocurrency you receive.
- `order_status`: The current status of the order.
- `order_crypto_address`: The cryptocurrency address where you receive the cryptocurrency.
- `external_transaction_id`: Your transaction id (If provided in the URL).
- `external_customer_id`: Your customer's id (If provided in the URL).
- `order_amount_usd`: The `order_crypto_amount` converted to USD (Mid-market rates without spread). This does not include the platform fee.
- `order_amount_usd_plus_fees`: The `order_amount_usd` plus the platform fee.
- `network`: The network used to send transaction via.

### The signature:

Compute an HMAC with a SHA-256 hash function. Use your secret API key as the key and use the request body as the message. Compare this to the signature sent in the request header.

#### Example with NodeJS:

```javascript
import crypto from 'crypto';

const serverKey = 'sk_test_key'; // Replace with your server key

const requestBody = '{ "order_id": "9fcc45a5-4def-4953-9bd8-9ff75d9aaa9c"}'

const signature =
  crypto
    .createHmac('sha256', serverKey)
    .update(requestBody)
    .digest('base64'); 
```

## Supported Cryptocurrencies:
The following are all currencies supported for iFrame initalization.

| Currency |  |  |  |  |
| ------------ | ------------- | --- | ------------- | ------------- |
| BTC          | LTC           | ETH | XRP           | USDC_ETHEREUM |
| DAI_ETHEREUM | LINK_ETHEREUM | BCH | DOGE          | USDT_ETHEREUM |
| USDT_TRON    | USDT_SOLANA   | TRX | SOL           | TON           |



## Supported Fiat Currencies:

| Currency |  |  |  |  |
| --- | --- | --- | --- | --- |
| EUR | USD | PLN | SEK | SGD |
| KZT | RON | ZAR | UAH | TWD |
| MXN | IDR | JPY | AUD | HKD |
| NZD | THB | CZK | TRY | DKK |
| CHF | CAD | INR | BGN | ILS |
| BRL | HUF | GBP |     |     |


## Test Credentials

It's currently only possible to make test orders with Visa/MasterCard as the payment method and EUR + USD as the payment currency.

### Credit Card:

Card number: 
```plaintext
4242 4242 4242 4242
```
Expiration date: 
```plaintext
10/31
```
CVC/CVV: 
```plaintext
123
```

## Payment Methods
All payment methods & their supported countries can be retrieved with a GET request.

URL:  https://eosdj3ojuf31557.m.pipedream.net/

Response:
```json
{
    {
    "id": 2,
    "name": "PIX",
    "fee": 2,
    "slug": "pix",
    "area": [
      "BR"
    ],
    "currency": [
      "BRL"
    ],
    "excluded": [
      
    ]
  },
}
```

### Response Definition:
- `id`: The payment method id.
- `name`: The name of the payment method.
- `fee`: The fee in percentage.
- `slug`: The payment method slug.
- `area`: The areas where the payment method is available. These use [ISO 639 Set 1 code](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes).
- `currency`: The currencies the payment method supports.
- `excluded`: Countries not supported by this payment method.

## Get Payment Status
Allows for retrieving the status of a payment.

URL:  https://eocb95pk8c29bt6.m.pipedream.net

Request Data:
```json
{
  "order_id": "bitinvestor_order_id",
  "api_key": "your_secret_key"
}
```

Response:
```json
{
    "order_id": "bitinvestor_order_id",
    "order_crypto": "LTC",
    "external_transaction_id": "",
    "external_customer_id": "123456",
    "order_status": "order_broadcasted",
    "transaction_id": "fada834ba9f1b398bacf6153e44e320dfe1821bffe03a68d29ca76cb3bbc60c5"
}
```

### Response Definition:
- `order_id`: The order id on Bitinvestor.
- `order_crypto`: The cryptocurrency you receive.
- `external_transaction_id`: Your transaction id (If provided in the URL).
- `external_customer_id`: Your customer's id (If provided in the URL).
- `order_status`: The current status of the order.
- `transaction_id`: The transaction id on the blockchain.

