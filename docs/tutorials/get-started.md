
**Get Started with Yojee APIs**
# Overview

In most use cases, the first thing when integrating with Yojee APIs is to flow orders into Yojee. To do this we will need the following steps:

1. Obtain the necessary authentication credentials
1. Make a call to create an order in Yojee

## Authentication
The Yojee API uses API Keys to authenticate requests in most cases. Most of these calls will require the credentials for a `Dispatcher` role. This is the same role as the user that logs into the `Dispatcher Portal` to perform most of the operations in Yojee through GUI.

### API Keys
To authenticae using API Keys, include the following parameters in the HTTP header.

```json json_schema
{
  "type": "API Keys",
  "properties": {
    "company_slug": {
      "type": "string",
      "example": "my_company"
    },
    "access_token": {
      "type": "string",
      "example": "4wiKfwe8afwela7WNIUAWF7w4kthwf="
    }
  }
  "required": ["company_slug", "access_token"]
}
```

**Company Slug** \
The `Company Slug` is a string to uniquely identify each instance of a customer’s company in Yojee. Each customer is assigned a slug which they will use as part of the authentication information.

**Access Token** \
A long-lived `Access Token` is generated for each `Dispatcher` account.


<!-- theme: success -->

> 💡 Obtain your `Company Slug` and `Access Token` from the Yojee team working with you.

#### Request Headers
Pass `Company Slug` and `Access Token` in the HTTP header of your API call, at the same time specifying the `Content-Type` as `application/json`.

|HTTP Header | Type | Description|
-----|----- | -----
| company_slug | string | Company Slug |
| access_token | string | Access Token for the user profile making the call |
| Content-Type | string | application/json |


## Call Create Order
---

We will be making the call to [**Create Order**](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders/post) to create an order.

### Sample Payload
Use the following sample payload for your first `Create Order` call:

``` json
 {
  "external_id": "TEST-ORDER-001",
  "external_sender_id": "Computer Corp",
  "item_steps": [
    {
      "item_id": 0,
      "order_step_id": 0,
      "step_group": 1,
      "step_sequence": 1,
      "type": "pickup"
    },
    {
      "item_id": 0,
      "order_step_id": 1,
      "step_group": 1,
      "step_sequence": 2,
      "type": "dropoff"
    }
  ],
  "steps": [
    {
      "quantity": 4,
      "address": "20 Pasir Pajang Road",
      "address2": "",
      "country": "SG",
      "state": "",
      "postal_code": "117439",
      "contact_company": "Sender Company Pte Ltd",
      "contact_name": "John Lim",
      "contact_phone": 62010000,
      "contact_email": "john@company1.example.com",
      "from_time": "2021-08-01T08:59:22.813Z",
      "to_time": "2021-08-02T07:59:59.813Z"
    },
    {
      "quantity": 4,
      "address": "1 Changi Business Park",
      "address2": "Avenue 1",
      "country": "SG",
      "state": "",
      "postal_code": "486058",
      "contact_company": "Customer Company Pte Ltd",
      "contact_name": "Peter Tan",
      "contact_phone": 62010001,
      "contact_email": "peter@company2.example.com",
      "from_time": "2021-08-03T08:59:22.813Z",
      "to_time": "2021-08-04T07:59:59.813Z"
    }
  ],
  "items": [
    {
      "description": "Laptop Computer",
      "width": 0.11,
      "length": 0.12,
      "height": 0.044,
      "weight": 334,
      "quantity": 4,
      "info": "Item information",
      "external_customer_id": "TN-001",
      "external_customer_id2": "CUSTOMER-001",
      "external_customer_id3": "",
      "payload_type": "same_day",
      "price_info": "",
      "service_type": "express",
      "volume": 1000,
      "volumetric_weight": 1,
      "price_amount": 100
    }
  ],
  "price_amount": "100",
  "price_currency": "SGD",
  "sender_type": "organisation",
}
```

<!-- theme: info -->
> ### external_sender_id
>
> Note that the `external_sender_id` needs to be a valid ID in your Yojee instance
### Making the call
There are a few ways you can make the call:

#### On the API Reference

1. Locate the `Try-it Console` on the right of the [**Create Order**](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders/post).
1. Fill in your `company_slug` and `access_token`.
1. Copy and paste the above sample payload in the **Body** section of the `Try-it Console`.
1. Click `Send Request`.

