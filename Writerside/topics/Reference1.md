<link-summary>The GOV.UK Pay API reference gives an overview of everything you can do in the GOV.UK Pay API.</link-summary>

# API reference 1

GOV.UK Pay is a payment platform for government. If you work in the public sector, [read more about using GOV.UK Pay to take payments](https://www.payments.service.gov.uk/).

The base URL for the GOV.UK Pay API is `https://publicapi.payments.service.gov.uk/`.

We use the same base URL for test and live accounts. You can create a test account from the [GOV.UK Pay homepage](https://www.payments.service.gov.uk/) and use it to test our API and your integration with it. You receive your live account when you [make your service live for your users](/switching_to_live/).

The API key you use determines which GOV.UK Pay service you use, and whether actions are treated as test payments, or processed as real payments. You can create test and live API keys through the GOV.UK Pay admin tool.

> If you’re testing our API, make sure you enter an API key from a test account to avoid creating real payments.
> {style="warning"}

The API is based on REST principles. It returns data in JSON format, and uses standard HTTP status codes.

You can read about how to [get started quickly with the GOV.UK Pay API](https://docs.payments.service.gov.uk/quick_start_guide/#quick-start).

## Authentication

The GOV.UK Pay API authenticates requests with [OAuth2 HTTP bearer tokens](http://tools.ietf.org/html/rfc6750). You can generate new API keys in the GOV.UK Pay admin tool.

When you make an API request, you need to use an `Authorization` HTTP header to provide your API key, with a `Bearer` prefix. For example:

```
Authorization: Bearer api_test123456
```

## Allowing traffic to our API

If you’re connecting to our API from a secure network, we recommend using [a web proxy like Squid](http://www.squid-cache.org/) to allow traffic to our base URL.

Do not add an allow list of IP addresses to your firewall, because GOV.UK Pay’s IP addresses will change regularly. Read more about [why GOV.UK does not support allow lists of IP addresses](https://technology.blog.gov.uk/2017/01/03/a-whitelisting-approach-for-cloud-apis/).

## Rate limits

Per second, you can make:

- up to 15 `POST` requests to [create a payment](/api_reference/create_a_payment_reference)
- up to 15 `POST `requests to [capture a delayed payment](/api_reference/capture_payment_reference)
- up to 15 other `POST` requests

`GET` requests are also rate-limited, but at a very high level. In the future, we will publish an official rate limit for `GET` requests.

If you exceed the rate limit, this will return a `429` HTTP status code (Too many requests) and error code `P0900`. After a second, you’ll be able to retry your attempt in a reasonable way. For example, using [exponential backoff](https://en.wikipedia.org/wiki/Exponential_backoff).

[Contact us](/support_contact_and_more_information/) if you would like to discuss rate limiting applied to your service account, or give us feedback.

## Pagination

Responses from search endpoints, such as [search payments](/api_reference/search_payments_reference) or [search refunds](/api_reference/search_refunds_reference), are split into pages. By default, these endpoints display 500 results per page and return the first page of results.

You can use query parameters to view different pages and control the number of results on each page.

{type="medium"}
`page`
: Returns a specific page of results.<br>Defaults to `1`.

`display_size`
: The number of results returned per results page.<br>Defaults to `500`. Maximum value is `500`.

### Pagination example

There are 74 payments in the `created` state. In this example search request, these 74 payments are split into 4 pages of results because `display_size` is `20`. This request returns payments 41-60 because `page` is `3`:

`GET /v1/payments?state=created&display_size=20&page=3`

### Pagination links

Search endpoints also return a `_links` object, which includes `href` and `method` fields you can use to move between pages. Use the fields in:

* `self` to run the same search again
* `first_page` to get the first page of results
* `last_page` to get the last page
* `prev_page` to get the previous page
* `next_page` to get the next page

## OpenAPI file

You can also generate a client library from [our OpenAPI file](https://github.com/alphagov/pay-publicapi/blob/master/openapi/publicapi_spec.json) using [Swagger Editor](https://editor.swagger.io/). This may be an easier way for you to integrate your service with GOV.UK Pay than writing a client library from scratch. You cannot use Swagger Editor to make test API requests.

## Responses

The GOV.UK Pay API uses HTTP status codes to show the outcome of a request. `4xx` and `5xx` status codes give more detail in the `code` and `description` attributes of the response.

### HTTP status codes

{type="wide"}
200 - OK 
: Your request was successful.

201 - Created
: You created a payment.

202 - Accepted
: Refund request accepted. The refund will reach the paying user soon.

204 - No content
: Your request was successful.

400 - Bad request
: Your request is invalid.<br>Check the `code` and `description` response attributes to find out why the request failed. |

401 - Unauthorised
: Your API key is missing or invalid.<br>[Read more about authenticating GOV.UK Pay API requests](/api_reference/#authentication).

404 - Not found 
: The payment, refund, or agreement you tried to access does not exist.<br>Check the ID you sent in your request.

409 - Conflict 
: The payment you tried to access has already been captured or cancelled, or you sent an `Idempotency-Key` that [has already been used](/recurring_payments/#use-idempotency-to-avoid-double-charging-a-user).

412 - Precondition failed
: The `refund_amount_available` value you sent does not match the amount available to refund.<br><br>[Read more about refunding payments](/refunding_payments/#creating-a-refund).

422 - Unprocessable entity
: One of the values you sent is formatted incorrectly. This could be an invalid value, or a value that exceeds a character limit.<br>Check the `code`, `description`, and the `field` or `header` attributes in the response for more information.

429 - Too many requests
: You've made too many requests using your API key.<br><br>[Read more about rate limits](/api_reference#rate-limits).

Any 500 error - Internal server error 
: There's something wrong with GOV.UK Pay. <br>If there are no problems on [our status page](https://payments.statuspage.io), you can [contact us with your error code](/support_contact_and_more_information/) and we'll investigate. |

### GOV.UK Pay API error codes

If your request fails, you’ll also receive a response that contains `code`, `description`, and `field` attributes to help you find out why the request failed.

For example:

```json
{
  "field": "amount",
  "code": "P0102",
  "description": "Invalid attribute value: amount. Must be less than or equal to 10000000"
}
```

These error descriptions are intended for developers, not your users.

<div style="height:1px;font-size:1px;">&nbsp;</div>

<table>
<tr><td>Error code</td><td>Endpoint(s)</td><td>Meaning</td></tr>
<tr><td>P0101</td><td>Create a payment<br>Send card details to authorise a MOTO payment</td><td>The request you sent is missing a required parameter.<br><br>Check the <code>field</code> attribute in the response to see which parameter is missing.</td></tr>
<tr><td>P0102</td><td>Create a payment<br><br>Send card details to authorise a MOTO payment</td><td>A header or parameter value you sent in your request is invalid.<br><br>Check the <code>description</code> attribute in the response to find out which value is invalid.</td></tr>
<tr><td>P0104</td><td>Create a payment</td><td>You included <code>return_url</code> and <code>&quot;authorisation_mode&quot;: &quot;moto_api&quot;</code> in your request. These parameters cannot be used together.<br><br>Remove the <code>return_url</code> parameter to create <a href="/moto_payments/moto_send_card_details_api">a Mail Order / Telephone Order (MOTO) payment that accepts card details sent through the API</a>.<br><br>Remove the <code>authorisation_mode</code> parameter to <a href="/making_payments">create a standard payment</a>.</td></tr>
<tr><td>P0191</td><td>Create a payment</td><td>The <code>Idempotency-Key</code> you sent in the request header has already been used to create a payment.</td></tr>
<tr><td>P0195</td><td>Create a payment</td><td>You tried to create a Mail Order / Telephone Order (MOTO) payment that accepts card details sent through the API but this feature is not turned on for your service.<br><br>You can <a href="/moto_payments/moto_send_card_details_api">read more about setting up your service to accept card details sent through the API</a>.<br><br>To create a standard payment, remove <code>&quot;authorisation_mode&quot;: &quot;moto_api&quot;</code> from your request.</td></tr>
<tr><td>P0196</td><td>Create a payment</td><td>You tried to create a type of payment that is not turned on for your service.<br><br>Contact us about <a href="/moto_payments">setting up your service for MOTO payments</a> or <a href="/recurring_payments">recurring payments</a>.</td></tr>
<tr><td>P0197</td><td>Create a payment</td><td>The JSON you sent in the request body is invalid.<br><br>Check the formatting of the request body.</td></tr>
<tr><td>P0198</td><td>Create a payment</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0199</td><td>Create a payment</td><td>There's a problem with your service account.<br><br><a href="/support_contact_and_more_information">Contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0200</td><td>Get information about a single payment</td><td>No payment matched the <code>{PAYMENT_ID}</code> you provided.<br><br>Check the <code>{PAYMENT_ID}</code> parameter you sent.</td></tr>
<tr><td>P0298</td><td>Get information about a single payment</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0300</td><td>Get a payment’s events</td><td>No payment matched the <code>{PAYMENT_ID}</code> you provided.<br><br>Check the <code>{PAYMENT_ID}</code> parameter you sent.</td></tr>
<tr><td>P0398</td><td>Get a payment’s events</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0401</td><td>Search payments</td><td>The value of a parameter you sent is invalid. <br><br>Check the parameters listed in the <code>description</code> response attribute.</td></tr>
<tr><td>P0402</td><td>Search payments</td><td>The requested page of search results does not exist.</td></tr>
<tr><td>P0498</td><td>Search payments</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0500</td><td>Cancel a payment</td><td>No payment matched the <code>{PAYMENT_ID}</code> you provided.<br><br>Check the <code>{PAYMENT_ID}</code> parameter you sent.</td></tr>
<tr><td>P0501</td><td>Cancel a payment</td><td>Cancelling the payment failed. This could be because this payment does not have a <code>cancel</code> attribute and so cannot be cancelled.<br><br>Read our <a href="/making_payments/" anchor="check-if-you-can-cancel-a-payment">guidance on checking if you can cancel a payment</a>.<br><br>If you think you should be able to cancel a payment but you're still receiving this error, <a href="/support_contact_and_more_information/">contact us</a>.</td></tr>
<tr><td>P0502</td><td>Cancel a payment</td><td>This payment cannot be cancelled. You can only cancel a payment if it has a <code>cancel</code> attribute when you check if you can cancel it. <br><br>There’s further <a href="/making_payments/" anchor="check-if-you-can-cancel-a-payment">guidance on checking if you can cancel a payment</a>.</td></tr>
<tr><td>P0598</td><td>Cancel a payment</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0600</td><td>Refund a payment</td><td>No payment matched the <code>{PAYMENT_ID}</code> you provided.<br><br>Check the <code>{PAYMENT_ID}</code> parameter you sent.</td></tr>
<tr><td>P0601</td><td>Refund a payment</td><td>Your request is missing a required attribute. <br><br>You must submit <code>amount</code> and <code>refund_amount_available</code> values in the body of your request.</td></tr>
<tr><td>P0602</td><td>Refund a payment</td><td>The value of an attribute you sent is invalid.<br><br>Check the <code>amount</code> and <code>refund_amount_available</code> values you sent.</td></tr>
<tr><td>P0603</td><td>Refund a payment</td><td>This payment cannot be refunded. <br><br>You can <a href="/refunding_payments/" anchor="check-if-you-can-refund-a-payment">read more about checking if you can refund a payment</a>.</td></tr>
<tr><td>P0604</td><td>Refund a payment</td><td>The <code>refund_amount_available</code> value you sent does not match the amount available to refund. <br><br><code>refund_amount_available</code> must match the <code>amount_available</code> value you receive when checking if you can refund a payment.<br><br><a href="/refunding_payments/" anchor="check-if-you-can-refund-a-payment">Read more about checking if you can refund a payment</a>.</td></tr>
<tr><td>P0697</td><td>Refund a payment</td><td>The JSON you sent in the request body is invalid.<br><br>Check the formatting of the request body.</td></tr>
<tr><td>P0698</td><td>Refund a payment</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0700</td><td>Check the status of a refund</td><td>Either no refund matched the <code>{REFUND_ID}</code> you sent, or no payment matched the <code>{PAYMENT_ID}</code> you sent.<br><br>Check the <code>{REFUND_ID}</code> and  <code>{PAYMENT_ID}</code> parameters you sent.</td></tr>
<tr><td>P0798</td><td>Check the status of a refund</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0800</td><td>Get information about a payment’s refunds</td><td>No payment matched the <code>{PAYMENT_ID}</code> you sent.<br><br>Check the <code>{PAYMENT_ID}</code> parameter you sent.</td></tr>
<tr><td>P0898</td><td>Get information about a payment’s refunds</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P0900</td><td>All endpoints</td><td>You've made too many requests too quickly using your API key. <br><br>You can <a href="/api_reference" anchor="rate-limits">read more about rate limits</a>.</td></tr>
<tr><td>P0920</td><td>All endpoints</td><td>Our firewall blocked your request.<br>To fix a <code>P0920</code> API error, make sure your API request:<li>has a <code>Content-Type: application/json</code> header<li>uses <code>application/json</code> in the <code>Accept</code> header if you’re using an <code>Accept</code> header<li>uses <code>https</code> in the <code>return_url</code>, not <code>http</code><li>does not use invalid characters like <code>&lt;</code>, <code>&gt;</code>, <code>&quot;</code>, <code>\</code>, or <code>|</code><li>does not have an empty request body if you’re making a <code>POST</code> request</td></tr>
<tr><td>P0940</td><td>All endpoints</td><td>Your payment service provider (PSP) account is not fully configured. <br><br>You can <a href="/switching_to_live/" anchor="go-live">read our Go live documentation to configure your PSP account</a>.</td></tr>
<tr><td>P0941</td><td>Multiple endpoints</td><td>GOV.UK Pay has turned off payment and refund creation on this account. <br><br> This error can have multiple causes. <br><br> <a href="/support_contact_and_more_information">Contact us with your error code</a>.</td></tr>
<tr><td>P0942</td><td>Multiple endpoints</td><td>Recurring card payments are currently turned off for this service.<br><br><a href="https://www.payments.service.gov.uk/support/">Contact us with your error code.</a></td></tr>
<tr><td>P0999</td><td>All endpoints</td><td>GOV.UK Pay is temporarily down. <br><br>Check <a href="https://payments.statuspage.io">our status page</a> for more information.</td></tr>
<tr><td>P1000</td><td>Capture a delayed payment</td><td>No payment matched the <code>{PAYMENT_ID}</code> you provided.<br><br>Check the <code>{PAYMENT_ID}</code> parameter you sent.</td></tr>
<tr><td>P1001</td><td>Capture a delayed payment</td><td>GOV.UK Pay could not capture this payment. <br><br><a href="/support_contact_and_more_information">Contact us with your error code</a>.</td></tr>
<tr><td>P1003</td><td>Capture a delayed payment</td><td>GOV.UK Pay could not capture this payment. The <code>status</code> of the payment must be <code>capturable</code>. <br><br>To check this, <a href="/api_reference/single_payment_reference">run <code>GET /v1/payments/{PAYMENT_ID}</code></a>.</td></tr>
<tr><td>P1098</td><td>Capture a delayed payment</td><td>There's something wrong with GOV.UK Pay. <br><br>If there are no problems on <a href="https://payments.statuspage.io">our status page</a>, you can <a href="/support_contact_and_more_information/">contact us with your error code</a> and we'll investigate.</td></tr>
<tr><td>P1101</td><td>Search refunds</td><td>The value of a parameter you sent is invalid.<br><Br>Check the parameters listed in the <code>description</code> response attribute.</td></tr>
<tr><td>P1100</td><td>Search refunds</td><td>The requested page of search results does not exist.</td></tr>
<tr><td>P1211</td><td>Send card details to authorise a MOTO payment</td><td>No payment matches the <code>one_time_token</code> you sent.<br><br>Check the <code>one_time_token</code> parameter you sent.</td></tr>
<tr><td>P1212</td><td>Send card details to authorise a MOTO payment</td><td>The <code>one_time_token</code> you sent has already been used.<br><br>You can only use a <code>one_time_token</code> once.<br><br>Create a new payment to get a new <code>one_time_token</code>. <a href="/moto_payments/moto_send_card_details_api">Read more about creating and authorising MOTO payments by sending card details through the API</a>.</td></tr>
<tr><td>P2101</td><td>Create an agreement for recurring payments</td><td>The request you sent is missing a required parameter.<br><br>Check the <code>field</code> attribute in the response to see which parameter is missing.</td></tr>
<tr><td>P2102</td><td>Create an agreement for recurring payments</td><td>The value of a parameter you sent is invalid.<br><br>Check the response to see which value in invalid.</td></tr>
<tr><td>P2197</td><td>Create an agreement for recurring payments</td><td>The JSON you sent in the request body is invalid.<br><br>Check the formatting of the request body.</td></tr>
<tr><td>P2198</td><td>Create an agreement for recurring payments</td><td>There’s something wrong with GOV.UK Pay.<br><br>If there are no problems on <a href="https://payments.statuspage.io/">our status page</a>, you can <a href="https://docs.payments.service.gov.uk/support_contact_and_more_information/">contact us with your error code</a> and we’ll investigate.</td></tr>
<tr><td>P2200</td><td>Get information about a single agreement for recurring payments</td><td>No agreement matched the <code>{AGREEMENT_ID}</code> you sent.<br><br>Check your <code>{AGREEMENT_ID}</code> parameter.</td></tr>
<tr><td>P2298</td><td>Get information about a single agreement for recurring payments</td><td>There’s something wrong with GOV.UK Pay.<br><br>If there are no problems on <a href="https://payments.statuspage.io/">our status page</a>, you can <a href="https://docs.payments.service.gov.uk/support_contact_and_more_information/">contact us with your error code</a> and we’ll investigate.</td></tr>
<tr><td>P2401</td><td>Search agreements for recurring payments</td><td>The value of a parameter you sent is invalid.<br><br>Check the parameters listed in the <code>description</code> response attribute.</td></tr>
<tr><td>P2402</td><td>Search agreements for recurring payments</td><td>The requested page of search results does not exist.</td></tr>
<tr><td>P2498</td><td>Search agreements for recurring payments</td><td>There’s something wrong with GOV.UK Pay.<br><br>If there are no problems on <a href="https://payments.statuspage.io/">our status page</a>, you can <a href="https://docs.payments.service.gov.uk/support_contact_and_more_information/">contact us with your error code</a> and we’ll investigate.</td></tr>
<tr><td>P2500</td><td>Cancel an agreement for recurring payments</td><td>No agreement matched the <code>{AGREEMENT_ID}</code> you sent.<br><br>Check your <code>{AGREEMENT_ID}</code> parameter.</td></tr>
<tr><td>P2501</td><td>Cancel an agreement for recurring payments</td><td>This agreement cannot be cancelled. You can only cancel agreements in the <code>active</code> status.<br><br>You can read more about <a href="/recurring_payments/" anchor="understanding-agreement-status">the agreement status lifecycle</a> or the <a href="/api_reference/single_agreement_reference">API reference for getting information about an agreement</a>.</td></tr>
<tr><td>P2598</td><td>Cancel an agreement for recurring payments</td><td>There’s something wrong with GOV.UK Pay.<br><br>If there are no problems on <a href="https://payments.statuspage.io/">our status page</a>, you can <a href="https://docs.payments.service.gov.uk/support_contact_and_more_information/">contact us with your error code</a> and we’ll investigate.</td></tr>
</table>



<div style="height:1px;font-size:1px;">&nbsp;</div>

### Errors caused by payment statuses

Failed payments return a `code` value in the `state` object when you use the API to get information about that payment. These error codes explain why the payment failed.

| Error code | Meaning | Cause |
|------------|---------|-------|
| P0010 | Payment method rejected | The payment was rejected due to the payment method selected or the payment information entered by the user.<br><br>For example, the user may have failed a fraud check, a 3D Secure authentication check, or they may not have sufficient funds in their account. |
| P0020 | Payment expired | The payment timed out because the paying user did not confirm and complete the payment within 90 minutes of the payment being created.<br><br>If the paying user's bank already authorised the payment, GOV.UK Pay will automatically send a cancellation to the payment service provider. |
| P0030 | Payment cancelled by your user | The paying user selected **Cancel payment** during the payment journey.<br><br>If the paying user's bank already authorised the payment, GOV.UK Pay will automatically send a cancellation to the payment service provider. |
| P0040 | Payment was cancelled by your service | Your service cancelled the payment.<br><br>You can [read more about cancelling payments](/making_payments/#cancel-a-payment-that-s-in-progress). |
| P0050 | Payment provider returned an error | This error has multiple possible causes. For example, a configuration problem with the payment service provider, or incorrect credentials.<br><br>[Contact us with your error code](/support_contact_and_more_information/) and we'll investigate. |


## Payment status lifecycle

You can [get information about a payment](/reporting/#get-information-about-a-single-payment) using the API and see where the payment is in the payment status lifecycle. You can see this in the `state` object in the response.

For example:

```json
{
  "amount": 2000,
  "state": {
      "status": "failed",
      "finished": true,
      "message": "Payment expired",
      "code": "P0020"
  }
  ...
}
```

### Payment status meanings

The `status` value indicates where the payment is in the payment status lifecycle.

`finished` indicates whether the payment journey is finished. A `finished` payment journey does not always mean the user has made a payment. For example, a user may submit their payment details but their bank rejects the payment - the payment journey is `finished` but no payment has actually been made.

| `status`     | Meaning                                                                                                                                                                                                                                                                                                                                                                | <nobr>`finished`</nobr> |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| <nobr>`created`</nobr>    | Payment created using the API. Your user has not yet visited `next_url`.                                                                                                                                                                                                                                                                                                | <nobr>`false`</nobr>    |
| <nobr>`started`</nobr>    | Your user has visited `next_url` and is entering their payment details.                                                                                                                                                                                                                                                                                                 | <nobr>`false`</nobr>    |
| <nobr>`submitted`</nobr>  | Your user submitted payment details and went through authentication, if it was required.<br><br>The payment service provider has authorised the payment, but the user has not yet selected **Confirm**.                                                                                                                                                                    | <nobr>`false`</nobr>    |
| <nobr>`capturable`</nobr> | The payment is a delayed capture and your user has submitted their payment details and selected **Confirm**.<br><br>You can [read more about how to capture this payment](https://docs.payments.service.gov.uk/delayed_capture/#delay-taking-a-payment).                                   | <nobr>`false`</nobr>    |
| <nobr>`success`</nobr>    | Your user successfully completed the payment by selecting **Confirm**.<br><br>We redirected your user to a payment confirmation page.                                                                                                                                                                                                                                      | <nobr>`true`</nobr>     |
| <nobr>`failed`</nobr>     | The payment failed. This failure could be because the payment timed out after 90 minutes, the user's payment method was rejected, or your user cancelled the payment.<br><br>We showed the user a failure screen.<br><br>Check the [payment status error codes](/api_reference/#errors-caused-by-payment-statuses) for the possible reasons for a `failed` payment. | <nobr>`true`</nobr>     |
| <nobr>`cancelled`</nobr>  | Your service cancelled the payment using an API request or the GOV.UK Pay admin tool.<br><br>You can [read more about how to cancel payments.](https://docs.payments.service.gov.uk/making_payments/#cancel-a-payment-that-s-in-progress)                                    | <nobr>`true`</nobr>     |
| <nobr>`error`</nobr>      | Something went wrong with GOV.UK Pay or the payment service provider. The payment failed safely with no money taken from the user.<br><br>We showed the paying user a screen stating ”**We’re experiencing technical problems. No money has been taken from your account. Cancel and go back to try the payment again.**”                                       | <nobr>`true`</nobr>     |

If something went wrong with a payment, the `code` and `message` attributes in the API response can help you find out what happened.

`code` is [an API error code](/api_reference/#gov-uk-pay-api-error-codes).

`message` is a description of what went wrong.

## API versioning

We work hard to make our API updates backwards compatible and avoid breaking changes.

When we add new properties to the JSON responses, the GOV.UK Pay API version number will not change. You should develop your service to ignore properties it does not understand.

[Read more about staying up to date with GOV.UK Pay API updates](https://docs.payments.service.gov.uk/versioning/).

## Endpoints

### Payment endpoints

The following endpoints are related to payments in GOV.UK Pay:

* [`POST /v1/payments`](#create-a-payment)
* [`GET /v1/payments/{PAYMENT_ID}`](#get-information-about-one-payment)

#### Create a payment

<api-endpoint openapi-path="../../Pay-api-spec.json" endpoint="/v1/payments" method="POST">
    <response type="201">
        <sample lang="JSON">
{
    "amount": 14500,
    "description": "Pay your council tax.",
    "reference": "12345",
    "language": "en",
    "state": {
        "status": "created",
        "finished": false
    },
    "payment_id": "hu20sqlact5260q2nanm0q8u93",
    "payment_provider": "stripe",
    "created_date": "2022-03-25T13:11:29.019Z",
    "refund_summary": {
        "status": "pending",
        "amount_available": 14500,
        "amount_submitted": 0
    },
    "settlement_summary": {},
    "delayed_capture": false,
    "moto": false,
    "return_url": "https://your.service.gov.uk/completed",
    "_links": {
        "self": {
            "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93",
            "method": "GET"
        },
        "next_url": {
            "href": "https://www.payments.service.gov.uk/secure/ef1b6ff1-db34-4c62-b854-3ed4ba3c4049",
            "method": "GET"
        },
        "next_url_post": {
            "type": "application/x-www-form-urlencoded",
            "params": {
                "chargeTokenId": "ef1b6ff1-db34-4c62-b854-3ed4ba3c4049"
            },
            "href": "https://www.payments.service.gov.uk/secure",
            "method": "POST"
        },
        "events": {
            "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93/events",
            "method": "GET"
        },
        "refunds": {
            "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93/refunds",
            "method": "GET"
        },
        "cancel": {
            "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93/cancel",
            "method": "POST"
        }
    }
}
        </sample>
    </response>
</api-endpoint>

#### Get information about one payment

<api-endpoint openapi-path="../../Pay-api-spec.json" endpoint="/v1/payments/{paymentId}" method="GET">
    <response type="200">
        <sample lang="JSON">
{
  "created_date": "2019-07-11T10:36:26.988Z",
  "amount": 3750,
  "state": {
    "status": "success",
    "finished": true
  },
  "description": "Pay your council tax",
  "reference": "12345",
  "language": "en",
  "metadata": {
    "ledger_code": "AB100",
    "internal_reference_number": 200
  },
  "email": "sherlock.holmes@example.com",
  "card_details": {
    "card_brand": "Visa",
    "card_type": "debit",
    "last_digits_card_number": "1234",
    "first_digits_card_number": "123456",
    "expiry_date": "04/24",
    "cardholder_name": "Sherlock Holmes",
    "billing_address": {
        "line1": "221 Baker Street",
        "line2": "Flat b",
        "postcode": "NW1 6XE",
        "city": "London",
        "country": "GB"
    }
  },
  "payment_id": "hu20sqlact5260q2nanm0q8u93",
  "authorisation_mode": "web",
  "authorisation_summary": {
    "three_d_secure": {
      "required": true
    }
  },
  "refund_summary": {
    "status": "available",
    "amount_available": 3500,
    "amount_submitted": 500
  },
  "settlement_summary": {
    "capture_submit_time": "2019-07-12T17:15:000Z",
    "captured_date": "2019-07-12",
    "settled_date": "2019-07-12"
  },
  "delayed_capture": false,
  "moto": false,
  "corporate_card_surcharge": 250,
  "total_amount": 4000,
  "fee": 200,
  "net_amount": 3800,
  "payment_provider": "worldpay",
  "provider_id": "10987654321",
  "return_url": "https://your.service.gov.uk/completed",
  "_links": {
     "self": {
         "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93",
         "method": "GET"
        },
     "events": {
         "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93/events",
         "method": "GET"
        },
     "refunds": {
         "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93/refunds",
         "method": "GET"
     }
  }
}
        </sample>
    </response>
    <response type="404">
        <sample lang="JSON">
{
  "field": "amount",
  "code": "P0102",
  "description": "Invalid attribute value: amount. Must be less than or equal to 10000000"
}
        </sample>
    </response>
</api-endpoint>