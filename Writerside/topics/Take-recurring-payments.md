# Take recurring payments

You can take recurring payments from a user using GOV.UK Pay. The user consents to making recurring payments, enters into an ‘agreement’ with your service, and provides their payment details. The agreement lets you take further payments through our API without the user interacting with your service again.

You'll need a technical team to integrate with our API and start taking recurring payments. We do not currently offer a no-code way of taking recurring payments.

Your service has [additional responsibilities when taking recurring payments](#1-understand-what-your-service-is-responsible-for).

You should be familiar with [the standard GOV.UK Pay payment flow](/payment_flow) before reading about recurring payments.

This page explains the recurring payment flow, your responsibilities, and how to set up and use agreements to take recurring payments.

## How recurring payments work

1. You create an ‘agreement’ between your service and your user through our API. This is an agreement for you to use your user’s payment details to make ongoing payments for your service.

1. You set up the agreement by getting your user’s consent and taking their first payment. You instruct our API to save their payment details. This makes the agreement for recurring payments active.

1. You take recurring payments by referencing the active agreement in your API requests when creating payments. The agreement lets GOV.UK Pay authorise the payment with the user’s saved payment details.

1. You cancel the agreement when it is no longer needed or when the user requests a cancellation. GOV.UK Pay cannot use the cancelled agreement to authorise payments. You can set the agreement up again by taking another payment and instructing our API to save the user’s payment details.

You can [read more about the lifecycle of agreements](#understanding-agreement-status).

### Understand the technical flow of recurring payments

In this integration example, the service creates an agreement when a user says they want to make recurring payments, then uses that agreement to continue taking payments.

![](recurring-payments-success-sequence.svg)

1. A user consents to your service taking recurring payments. You record their consent.

2. Your service creates an 'agreement' for recurring payments through the <tooltip term="create-agreement"><code>POST /v1/agreements</code></tooltip> endpoint.

3. GOV.UK Pay returns a `201` successful response that includes an `agreement_id`.

4. Your service saves the `agreement_id` from the response.

5. Your service [sets up the agreement by creating the user's first payment](#set-up-an-agreement-for-recurring-payments) using the <tooltip term="create-payment"><code>POST /v1/payments</code></tooltip> endpoint.<p>This payment follows [the standard GOV.UK Pay payment flow](/payment_flow), but you must include the `set_up_agreement` parameter with the `agreement_id` as its value.

6. GOV.UK Pay returns a `201` successful response to your payment creation request.

7. Your service directs the user to GOV.UK Pay to make their first payment. The user follows the standard payment flow.

8. The user completes their first payment and GOV.UK Pay returns them to your service. Your PSP stores the user's payment details.

9. Your service creates recurring payments using the <tooltip term="create-payment"><code>POST /v1/payments</code></tooltip> endpoint.<p>In your request body, you  set `authorisation_mode` to `agreement` and include `"agreement_id": "{AGREEMENT_ID}"`, replacing `{AGREEMENT_ID}` with the agreement you set up earlier.

10. GOV.UK Pay authorises the payment using the user's payment details your PSP saved during the first payment.

## Before you start taking recurring payments

You need to understand your service’s responsibilities and turn on recurring payments with your payment service provider (PSP) before you can take recurring payments.

### 1. Understand what your service is responsible for

You have more responsibilities when taking recurring payments than when taking standard payments through GOV.UK Pay.

Your additional responsibilities include:

* managing the direct relationship between your service and users

* providing a system to authenticate users when logging in to your service – if you’re eligible, we recommend using [GOV.UK One Login](https://www.sign-in.service.gov.uk/) to do this

* making the terms, conditions, and purpose of the recurring payments clear to your users so they can consent and you can record this consent

* setting up, managing, and meeting the schedule to take payments from users

* telling users when you're going to take a recurring payment and how much the payment will be

* telling users if you change the amount or timing of their recurring payments

* letting users manage or cancel recurring payments

### 2. Turn on recurring payments with GOV.UK Pay

Email [govuk-pay-support@digital.cabinet-office.gov.uk](mailto:govuk-pay-support@digital.cabinet-office.gov.uk) and ask us to turn on recurring payments on your test account.

### 3. Turn on recurring payments with your payment service provider

If your PSP is Worldpay, you must turn on tokenisation on your account before you can take recurring payments. Contact your Worldpay relationship manager and ask them to turn on tokenisation.

If your PSP is Stripe, you do not need to do anything before you start setting up recurring payments.

When you've tested your recurring payments service, follow our [go live documentation](/switching_to_live) to start taking payments from your users.

## Create an agreement for recurring payments

An agreement represents an understanding between you and your paying user that you’ll use their payment details to make ongoing payments for a service.

Use this endpoint to create an agreement:

<tooltip term="create-agreement"><code>POST /v1/agreements</code></tooltip>

This example request would create an agreement to take council tax from the paying user `user-3fb81107-76b7-4910`:

<tabs>
<tab title="cURL">
<code-block lang="json">
curl --location "https://publicapi.payments.service.gov.uk/v1/agreements" --H "Authorization: Bearer {API_TEST_KEY}" --H "Content-Type: application/json" 
--data "{
"reference": "CT-22-23-0001",
"description": "Dorset Council 2022/23 council tax subscription.",
"user_identifier": "user-3fb81107-76b7-4910"
}"
</code-block>
</tab>
<tab title="Python">
<code-block lang="python">
//KNOW THAT I DO NOT KNOW PYTHON AND SO THIS IS JUST FROM A CONVERTOR, SOZ
import requests
import json

url = "https://publicapi.payments.service.gov.uk/v1/agreements"

payload = json.dumps({
"reference": "CT-22-23-0001",
"description": "Dorset Council 2022/23 council tax subscription.",
"user_identifier": "user-3fb81107-76b7-4910"
})
headers = {
'Content-Type': 'application/json',
'Accept': 'application/json',
'Authorization': 'Bearer api_test_key_123456789'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)
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

url := "https://publicapi.payments.service.gov.uk/v1/agreements"
method := "POST"

payload := strings.NewReader(`{
"reference": "CT-22-23-0001",
"description": "Dorset Council 2022/23 council tax subscription.",
"user_identifier": "user-3fb81107-76b7-4910"
}`)

client := &http.Client {
}
req, err := http.NewRequest(method, url, payload)

if err != nil {
fmt.Println(err)
return
}
req.Header.Add("Content-Type", "application/json")
req.Header.Add("Accept", "application/json")
req.Header.Add("Authorization", "Bearer api_test_key_123456789")

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

In the body of your request, you must include:

`description` (required)
: A human-readable description of the purpose of the agreement. We'll show this to your user when they enter their payment details.

`reference` (required)
: A reference for this agreement.

`user_identifier` (optional)
: An identifier to help your service identify the user.

You’ll receive a response like this:

```json
{
"agreement_id": "cgc1ocvh0pt9fqs0ma67r42l58",
"reference": "CT-22-23-0001",
"description" : "Dorset Council 2022/23 council tax subscription.",
"status": "CREATED",
"user_identifier": "user-3fb81107-76b7-4910",
"created_date": "2022-07-08T14:33:00.000Z",
}
```
{collapsible="true" collapsed-title="Create agreement response"}

This response contains:

`agreement_id`
: A unique ID GOV.UK Pay automatically associated with this agreement.

`reference`, `description`, `user_identifier`
: The reference, description, and user identifier (if any) you associated with this agreement

`status`
: The [agreement’s current status](#understanding-agreement-status).

`created_date`
: The date and time you created the agreement.

You can see definitions and possible values of every parameter and attribute in [our reference documentation for this endpoint](/api_reference/create_an_agreement_reference).

## Set up an agreement for recurring payments

### Update a user's recurring payment card details

## Taking recurring payments

### Handling failed recurring payments

#### Handling failed recurring payments that can be retried

#### Handling failed recurring payments that cannot be retried

### Use GOV.UK Pay's optional features with recurring payments {collapsible="true"}

### Use idempotency to avoid double-charging a user {collapsible="true"}

### Use webhooks to receive automatic event updates {collapsible="true"}

### Test your recurring payments integration

## Reporting on recurring payments

### Reporting using the GOV.UK Pay admin tool

### Searching for agreements for recurring payments

### Getting information about an agreement for recurring payments

### Finding recurring payments by agreement

### Understanding agreement status

## Cancel an agreement for recurring payments

### Cancel an agreement through the GOV.UK Pay admin tool

### Cancel an agreement through the API

