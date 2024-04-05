<show-structure for="chapter" depth="3"/>
# Take a payment

This section tells you how to take a payment using the GOV.UK Pay API.

If you want to take payments using our no-code solution, you can use [payment links](https://www.payments.service.gov.uk/govuk-payment-pages/).

When your user makes a payment, your service should supply a `reference`. If possible, this should be a unique identifier for the payment. If your organisation already has an existing identifier for payments, you can use it as this `reference`. 

Your service will also need to supply a [`return_url`](#choose-the-return-url-and-match-your-users-to-payments). This is a URL that your service hosts for your user to return to, after their payment journey on GOV.UK Pay ends.

## Create a payment

The endpoint to create a payment with the GOV.UK Pay API is:

`POST  /v1/payments`

You must send the following parameters in your request body when creating a payment:

* the amount the user will pay, in pence (`amount`)
* a reference to help you identify the payment (`reference`)
* a human-readable description of the purpose of the payment (`description`)
* a URL to redirect your user to after they complete their payment (`return_url`)

This example request creates a £145 council tax payment:

<tabs>
<tab title="cURL">
<code-block lang="json">
curl --location "https://publicapi.payments.service.gov.uk/v1/payments" --H "Authorization: Bearer {API_TEST_KEY}" --H "Content-Type: application/json" 
--data "{
  "amount": 14500,
  "reference" : "12345",
  "description": "Pay your council tax",
  "return_url": "https://your.service.gov.uk/completed"
}"
</code-block>
</tab>
<tab title="Python">
<code-block lang="python">
//KNOW THAT I DO NOT KNOW PYTHON AND SO THIS IS JUST FROM A CONVERTOR, SOZ
import http.client
import json

conn = http.client.HTTPSConnection("publicapi.payments.service.gov.uk")
payload = json.dumps({
"amount": 12000,
"reference": "12345",
"description": "You have committed a crime against fashion and must pay a fine.",
"return_url": "https://stripe.com/docs/payments/checkout/custom-success-page"
})
headers = {
'Authorization': 'Bearer API_TEST_KEY',
'Content-Type': 'application/json'
}
conn.request("POST", "/v1/payments", payload, headers)
res = conn.getresponse()
data = res.read()
print(data.decode("utf-8"))
</code-block>
</tab>
<tab title="Go">
<code-block lang="go">
//KNOW THAT I DO NOT KNOW GO AND SO THIS IS JUST FROM A CONVERTOR, SOZ

package main

import (
"fmt"
"strings"
"net/http"
"io/ioutil"
)

func main() {

url := "https://publicapi.payments.service.gov.uk/v1/payments"
method := "POST"

payload := strings.NewReader(`{
"amount": 12000,
"reference": "12345",
"description": "You have committed a crime against fashion and must pay a fine.",
"return_url": "https://stripe.com/docs/payments/checkout/custom-success-page"
}`)

client := &http.Client {
}
req, err := http.NewRequest(method, url, payload)

if err != nil {
fmt.Println(err)
return
}
req.Header.Add("Authorization", "Bearer API_TEST_KEY")
req.Header.Add("Content-Type", "application/json")

res, err := client.Do(req)
if err != nil {
fmt.Println(err)
return
}
defer res.Body.Close()

body, err := ioutil.ReadAll(res.Body)
if err != nil {
fmt.Println(err)
return
}
fmt.Println(string(body))
}
</code-block>
</tab>
</tabs>

Depending on your integration, you can also:

* [delay taking a payment](/delayed_capture)
* [prefill some of the fields on the user's payment page](/optional_features/prefill_user_details/)
* [add custom metadata to a payment](/reporting/#add-more-information-to-a-payment-39-custom-metadata-39-or-39-reporting-columns-39)
* [use Welsh on your payment pages](/optional_features/welsh_language/)
* [take the payment over the phone](/moto_payments)

Your user has 90 minutes to complete their payment once you have created it.

You can see definitions, values, and the limits of every parameter for this endpoint in our [API reference](/api_reference/create_a_payment_reference).

#### The `amount` parameter

The `amount` parameter sets the payment amount in pence. `amount` must be a number.

The minimum amount is one pence plus your payment service provider's (PSP's) transaction fee. To see your PSP's transaction fee, check your contract by either:

- signing in to the [GOV.UK Pay admin tool](https://selfservice.payments.service.gov.uk/login) if you use Stripe
- [contacting Government Banking](mailto:serviceteam.gbs@hmrc.gov.uk) if you are using Worldpay
- asking your PSP

The maximum amount is either:

- 10,000,000 pence (£100,000)
- 500,000 pence (£5,000) if you use Government Banking

## Receiving the API response

If your request to create a payment succeeds, you'll receive a response that looks like this:

```json
{
  "created_date": "2020-03-03T16:17:19.554Z",
  "state": {
    "status": "created",
    "finished": false
  },
  "_links": {
    "self": {
      "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93",
      "method": "GET"
   },
    "next_url": {
      "href": "https://www.payments.service.gov.uk/secure/bb0a272c-8eaf-468d-b3xf-ae5e000d2231",
      "method": "GET"
    },
    ...
  },
  "amount": 14500,
  "reference" : "12345",
  "description": "Pay your council tax",
  "return_url": "https://your.service.gov.uk/completed",
  "payment_id": "hu20sqlact5260q2nanm0q8u93",
  "payment_provider": "worldpay",
  "provider_id": "10987654321",
  ...
}
```

As well as the details you sent in your request (`description`, `amount`, `reference`, `return_url`), this response includes:

{type="medium"}
`payment_id`
: the unique ID GOV.UK Pay assigned to this payment

`provider_id`
: the unique ID your PSP assigned to this payment

`created_date`
: the date and time you created the payment

`_links.next_url`
: the URL the paying user will visit to make this payment

`_links.self`
: the URL and method you can use to check the status of this payment

`_links`
: other URLs and API methods for further actions for this payment

You can see full definitions and values for every response attribute in our [API reference](api-reference)

Read more about [reporting and reconciliation](/reporting) and [payment flow](/payment_flow/#how-gov-uk-pay-works).

> Do not expose the `next_url` to anyone other than your paying user. Anyone in possession of the `next_url` could hijack your user’s payment.
> {style="warning"}

## Track the progress of a payment

You can track the progress of a payment by [getting information about it from our API](/reporting/#get-information-about-a-single-payment) (`GET /v1/payments/{PAYMENT_ID}`) or [by receiving automatic updates through a webhook](/webhooks).

The status of the payment will pass through several states until it either
succeeds or fails. Read more about the [payment status lifecycle in our API reference](https://docs.payments.service.gov.uk/api_reference/#payment-status-lifecycle).

## Choose the return URL and match your users to payments

For security reasons, GOV.UK Pay does not add the `payment_id` or outcome to your `return_url` as parameters.

You must [use HTTPS for your `return_url`](/security/#https), but you can use HTTP with test accounts.

You can match your users with payments with an encrypted cookie, or by creating a
secure random ID:

### Use an encrypted cookie

You can use an encrypted cookie containing the payment ID from GOV.UK Pay. Your
service should issue this when you create a payment, before sending your user
to `next_url`. Your users will not be able to decrypt an encrypted cookie, so a
malicious user could not alter the payment ID and intercept other users’
payments.

### Create a secure random ID

You can create a secure random ID, such as a UUID, and include this as part of
the `return_url`. You should use a different `return_url` for each payment.
Malicious users will be unable to guess securely generated UUIDs.

If you use this method, you should store your IDs safely. For example, in a
datastore mapped to the payment ID just after you create a payment.

> Make sure the way you match your returning users to payments is tamper-proof. For example, don’t store the payment ID in an unencrypted cookie or rely on a `return_url` parameter that isn’t secret (like the payment ID or reference).
> {style="warning"}

## Accepting your users when they return

If we direct your user to the `return_url`, they could have:

* paid successfully
* not paid because their card was rejected, or they selected cancel
* not paid because your service (via the API) cancelled the payment in progress
* not paid because of a technical error

Your service should use a `GET /v1/payments/{PAYMENT_ID}` request to [check the payment's status](https://docs.payments.service.gov.uk/reporting/#get-information-about-a-single-payment) when your user
reaches the `return_url`. You should provide an appropriate response based on the `state` the payment attempt.

Do not use other API requests to check the payment’s status. Other requests, such as [search payments](/reporting/#get-a-list-of-payments-matching-search-criteria), do not update immediately and so may not be up-to-date.

## Handling a user that do not complete their payment journey

Your user may close their browser or lose internet connectivity in the middle of their payment journey on GOV.UK Pay. In this event, GOV.UK Pay will not redirect your user back to your service.

You can still [check the status of these payments](/payment_flow/#7-show-a-final-page), but you should be aware that the payment will expire 90 minutes after it was created. You can only be certain of the payment’s status after this time.

If a [payment fails](/payment_flow/#show-a-failure-page), GOV.UK Pay will not let your user attempt the payment again with alternative card details. In this situation your user would need to return to the service to retry the payment. Your service should account for this.

### Resume an incomplete payment

You can send paying users back to GOV.UK Pay if they return to your service before they've finished their payment.

[Get information about the payment](/reporting/#get-information-about-a-single-payment) to check the payment's `status`. The payment is incomplete if the `status` is one of the following:

* `created`
* `started`
* `submitted`

Resume the payment by redirecting the paying user to the `next_url` in the API response.

Do not reuse the `next_url` you received in the API response when you first created the payment. Each `next_url` has a unique token, so you can only use that `next_url` once.

GOV.UK Pay will take your user to a screen that's appropriate for the status of the payment and lets the paying user finish their payment. For example, if the `status` of the payment is `submitted`, we'll show the Confirm your payment page.

You cannot resume a payment if the `status` is `failed`, `cancelled`, or `error`. [Create a new payment](/making_payments/#creating-a-payment) instead.

## Cancel a payment that’s in progress

Your user can select a link on the payment page to cancel a payment that’s in progress. Alternatively, you can make the following API request:

`POST /v1/payments/{PAYMENT_ID}/cancel`

Replace `{PAYMENT_ID}` with the ID of the payment you are cancelling.

A `204` response indicates success. Any other response indicates an error.

You can see parameters, example requests, and example responses for this endpoint in our [API reference](/api_reference/cancel_payment_reference/).

### Check if you can cancel a payment

Get information about a specific payment to check if you can cancel that payment:

`GET /v1/payments/{PAYMENT_ID}`

Replace `{PAYMENT_ID}` with the ID of the payment you are checking.

If the JSON response body contains a `"cancel"` field (in the `_links` object)
that’s not `null`, you can cancel the payment. For example:

```json
"cancel": {
  "href": "https://publicapi.payments.service.gov.uk/v1/payments/hu20sqlact5260q2nanm0q8u93/cancel",
  "method": "POST"
}
```

<seealso>
    <category ref="related">
        <a href="Take-recurring-payments.md">Taking recurring payments</a>
        <a href="api-reference.md#create-a-payment">API reference for creating a payment</a>
    </category>
    <category ref="product">
        <a href="https://www.payments.service.gov.uk/">GOV.UK Pay product pages</a>
        <a href="https://selfservice.payments.service.gov.uk/my-services">GOV.UK Pay admin tool</a>
    </category>
</seealso>