# Create a payment

<api-endpoint openapi-path="../../Pay-api-spec.json" endpoint="/v1/payments" method="POST">
    <response type="201">
        <sample>
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