#### On the command line

1. Copy this command:
```
curl --request POST \
  --url https://umbrella-dev.yojee.com/api/v3/dispatcher/orders \
  --header 'Content-Type: application/json' \
  --header 'access_token: [ACCESS_TOKEN]' \
  --header 'company_slug: [COMPANY_SLUG]' \
  --data '{
  "external_id": "TEST-ORDER-001",
  "external_sender_id": "KCYCorp",
  "item_steps": [
    {
      "item_id": 0,
      "order_step_id": 0,
      "step_group": 1,
      "step_sequence": 1,
      "type": "pickup"
    },
    {
      "item_id": 0,
      "order_step_id": 1,
      "step_group": 1,
      "step_sequence": 2,
      "type": "dropoff"
    }
  ],
  "steps": [
    {
      "quantity": 4,
      "address": "20 Pasir Pajang Road",
      "address2": "",
      "country": "SG",
      "state": "",
      "postal_code": "117439",
      "contact_company": "S Company",
      "contact_name": "John Lim",
      "contact_phone": 62010000,
      "contact_email": "john@company1.example.com",
      "from_time": "2021-08-01T08:59:22.813Z",
      "to_time": "2021-08-02T07:59:59.813Z"
    },
    {
      "quantity": 4,
      "address": "1 Changi Business Park",
      "address2": "Avenue 1",
      "country": "SG",
      "state": "",
      "postal_code": "486058",
      "contact_company": "C Company",
      "contact_name": "Peter Tan",
      "contact_phone": 62010001,
      "contact_email": "peter@company2.example.com",
      "from_time": "2021-08-03T08:59:22.813Z",
      "to_time": "2021-08-04T07:59:59.813Z"
    }
  ],
  "items": [
    {
      "description": "Laptop Computer",
      "width": 0.11,
      "length": 0.12,
      "height": 0.044,
      "weight": 334,
      "quantity": 4,
      "info": "Item Infor",
      "external_customer_id": "TN-001",
      "external_customer_id2": "CUSTOMER-INFO-001",
      "external_customer_id3": "",
      "payload_type": "same_day",
      "price_info": "",
      "service_type": "express",
      "volume": 1000,
      "volumetric_weight": 1,
      "price_amount": 0
    }
  ],
  "price_amount": 0,
  "price_currency": "SGD",
  "sender_type": "organisation",
}'
```
1. Remember to replace **[COMPANY_SLUG]** and **[ACCESS TOKEN]** with your `Company_Slug` and `Access_Token`
## Troubleshooting
You may encounter the following messages when you make your first requests.

### Unauthorized
```json
{
    "message": "Unauthorized"
}
```
This means either your `Company_Slug` or your `Access_Token` is wrong. Check them and resend again. If the problem persists, check with the Yojee team that is working with you.
### External Sender Id
```json
{
    "data": {
        "AC0014": [
            "No corporate and sender exists. please create a sender account before create a order."
        ]
    }
}
```
The `external_sender_id` shown in the sample payload is just a sample. You will need to set up at least one sender in the Yojee `Dispatcher Portal` and create the `external_sender_id` for this sender.
Alternatively, you may also use the numeric `sender_id` listed beside your sender in the `Dispatcher Portal` and pass the Id value in the `sender_id` field in your payload.

## Verify Order is created

### Success Response
If the API call is successful, you should receive a HTTP 200 response with a payload that looks like the following sample:
```json
{
    "data": {
        "cancelled_at": null,
        "container_no": null,
        "display_price": "SGD 0",
        "external_id": "TEST-ORDER-001",
        "id": 410013,
        "inserted_at": "2021-07-25T11:13:01.524025Z",
        "number": "O-QTROJOUMTNM8",
        "order_items": [
            {
                "cod_price": null,
                "id": 434791,
                "tracking_number": "YOJ-4UABMDSOJ2VD"
            }
        ],
        "paid": false,
        "placed_by_user_profile_id": 5929,
        "price": {
            "amount": "0",
            "currency": "SGD"
        },
        "sender_id": 1750,
        "status": "created"
    },
    "message": "Created the order.",
    "warning": false
}
```

### Checking in Portal
You can also log in to the Yojee `Dispatcher Portal` with your `Dispatcher` credentials to check that the order has been created.