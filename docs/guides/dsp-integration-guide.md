## Downstream System Integrator Guide

This Downstream System Integrator Guide provides a comprehensive introduction and reference for integrating with our TCMS platform. It is designed for developers, system integrators, and technical partners who need to receive, process, and update information related to incoming orders and associated documents from our upstream system.

**Key Capabilities Covered**  
1. Registering for order and document notifications  
2. Receiving incoming orders  
3. Managing incoming orders - Accept, Reject or Cancel
4. Updating order charges
5. Retrieving order documents
6. Assigning delivery vehicles/drivers
7. Reporting delivery status changes  
8. Attaching new documents (e.g. Proof of Delivery)

**Integration Workflow Summary**  

The typical integration workflow includes:
- **Authentication**: Obtain an API key and use it to authenticate your requests.
- **Webhook Registration**: Subscribe to order and document notifications.
- **Order Handling**: Receive, accept/reject, and update orders.
- **Document Management**: Retrieve, receive, and attach documents.
- **Delivery Updates**: Assign vehicles/drivers and report delivery status changes.

## Typical lifecycle of a downstream order

1. Receive an order request from an upstream company
2. Accept the order
3. (Optional) Retrieve documents for the order if necessary (eg. Consignment note)
4. Order execution/delivery
5. (Optional) Attach/upload POD

As a downstream partner, orders flow from an upstream company into your company on TCMS. This will result in a pending order in your (downstream) company slug and trigger a `order.created` webhook event. At this point, you may query for the order's details but cannot execute on the order until you accept the order from the upstream company.

Accepted orders can be updated with additional details (eg. container number, reference numbers from your system etc), assigned to a driver, and the delivery can be executed. Delivery status changes such as pick up completion, delivery completion or reporting of delivery failure can be done through TCMS APIs. Supplementary documents such as proof of pickup or proof of delivery can be uploaded through TCMS APIs and will be made available to the upstream partners.

Upstream systems may also send down certain documents that are used for delivery, such as consignment notes, bills of lading, customs documents etc. When such a document is added by an upstream partner, you will receive a `document.created` event with the details to retrieve the document.

As a walkthrough to the above flow, we will show you how to:
1. [Register a webhook on your downstream company slug to receive events](#webhooks)
2. [Retrieve order details](#dispatcher-get-order)
3. [Accept](#accept-the-order) or [reject](#decline-the-order) an order
4. [Send delivery status changes into TCMS](#order-completion)
5. [Send documents to TCMS](#documents)

## Basic information on APIs

### Base URL

In this document we will use `[BASEURL]` to represent the base URL for the calls.

For **development and testing purposes**, please use https://umbrella-staging.yojee.com.

The base URL for the **Production API** is https://umbrella.yojee.com.

### Authentication

Most of the API calls will require the following parameters in the HTTP header:

<table style="text-align: left;">
    <tr>
        <td><strong>Parameter</strong></td>
        <td><strong>Type</strong></td>
    </tr>
    <tr>
        <td>company_slug</td>
        <td>string</td>
    </tr>
    <tr>
        <td>access_token</td>
        <td>string</td>
    </tr>
</table>

#### Company Slug

The Company Slug is a string to uniquely identify each instance of a customer's company in Yojee. Each customer is assigned a slug which they will use as part of the authentication information.

#### Access Token

A long-lived Access Token is generated for the `Dispatcher` account. This token will only change upon a change in the password of the Dispatcher account.

**Obtain this information from the Yojee team working with you.**

In this document we will use `[SLUG]` and `[TOKEN]` to represent the `company_slug` and `access_token` respectively.

## Webhooks

Webhook URLs can be registered with TCMS and the TCMS system will make HTTP calls to the URL with the relevant payload when a status update of the Order is triggered.
For the API calls, the Integration Layer needs to authenticate using the company's Dispatcher credentials. This is typically done by including the COMPANY_SLUG and the Dispatcher's ACCESS_TOKEN in the HTTP header.

### Events Supported

Events currently being supported are:

<table style="text-align: left;">
  <tr>
    <td rowspan="2"><strong>Event Name</strong></td>
    <td rowspan="2"><strong>Description</strong></td>
  </tr>
  <tr>
    <td><strong>Company</strong></td>
    <td><strong>Sender</strong></td>
  </tr>
  <tr>
    <td><a href="#event-ordercreated">order.created</a></td>
    <td>When an Order is created</td>
  </tr>
  <tr>
    <td><a href="#event-ordercancelled">order_cancelled</a></td>
    <td>When an Order Item is cancelled</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
   <tr>
    <td><a href="#event-documentcreated">document.created</a></td>
    <td>When a document is created and attached to an order/order item</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
   <tr>
    <td><a href="#event-documentupdated">document.updated</a></td>
    <td>When an attached document is updated</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
</table>

### Webhook registration
To register a webhook, use the following API endpoint:

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>POST</td>
    <td>[BASEURL]/api/v3/dispatcher/webhooks</td>
  </tr>
</table>

For full details, please click [here](https://yojee.stoplight.io/docs/developer-doc/92216c2435405-revision#webhooks).

#### Request Headers

<table style="text-align: left;">
  <tr>
    <td><strong>Parameter</strong></td>
    <td><strong>Required?</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>company_slug</td>
    <td>Y</td>
    <td>Dispatcher company slug</td>
  </tr>
  <tr>
    <td>access_token</td>
    <td>Y</td>
    <td>Access token <strong>(Dispatcher Credentials)</strong></td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use 'application/json'</td>
  </tr>
</table>

#### Request Body

#### Send as JSON data

<table style="text-align: left;">
  <tr>
    <td><strong>Parameter</strong></td>
    <td><strong>Required?</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>url</td>
    <td>Y</td>
    <td>The endpoint to register the webhook with</td>
  </tr>
  <tr>
    <td>events</td>
    <td>Y</td>
    <td>The event to register the webhook with. For multiple events, separate event name with a comma ","</td>
  </tr>
  <tr>
    <td>sender_id</td>
    <td>N</td>
    <td>Specify the Sender with the sender_id to register this webhook for</td>
  </tr>
</table>

To illustrate the params needed please take the following cURL command as reference, remember to replace the **[BASEURL]**, **[SLUG]** and **[TOKEN]**. **[WEBHOOK_URL]** will be the URL where you will receive the webhook HTTP POST calls.
**Note:** you can choose the events you want to register for and it is _not mandatory to register for all events_.

#### Sample curl command with JSON data

```shell
curl --location --request POST '[BASEURL]/api/v3/dispatcher/webhooks' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
  "url": "[WEBHOOK_URL]",
  "events": [
    "order.created",
    "order.updated",
    "order.cancelled",
    "document.created",
    "document.updated"
    ]
  }'
```

### Responses

<table style="text-align: left;">
  <tr>
    <td><strong>HTTP Response Code</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>200</td>
    <td>Webhook registration is successful</td>
  </tr>
  <tr>
    <td>422</td>
    <td>Webhook registration failed. Reason is included in the payload</td>
  </tr>
</table>

#### Sample Success Response: 200

```json
{
  "data": {
    "company_id": 1,
    "events": [
      "order.created",
      "order.updated",
      "order.cancelled",
      "document.created",
      "document.updated"
    ],
    "id": 125,
    "inserted_at": "2021-01-13T07:59:08.542587",
    "secret_token": "OQGAM5JKCXYRYPIQ6MLI5KRTQJ6OUZL6",
    "status": "active",
    "updated_at": "2021-01-13T07:59:08.542587",
    "url": "https://kcyyojee.free.beeceptor.com"
  },
  "message": "Webhook was created."
}
```

#### Sample Failure Response: 422

```json
{
  "data": {
    "EC401": ["url can't be blank"]
  }
}
```

### Webhook Payload Samples

When the respective event is triggered, a HTTP POST will be called to the registered Webhook URL with the following sample payloads.

#### Request Body

This is the format of the HTTP Post request body your system will receive in the webhook call

<table style="text-align: left;">
  <tr>
    <td><strong>Parameter</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>id</td>
    <td>Unique identifier for the event</td>
  </tr>
  <tr>
    <td>event_type</td>
    <td>Event type triggered</td>
  </tr>
  <tr>
    <td>company_slug</td>
    <td>Company slug the event belongs to</td>
  </tr>
  <tr>
    <td>webhook_id</td>
    <td>The id of the webhook that has subscribed to this event</td>
  </tr>
  <tr>
    <td>created_at</td>
    <td>Time at which the event was created. Measured in seconds since the Unix epoch</td>
  </tr>
  <tr>
    <td>data</td>
    <td>Object containing the data associated with the event. For a different event type the data is different. See sample payloads below for examples</td>
  </tr>
  <tr>
    <td>yojee_instance</td>
    <td>TCMS API environment</td>
  </tr>

</table>

#### Event: order.created

```json
{
  "data": {
    "attributes": {},
    "id": 4234066,
    "status": "created",
    "number": "O-1YV56RV3BKOM",
    "sender": {
      "id": 20086,
      "name": null,
      "type": "organisation",
      "organisation_name": "ACME Co."
    },
    "external_id": null,
    "inserted_at": "2025-06-20T07:01:52.204462Z",
    "event_type": "order.created",
    "cancelled_at": null,
    "price": {
      "currency": "SGD",
      "amount": "0.00000000"
    },
    "container_no": null,
    "order_items": [
      {
        "id": 5286199,
        "status": "created",
        "item": {
          "id": 3911862,
          "length": "11",
          "description": "BICYCLE PARTS & ACCESSORIES",
          "width": "10",
          "payload_type": "Pallet",
          "weight": "2.000",
          "height": "12",
          "volume": "3.700",
          "volumetric_weight": "0.264",
          "quantity": 1,
          "global_tracking_number": "Y-EDKKTMIHGL2B",
          "is_using_container": true
        },
        "inserted_at": "2025-06-20T07:01:52.215455Z",
        "price": null,
        "external_customer_id": "TB01816402",
        "external_customer_id2": null,
        "external_customer_id3": null,
        "tracking_number": "YOJ-2LAMWXM0ZFVB",
        "service_type": "express",
        "transfer_info": null
      }
    ],
    "completion_time": null,
    "display_price": "SGD 0",
    "service_type_id": 7402,
    "organisational_unit_id": null,
    "delivery_status": "processing",
    "delivery_status_updated_at": "2025-06-20T07:01:52.455284Z",
    "custom_field_1": null,
    "custom_field_2": "Sea",
    "custom_field_3": null,
    "custom_field_4": null,
    "external_carrier_references": [],
    "planning_status": "unassigned",
    "planning_status_updated_at": "2025-06-20T07:01:52.435859Z"
  },
  "id": "f041dcc2-07df-4c81-be97-2a614ff4741a",
  "version": "1",
  "event_type": "order.created",
  "company_slug": "your-company-slug",
  "created_at": 1750402912,
  "webhook_id": 56,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

#### Event: order.updated

```json
{
  "company_slug": "your-company-slug",
  "created_at": 1750403136,
  "data": {
    "cancelled_at": null,
    "completion_time": null,
    "container_no": null,
    "display_price": null,
    "external_id": null,
    "id": 3465,
    "inserted_at": "2025-06-20T07:01:52.215455Z",
    "number": "O-IWWYG6ADYLLW",
    "order_items": [
      {
        "external_customer_id": null,
        "external_customer_id2": null,
        "external_customer_id3": null,
        "id": 14411,
        "inserted_at": "2025-06-20T07:01:52.215455Z",
        "item": {
          "description": null,
          "global_tracking_number": "Y-V6QYRPCGCU9B",
          "height": "50",
          "id": 14279,
          "length": "50",
          "payload_type": "package",
          "quantity": null,
          "volume": "125000",
          "volumetric_weight": "25",
          "weight": "30",
          "width": "50"
        },
        "service_type": "next_day",
        "status": "created",
        "tracking_number": "YOJ-ZWRW1ZBWF5MZ",
        "transfer_info": null
      }
    ],
    "price": null,
    "sender": {
      "id": 745,
      "name": null,
      "organisation_name": "ACME Co.",
      "type": "organisation"
    },
    "status": "accepted"
  },
  "event_type": "order.updated",
  "id": "72574e3c-77e1-445f-805a-5c246d4dda55",
  "version": "2",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

#### Event: order.cancelled

```json
{
  "data": {
    "attributes": {},
    "id": 4234066,
    "status": "cancelled",
    "number": "O-1YV56RV3BKOM",
    "sender": {
      "id": 20086,
      "name": null,
      "type": "organisation",
      "organisation_name": "ACME Co."
    },
    "external_id": null,
    "inserted_at": "2025-06-20T07:01:52.204462Z",
    "cancelled_at": "2025-06-20T08:32:37.544634Z",
    "price": {
      "currency": "SGD",
      "amount": "21.62000000"
    },
    "container_no": null,
    "order_items": [
      {
        "id": 5286199,
        "status": "cancelled",
        "item": {
          "id": 3911862,
          "length": "11",
          "description": "BICYCLE PARTS & ACCESSORIES",
          "width": "10",
          "payload_type": "Pallet",
          "weight": "2.000",
          "height": "12",
          "volume": "3.700",
          "volumetric_weight": "0.264",
          "quantity": 1,
          "global_tracking_number": "Y-EDKKTMIHGL2B",
          "is_using_container": true
        },
        "inserted_at": "2025-06-20T07:01:52.215455Z",
        "price": null,
        "external_customer_id": "TB01816402-GJ1",
        "external_customer_id2": null,
        "external_customer_id3": null,
        "tracking_number": "YOJ-2LAMWXM0ZFVB",
        "service_type": "express",
        "transfer_info": null
      }
    ],
    "completion_time": null,
    "display_price": "SGD 21.62",
    "service_type_id": 7402,
    "organisational_unit_id": null,
    "delivery_status": "cancelled",
    "delivery_status_updated_at": "2025-06-20T08:32:37.858485Z",
    "custom_field_1": null,
    "custom_field_2": "Sea",
    "custom_field_3": null,
    "custom_field_4": null,
    "external_carrier_references": [
      {
        "value": "TBAZ00025056",
        "order_id": 4234066,
        "name": "carrierConsignmentId"
      }
    ],
    "planning_status": "cancelled",
    "planning_status_updated_at": "2025-06-20T08:32:37.803836Z"
  },
  "id": "7859c6d9-0ba4-468c-9f73-6cde33e0b3d6",
  "version": "1",
  "event_type": "order.cancelled",
  "company_slug": "your-company-slug",
  "created_at": 1750408357,
  "webhook_id": 56,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

#### Event: document.created

```json
{
  "data": {
    "id": 21353,
    "name": "DangerousGoodsDocument_20250619T043622.pdf",
    "file_name": "DangerousGoodsDocument_20250619T043622.pdf",
    "file_size": 48859,
    "source": "cargowise",
    "order": {
      "attributes": {},
      "id": 4233980,
      "status": "accepted",
      "number": "O-YW5KWP5RBPWF",
      "external_id": "DG19062025-II",
      "service_type_name": "Express"
    },
    "mime_type": "application/pdf",
    "updated_at": "2025-06-19T04:36:23.514093Z",
    "classification_code": "DGD",
    "document_url": "[DOCUMENT_URL]",
    "privacy": "public",
    "file_saved_at": "2025-06-19T04:36:23.564097Z",
    "attached_to": {
      "items": [],
      "order_level": true
    },
    "uploaded_by_partner": false
  },
  "id": "67386c79-477a-42ee-bc8f-02840c5cb3f3",
  "version": "2",
  "event_type": "document.created",
  "company_slug": "your-company-slug",
  "created_at": 1750307783,
  "webhook_id": 66,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

#### Event: document.updated

```json
{
  "data": {
    "id": 4323,
    "name": "Proof of delivery",
    "file_name": "proof_of_delivery.pdf",
    "file_size": 2830,
    "order": {
      "id": 3071774,
      "status": "accepted",
      "number": "O-LZONJSOECJJP",
      "external_id": "external id"
    },
    "mime_type": "application/pdf",
    "updated_at": "2025-06-19T04:38:12.223034Z",
    "classification_code": "POD",
    "document_url": "[DOCUMENT_URL]",
    "privacy": "private",
    "attached_to": { "items": [], "order_level": true },
    "uploaded_by_partner": false
  },
  "id": "ae498265-7e41-48fb-8b9d-39699bdb5b61",
  "version": "2",
  "event_type": "document.updated",
  "company_slug": "your-company-slug",
  "created_at": 1706081904,
  "webhook_id": 73,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

### Delivery Headers

HTTP POST payloads that are delivered to your webhook's configured URL endpoint will contain a header.

<table style="text-align: left;">
  <tr>
    <td><strong>Header</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>yojee-signature</td>
    <td>The HMAC hex digest of the response body. <br/>
   The HMAC hex digest is generated using the SHA256 hash function and the secret as the HMAC key.</td>
  </tr>
  <tr>
    <td>yojee-request-timestamp</td>
    <td>Timestamp used as input for verifying signature</td>
  </tr>
</table>

#### Verifying Signatures

TCMS signs the webhook events it sends to your endpoints. We do so by including a signature using a hash-based message authentication code (HMAC) with SHA-256 in each event's **yojee-signature** header. This allows you to validate that the events were sent by TCMS, not by a third party.

<!-- theme: info -->

> #### Note
>
> Before you can verify signatures, you need to retrieve your endpoint's secret from us. Each secret is unique to the endpoint to which it corresponds.

The following are the steps to verify the signature:

1. Retrieve **yojee-signature** and **yojee-request-timestamp** from header

2. Prepare signed payload string

   - Concatenate: yojee-request-timestamp (as string), the character ".", and the JSON request body.

3. Determine Expected Signature

   - Compute a HMAC with the SHA256 hash function.
     Use the endpoint's signing **secret** as the key, and use the prepared signed payload string as the message.

4. Compare signatures
   - Compare the signature in the HTTP Header to the Expected Signature. If the signatures match, compute the difference between the current timestamp and the received timestamp to decide if the difference is within the tolerance of your system.

### Responding to a Webhook

To acknowledge receipt of a webhook, your endpoint should return a **_2xx HTTP status code_**. All response codes outside this range, including **_3xx codes_**, will indicate to TCMS that you did not receive the webhook.

We will attempt to deliver your webhooks for up to five times with an exponential back off, according to the following table:

<table style="text-align: left;">
  <tr>
    <td><strong>Retry</strong></td>
    <td><strong>Seconds</strong></td>
  </tr>
  <tr>
    <td>1</td>
    <td>120</td>
  </tr>
  <tr>
    <td>2</td>
    <td>360</td>
  </tr>
  <tr>
    <td>3</td>
    <td>840</td>
  </tr>
  <tr>
    <td>4</td>
    <td>1800</td>
  </tr>
  <tr>
    <td>5</td>
    <td>3720</td>
  </tr>
</table>

#### Best Practice

If your webhook script performs complex logic, or makes network calls, it's possible that the script would time out before TCMS sees its complete execution. For that reason, you might want to have your webhook endpoint immediately acknowledge receipt by returning a **_2xx HTTP status code_**, and then perform the rest of its duties.

> ### Note
>
> See the section on **Basic Information on APIs - Authentication** at the end of this document for more information on authentication.

## Order details
```mermaid
flowchart TD
  A[Upstream Company] -->|Create Order| B[Downstream Company]
  B -->|Pending Order Created<br/>Triggers order.created webhook| C[Order Status: Created]
```
### Dispatcher Get Order

This API call will retrieve order information based on either order number or order external id.

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>GET</td>
    <td>[BASEURL]/api/v4/company/order</td>
  </tr>
</table>

#### Request Headers

<table style="text-align: left;">
  <tr>
    <td><strong>Parameter</strong></td>
    <td><strong>Required?</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>company_slug</td>
    <td>Y</td>
    <td>Dispatcher company slug</td>
  </tr>
  <tr>
    <td>access_token</td>
    <td>Y</td>
    <td>Access token</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use 'application/json'</td>
  </tr>
</table>

###### Sample Curl Command

```shell
curl --location -g --request GET '[BASEURL]/api/v4/company/order?number=O-K02IHA1XHHWU' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```

## Accepting/rejecting orders
```mermaid
flowchart TD
  C[Order Status: Created]

  C -->|Query Order Details| E[Get Order Information]
  C --> D[Accept Order API]
  C --> F[Reject Order API]
  
  D -->|Success| H[Order Status: Accepted]
  F -->|Success| G[Order Status: Rejected]
```
### Accept the order

Call this API to **accept** the order from upstream partner.

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>POST</td>
    <td>[BASEURL]/api/v4/company/order/bulk_accept</td>
  </tr>
</table>

#### Request Headers

<table style="text-align: left;">
  <tr>
    <td><strong>Parameter</strong></td>
    <td><strong>Required?</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>company_slug</td>
    <td>Y</td>
    <td>Dispatcher company slug</td>
  </tr>
  <tr>
    <td>access_token</td>
    <td>Y</td>
    <td>Access token</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use 'application/json'</td>
  </tr>
</table>

#### Request Body
```json
{
  "data": {
      "order_number": "O-7CKFRWUIXFTU",
      "carrier_references": [{"name": "BookingID", "value": "A123"}]
    }
  
}
```

###### Sample Curl Command

```shell
curl --location --request POST '[BASEURL]/api/v4/company/order/bulk_accept' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--data-raw '{
    "data": [
        
         {
            "order_number": "O-7CKFRWUIXFTU",
            "carrier_references": [{"name": "BookingID", "value": "A123"}]
        }
    ]

}'
```

### Get Reason codes

Call this api to get the list of reason codes, that can be used in order cancellation / rejection apis.

###### Sample Curl Command

```shell
curl --location -g --request GET '[BASEURL]/api/v4/company/reason_codes' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


### Decline the order

Call this API to **reject** the order from upstream partner.

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>POST</td>
    <td>[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/bulk_reject_order</td>
  </tr>
</table>

#### Request Headers

<table style="text-align: left;">
  <tr>
    <td><strong>Parameter</strong></td>
    <td><strong>Required?</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>company_slug</td>
    <td>Y</td>
    <td>Dispatcher company slug</td>
  </tr>
  <tr>
    <td>access_token</td>
    <td>Y</td>
    <td>Access token</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use 'application/json'</td>
  </tr>
</table>

#### Request Body

```json
{
  "data": {
      {
        "order_numbers":["O-JYUTHCO2EVO8","O-AGIO5BYIZKBQ"],
        "cancelled_notes":"",
        "reason_code":""
      }
    }
}
```

###### Sample Curl Command

```shell
curl --location --request POST '[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/bulk_reject_order' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--data-raw '{"order_numbers":["O-JYUTHCO2EVO8","O-AGIO5BYIZKBQ"],"cancelled_notes":"Insufficient capacity","reason_code":"INCP"}'
```

## Charges
### Creating/Updating charges

Call this API to **create / update** charges linked to an order

###### Sample Curl Command

```shell
curl --location --request PUT '[BASEURL]/api/v4/company/integration/order/{number}/charges' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--data-raw '{
  "number": "O-123",
  "charges": [
    {
      "charge_code": "BFC",
      "currency": "AUD",
      "description": "Route Price",
      "sell": 3
    },
    {
      "charge_code": "SURCHARGES",
      "currency": "AUD",
      "description": "Carrier Surcharges",
      "sell": 20
    }
  ]
}'
```

### Get Rate Charge Types

Call this API to **get** rate charge types

###### Sample Curl Command

```shell
curl --location --request GET '[BASEURL]/api/v3/dispatcher/rates/rate_charge_types' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
```

## Driver Assignment
```mermaid
flowchart TD
  H[Order Status: Accepted]

  H -->|Update Order Details| L[Update Order API]
  H -->|Assign Driver| M[Assign Driver API]
    
  M -->|Success| N[Order Status: Assigned]
```
### Dispatcher Assign Driver to tasks


Call this API to assign a Driver to tasks.

###### Sample Curl Command

```shell
curl --location --request POST '[BASEURL]/api/v4/company/delivery_execution/assign' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
  "data": 
    [
      {
        "orders": [
          {
            "number": "O-MXNCVU9PEFC2",
            "step_sequences": [0]
          }
        ],
        "driver": {
          "assigned_time": "2025-06-19T06:35:12.78"
        }
      }
    ]
    
}'
```

**Step sequence refers to a specific task in the delivery journey, starting from 0. For example, for a single leg delivery, the pick up would be step_sequence 0 while the drop off would be step_sequence 1.**

**Tasks in the same leg will be assigned together. In the above example, since the pick up task is assigned to a driver, the associated drop off task will be assigned to the driver as well since they are part of the same leg**

###### Sample Response

```json
{
  "data": [
    {
      "data": {
        "selectors": [
          {
            "number": "O-EMEM10XJADI6"
          }
        ]
      },
      "status": "success"
    }
  ]
}
```

## Order completion

```mermaid
flowchart TD
  N[Order Status: Assigned]

  N -->|Start Delivery| O[Delivery Execution]
  O -->|Pickup Complete| P[Complete Task API<br/>Step Sequence: 0]
  P -->|Success| Q[Order Status: Pickup Complete]
  
  Q -->|Delivery Complete| R[Complete Task API<br/>Step Sequence: 1]
  R -->|Success| S[Order Status: Delivery Complete]
```

Order completion is done by a background job. The first call is to send the parameters to the background job for execution, and the second call is to get the status of the background job to see the outcome.

### Dispatcher mark the task / order as completed

###### Sample Curl Command

```shell
curl --location --request POST '[BASEURL]/api/v4/company/delivery_execution/complete' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
    "data": [
        {
            "orders": [
                {
                    "number": "O-3LBETYLWASX1",
                    "step_sequence": 0
                }
            ],
            "event_data": {
                "event_type": "complete",
                "event_time": "2024-03-09T14:25:33.55",
                "departure_time": "2024-03-10T13:00:00"
            }
        }
    ]
}'
```

**Step sequence refers to a specific task in the delivery journey, starting from 0. For example, for a single leg delivery, the pick up would be step_sequence 0 while the drop off would be step_sequence 1**

###### Sample Response

```json
{
    "data": {
        "id": "6bbaf969ab3041eeb1e70ddab60a4e90",
        "type": "complete_or_report",
        "inserted_at": "2025-06-19T03:58:05.090793"
    }
}
```

### Dispatcher check order completion status

Call this api to check if the completion status update is successful.

###### Sample Curl Command

```shell
curl --location --request POST '[BASEURL]/api/v4/company/delivery_execution/bg_status' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
    "id": "6bbaf969ab3041eeb1e70ddab60a4e90"
}'
```
#### Sample response

```json
{
    "data": {
        "id": "6bbaf969ab3041eeb1e70ddab60a4e90",
        "total": null,
        "type": "complete_or_report",
        "progress": null,
        "result": [
            {
                "data": {
                    "selectors": [
                        {
                            "number": "O-3LBETYLWASX1",
                            "step_sequence": 0
                        }
                    ]
                },
                "status": "success"
            }
        ],
        "inserted_at": "2025-06-19T03:58:05.090793",
        "updated_at": "2025-06-19T03:58:07.207055",
        "completed_at": "2025-06-19T03:58:07.207055",
        "cancelled_at": null,
        "processed": null,
        "failed_at": null
    }
}
```

## Documents
```mermaid
flowchart TD
    UC[Upstream Company<br/>in TCMS]
    TCMS[TCMS System]
    DSS[Downstream System<br/>DSS]
    
    UC -->|1.Attach Document| TCMS
    TCMS -->|2.Triggers<br/>document.created webhook| DSS
    DSS -->|"3.(Optional) Call<br/>Get Documents API"| TCMS
    TCMS -->|4.Return document data| DSS
```
### Get documents

Call this api to get the documents attached to the order;

###### Sample Curl Command

```shell
curl --location --request GET '[BASEURL]/api/v3/dispatcher/documents?order_number=O-K02IHA1XHHWU' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' 
```

### Attach documents to the order

1. Upload the document to TCMS and get a pre-signed url
2. Send the presigned url and the order details to attach the document to the order

### Upload document to TCMS and get presigned url

Call this api to obtain a pre-signed url to upload your document to S3;

###### Sample Curl Command

```shell
curl --location --request GET '[BASEURL]/api/v3/dispatcher/documents/presigned_url?filename={filename}' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json'
```

### Attach documents to the order
```mermaid
flowchart TD
  Q[Order Status: Pickup/Delivery Complete]

  Q -->|Optional| V[Upload Document API]
  
  V -->|Success| Z[Document Available to Upstream]
```

Call this api to attach documents to the order;

###### Sample Curl Command

```shell
curl --location --request POST '[BASEURL]/api/v3/dispatcher/documents' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json'
--data-raw '{
    "order_number": "O-3LBETYLWASX1",
    "name": "POD.pdf",
    "document_url": <Your document URL>,
    "classification_code": "POD",
    "privacy": "public"
}'
```

### Get list of document classification codes

###### Sample Curl Command

```shell
curl --location --request GET '[BASEURL]/api/v4/dispatcher/document_classifications' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' 
```
