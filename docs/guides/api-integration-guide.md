**API Integration Introduction Guide**

# Overview

The following diagram details the data flow between an external system and Yojee.

The **three primary** RESTFul API calls related to this project are:

1. [Multi-leg Order Creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders_multi_leg/post)

2. [Cancel Order](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders~1cancel/post)

3. [Create Webhook](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-webhook-api.yaml/paths/~1api~1v3~1dispatcher~1webhooks/post)

## <span style="text-decoration:underline;">Basic information on APIs</span>

### Base URL

[https://umbrella.yojee.com](https://umbrella.yojee.com) is the base URL for the PRODUCTION API.

For development and testing purposes, please use [https://umbrella-staging.yojee.com](https://umbrella-staging.yojee.com).

In this document we will use [BASEURL] to represent the base URL.

### Mandatory Parameters

Most of the API calls will require the following parameters in the header:

| Parameter    | Type   |
| ------------ | ------ |
| company_slug | string |
| access_token | string |

Obtain this information from the Yojee team working with you. In this document we will use [SLUG] and [TOKEN] to represent the company_slug and access_token respectively.

## <span style="text-decoration:underline;">Multi-leg Order Creation</span>

This API call will create an order in Yojee.

<table>
  <tr>
    <td><strong>Method</strong>
    </td>
    <td><strong>Endpoint</strong>
    </td>
  </tr>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/dispatcher/orders_multi_leg
   </td>
  </tr>
</table>

### Request Headers

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>company_slug
   </td>
   <td>Y
   </td>
   <td>Dispatcher company slug
   </td>
  </tr>
  <tr>
   <td>access_token
   </td>
   <td>Y
   </td>
   <td>Access token
   </td>
  </tr>
  <tr>
   <td>Content-Type
   </td>
   <td>Y
   </td>
   <td>Use ‘application/json’
   </td>
  </tr>
</table>

### Request Body

#### Main

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>items
   </td>
   <td>Y
   </td>
   <td>An array of items to be transported in this order. See <strong>Item</strong> below for the item object definition
   </td>
  </tr>
  <tr>
   <td>steps
   </td>
   <td>Y
   </td>
   <td>An array of steps (pickup/dropoff) for the order. See <strong>Step</strong> below for the step object definition
   </td>
  </tr>
  <tr>
   <td>item_steps
   </td>
   <td>Y
   </td>
   <td>An array defining the sequence for the steps for the items to be transported. See <strong>ItemStep</strong> below for the item_step object definition
   </td>
  </tr>
  <tr>
   <td>sender_id
   </td>
   <td>Y
   </td>
   <td>The id of the sender for this order
   </td>
  </tr>
  <tr>
   <td>sender_type
   </td>
   <td>Y
   </td>
   <td>For most integration this is ‘organisation’
   </td>
  </tr>
  <tr>
   <td>placed_by_user_profile_id
   </td>
   <td>N
   </td>
   <td>Only in cases where there are Corporate Users created under Corporate Sender
   </td>
  </tr>
  <tr>
   <td>external_id
   </td>
   <td>N
   </td>
   <td>For entering any tracking number from partner system
   </td>
  </tr>
  <tr>
   <td>container_no
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
</table>

#### Item

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>description
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>height
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>length
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>width
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>weight
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>volume
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>volumetric _weight
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>payload_type
   </td>
   <td>Y
   </td>
   <td>Item Type defined in Yojee Dispatcher
   </td>
  </tr>
  <tr>
   <td>service_type
   </td>
   <td>Y
   </td>
   <td>Service Type define in Yojee Dispatcher
   </td>
  </tr>
  <tr>
   <td>service_type_id
   </td>
   <td>Y
   </td>
   <td>Service Type ID of Service Type
   </td>
  </tr>
  <tr>
   <td>external_customer_id
   </td>
   <td>N
   </td>
   <td>For storing external reference
   </td>
  </tr>
  <tr>
   <td>external_customer_id2
   </td>
   <td>N
   </td>
   <td>For storing external reference
   </td>
  </tr>
  <tr>
   <td>external_customer_id3
   </td>
   <td>N
   </td>
   <td>For storing external reference
   </td>
  </tr>
  <tr>
   <td>info
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
</table>

#### Step

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>position
   </td>
   <td>Y
   </td>
   <td>Start from 0 and increment 1 for each step
   </td>
  </tr>
  <tr>
   <td>address
   </td>
   <td>Y
   </td>
   <td>Pickup/Dropoff Address
   </td>
  </tr>
  <tr>
   <td>address2
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>country
   </td>
   <td>Y
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>state
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>postal_code
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>contact_company
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>contact_name
   </td>
   <td>N
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>contact_phone
   </td>
   <td>N
   </td>
   <td>The phone number if SMS notification is required. The corresponding notification setting also needs to be enabled in Yojee Dispatcher
   </td>
  </tr>
  <tr>
   <td>contact_email
   </td>
   <td>N
   </td>
   <td>The email address if email notification is required. The corresponding notification setting also needs to be enabled in Yojee Dispatcher
   </td>
  </tr>
  <tr>
   <td>from_time
   </td>
   <td>Y
   </td>
   <td>Start of time slot for this step
   </td>
  </tr>
  <tr>
   <td>to_time
   </td>
   <td>Y
   </td>
   <td>End of time slot for this step
   </td>
  </tr>
  <tr>
   <td>lat
   </td>
   <td>N
   </td>
   <td>If lat/lng for the location is not provided, Yojee will attempt to geocode using the address
   </td>
  </tr>
  <tr>
   <td>lng
   </td>
   <td>N
   </td>
   <td>If lat/lng for the location is not provided, Yojee will attempt to geocode using the address
   </td>
  </tr>
</table>

### ItemStep

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>order_step_id
   </td>
   <td>Y
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>item_id
   </td>
   <td>Y
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>step_group
   </td>
   <td>Y
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>step_sequence
   </td>
   <td>Y
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>type
   </td>
   <td>Y
   </td>
   <td>‘pickup’ or ‘dropoff’
   </td>
  </tr>
</table>

### Sample Request Body Payload

```json
{
  "items": [
    {
      "description": null,
      "height": 333,
      "length": 333,
      "weight": 33,
      "width": 333,
      "volume": 36926037,
      "volumetric_weight": 7385.2074,
      "payload_type": "package",
      "service_type": "region2",
      "service_type_id": 325,
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "info": null
    }
  ],
  "steps": [
    {
      "position": "0",
      "address": "Outram Road, Singapore General Hospital, Singapore",
      "address2": null,
      "country": "Singapore",
      "state": "",
      "postal_code": "169608",
      "contact_company": null,
      "contact_name": null,
      "contact_phone": "+65",
      "contact_email": null,
      "from_time": "2020-12-28T17:00:00.000Z",
      "to_time": "2020-12-29T17:00:00.000Z",
      "lat": 1.279677,
      "lng": 103.835984
    },
    {
      "position": "1",
      "address": "Singapore Zoo, Singapore",
      "address2": null,
      "country": "Singapore",
      "state": "",
      "postal_code": "729826",
      "contact_company": null,
      "contact_name": null,
      "contact_phone": "+65",
      "contact_email": null,
      "from_time": "2021-01-01T17:00:00.000Z",
      "to_time": "2021-01-04T17:30:00.000Z",
      "lat": 1.4043485,
      "lng": 103.793023
    }
  ],
  "item_steps": [
    {
      "order_step_id": 0,
      "item_id": 0,
      "step_group": 1,
      "step_sequence": 1,
      "type": "pickup"
    },
    {
      "order_step_id": 1,
      "item_id": 0,
      "step_group": 1,
      "step_sequence": 2,
      "type": "dropoff"
    }
  ],
  "sender_id": 1437,
  "sender_type": "organisation",
  "placed_by_user_profile_id": "1437",
  "external_id": null,
  "container_no": null
}
```

### Responses

<table>
  <tr>
   <td><strong>HTTP Response Code</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>200
   </td>
   <td>Order creation is successful
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Order creation failed. Reason is included in the payload
   </td>
  </tr>
</table>

#### Sample Success Response

```json
{
  "data": {
    "cancelled_at": null,
    "container_no": null,
    "display_price": null,
    "external_id": null,
    "id": 245215,
    "inserted_at": "2021-01-13T17:09:52.218457Z",
    "number": "O-1NEB0HTXJFVW",
    "order_items": [
      {
        "cod_price": null,
        "id": 268096,
        "tracking_number": "YOJ-7HSZ5YYUJP1A"
      }
    ],
    "paid": false,
    "placed_by_user_profile_id": 1437,
    "price": null,
    "sender_id": 1437,
    "status": "created"
  },
  "message": "Order created!"
}
```

#### Same Failure Response

HTTP Response Code: 422

```json
{
  "data": {
    "00000": [
      "item_steps is referring the items or steps which doesn't exist or steps or items has extra entries that item_steps is not referring to"
    ]
  }
}
```

##

## <span style="text-decoration:underline;">Order Cancellation</span>

This API call will cancel an order in Yojee.

<table>
  <tr>
    <td><strong>Method</strong>
    </td>
    <td><strong>Endpoint</strong>
    </td>
  </tr>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/dispatcher/orders/cancel
   </td>
  </tr>
</table>

### Request Headers

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>company_slug
   </td>
   <td>Y
   </td>
   <td>Dispatcher company slug
   </td>
  </tr>
  <tr>
   <td>access_token
   </td>
   <td>Y
   </td>
   <td>Access token
   </td>
  </tr>
  <tr>
   <td>Content-Type
   </td>
   <td>Y
   </td>
   <td>Use ‘application/json’
   </td>
  </tr>
</table>

### Request Body

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>order_number
   </td>
   <td>Y
   </td>
   <td>The Yojee order number to cancel
   </td>
  </tr>
  <tr>
   <td>external_id
   </td>
   <td>N
   </td>
   <td>An external ID for the order entered during Order Creation. This can be the tracking number from partner system
   </td>
  </tr>
</table>

### Responses

<table>
  <tr>
   <td><strong>HTTP Response Code</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>200
   </td>
   <td>Order cancellation is successful
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Order cancellation failed. Reason is included in the payload
   </td>
  </tr>
</table>

#### Sample Success Response

```json
{
  "message": "Order canceled."
}
```

#### Same Failure Response

HTTP Response Code: 422

```json
{
  "data": {
    "BK0510": ["Order not found"]
  }
}
```

## <span style="text-decoration:underline;">Webhooks</span>

Webhook URLs can be registered with Yojee and the Yojee system will make HTTP calls to the URL with the relevant payload when a status update of the Order is triggered.

### Events Supported

Events currently being supported are:

<table>
  <tr>
   <td><strong>Event Name</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>sender.created
   </td>
   <td>When a Sender is created
   </td>
  </tr>
  <tr>
   <td>order.created
   </td>
   <td>When an Order is created
   </td>
  </tr>
  <tr>
   <td>task.accepted
   </td>
   <td>When a task (assigned to a driver - pickup / drop-off) is accepted
   </td>
  </tr>
  <tr>
   <td>task.completed
   </td>
   <td>When a task (assigned to a driver - pickup / drop-off) is completed
   </td>
  </tr>
  <tr>
   <td>task.failed
   </td>
   <td>When a task (assigned to a driver - pickup / drop-off) is reported
   </td>
  </tr>
  <tr>
   <td>task.reassigned
   </td>
   <td>When a task (assigned to a driver & not accepted yet) is reassigned
   </td>
  </tr>
  <tr>
   <td>task.transferred
   </td>
   <td>When a task is transferred to an upstream / downstream partner
   </td>
  </tr>
  <tr>
   <td>order_item.cancelled
   </td>
   <td>When an Order Item is cancelled
   </td>
  </tr>
  <tr>
   <td>driver.arrived
   </td>
   <td>When a driver has arrived at the pickup / drop-off destination point
   </td>
  </tr>
  <tr>
   <td>driver.departed
   </td>
   <td>When a driver has departed from the pickup / drop-off destination point
   </td>
  </tr>
  <tr>
   <td>order.transfer.rejected
   </td>
   <td>When a transferred Order is rejected by the downstream partner
   </td>
  </tr>
</table>

To register a webhook, use the following API endpoint:

<table>
  <tr>
    <td><strong>Method</strong>
    </td>
    <td><strong>Endpoint</strong>
    </td>
  </tr>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/dispatcher/webhooks
   </td>
  </tr>
</table>

### Request Headers

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>company_slug
   </td>
   <td>Y
   </td>
   <td>Dispatcher company slug
   </td>
  </tr>
  <tr>
   <td>access_token
   </td>
   <td>Y
   </td>
   <td>Access token
   </td>
  </tr>
</table>

### Request Body

<!-- theme: info -->

> #### Note
>
> You can send as a form data or send as JSON in the POST request body

#### Send as form data

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>url
   </td>
   <td>Y
   </td>
   <td>The endpoint to register the webhook with
   </td>
  </tr>
  <tr>
   <td>events[]
   </td>
   <td>Y
   </td>
   <td>The event to register the webhook with. For multiple events, repeat with another events[] parameter
   </td>
  </tr>
</table>

#### Send as JSON data

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Required?</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>url
   </td>
   <td>Y
   </td>
   <td>The endpoint to register the webhook with
   </td>
  </tr>
  <tr>
   <td>events
   </td>
   <td>Y
   </td>
   <td>The event to register the webhook with. For multiple events, separate event name with a comma ","
   </td>
  </tr>
</table>

To illustrate the params needed please take the following cURL command as reference, remember to replace the **[BASEURL]**, **[SLUG]** and **[TOKEN]**. **[WEBHOOK_URL]** will be the URL where you will receive the webhook HTTP POST calls.
**Note:** you can choose the events you want to register for and it is _not mandatory to register for all events_.

#### Sample curl command with form data

```shell
curl --location --request POST '[BASEURL]/api/v3/dispatcher/webhooks' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--form 'url="[WEBHOOK_URL]"' \
--form 'events[]="sender.created"' \
--form 'events[]="task.accepted"' \
--form 'events[]="task.reassigned"' \
--form 'events[]="task.transferred"' \
--form 'events[]="task.completed"' \
--form 'events[]="task.failed"' \
--form 'events[]="driver.arrived"' \
--form 'events[]="driver.departed"' \
--form 'events[]="order.created"' \
--form 'events[]="order_item.cancelled"' \
--form 'events[]="order_item.cancelled"'
```

#### Sample curl command with JSON data

```shell
curl --location --request POST '[BASEURL]/api/v3/dispatcher/webhooks' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
  "url": "[WEBHOOK_URL]",
  "events": [
    "sender.created",
    "task.accepted",
    "task.reassigned",
    "task.transferred",
    "task.completed",
    "task.failed",
    "driver.arrived",
    "driver.departed",
    "order.created",
    "order_item.cancelled",
    "order_item.cancelled"
    ]
  }'
```

### Responses

<table>
  <tr>
   <td><strong>HTTP Response Code</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>200
   </td>
   <td>Webhook registration is successful
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Webhook registration failed. Reason is included in the payload
   </td>
  </tr>
</table>

#### Sample Success Response

```json
{
  "data": {
    "company_id": 1,
    "events": [
      "sender.created",
      "task.accepted",
      "task.reassigned",
      "task.transferred",
      "task.completed",
      "task.failed",
      "driver.arrived",
      "driver.departed",
      "order.created",
      "order_item.cancelled",
      "order.transfer.rejected"
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

#### Same Failure Response

HTTP Response Code: 422

```json
{
  "data": {
    "EC401": ["url can't be blank"]
  }
}
```

### Webhook Payload Samples

When the respective event is triggered, a HTTP POST will be called to the registered Webhook URL with the following sample payloads.

### Request Body

This is the format of the HTTP Post request body your system will receive in the webhook call

<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>id
   </td>
   <td>Unique identifier for the event
   </td>
  </tr>
  <tr>
   <td>event_type
   </td>
   <td>Event type triggered
   </td>
  </tr>
  <tr>
   <td>company_slug
   </td>
   <td>Company slug the event belongs to
   </td>
  </tr>
  <tr>
   <td>webhook_id
   </td>
   <td>The id of the webhook that has subscribed to this event
   </td>
  </tr>
  <tr>
   <td>created_at
   </td>
   <td>Time at which the event was created. Measured in seconds since the Unix epoch
   </td>
  </tr>
  <tr>
   <td>data
   </td>
   <td>Object containing the data associated with the event. For a different event type the data is different. See sample payloads below for examples
   </td>
  </tr>
  <tr>
    <td>yojee_instance
    </td>
     <td>Yojee API environment
    </td>
  </tr>

</table>

#### Event: order.created

```json
{
  "company_slug": "yojee",
  "created_at": 1573616200,
  "data": {
    "cancelled_at": null,
    "completion_time": null,
    "container_no": null,
    "display_price": null,
    "external_id": null,
    "id": 3465,
    "inserted_at": "2019-11-13T03:36:19.446090Z",
    "number": "O-IWWYG6ADYLLW",
    "order_items": [
      {
        "external_customer_id": null,
        "external_customer_id2": null,
        "external_customer_id3": null,
        "id": 14411,
        "inserted_at": "2019-11-13T03:36:19.473411Z",
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
      "organisation_name": "Nikola Test Company",
      "type": "organisation"
    },
    "status": "accepted"
  },
  "event_type": "order.created",
  "id": "72574e3c-77e1-445f-805a-5c246d4dda55",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: order_item.cancelled

```json
{
  "company_slug": "yojee",
  "created_at": 1573616529,
  "data": {
    "event_time": "2019-11-13T03:40:44.802915Z",
    "external_customer_id": null,
    "external_customer_id2": null,
    "external_customer_id3": null,
    "tracking_number": "YOJ-ZWRW1ZBWF5MZ"
  },
  "event_type": "order_item.cancelled",
  "id": "1c0d2750-b75b-4831-b7d2-2c79a1ef36ce",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: task.accepted (Pickup)

```json
{
  "company_slug": "yojee",
  "created_at": 1573618207,
  "data": {
    "driver": {
      "id": 683,
      "name": "Nikola Test Driver"
    },
    "event_time": "2019-11-13T04:08:36.848982Z",
    "id": 35852,
    "inserted_at": "2019-11-13T04:02:50.850823Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "task_type": "pickup"
  },
  "event_type": "task.accepted",
  "id": "4f8d2be4-1306-465e-8651-1e0d5b86c8e6",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: task.accepted (Drop-off)

```json
{
  "company_slug": "yojee",
  "created_at": 1573618207,
  "data": {
    "driver": {
      "id": 683,
      "name": "Nikola Test Driver"
    },
    "event_time": "2019-11-13T04:08:36.848982Z",
    "id": 35851,
    "inserted_at": "2019-11-13T04:02:50.850823Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "task_type": "dropoff"
  },
  "event_type": "task.accepted",
  "id": "16f40019-424e-4a29-baa2-a7886c661921",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: driver.arrived

```json
{
  "company_slug": "yojee",
  "created_at": 1573618472,
  "data": {
    "driver": {
      "id": 683,
      "name": "Test Driver"
    },
    "event_time": "2019-11-13T04:12:47.529000Z",
    "id": 35852,
    "inserted_at": "2019-11-13T04:02:50.850823Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "task_type": "pickup"
  },
  "event_type": "driver.arrived",
  "id": "6e450fe1-973c-4056-b588-11b433095718",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: driver.departed

```json
{
  "company_slug": "yojee",
  "created_at": 1573618776,
  "data": {
    "driver": {
      "id": 683,
      "name": "Nikola Test Driver"
    },
    "event_time": "2019-11-13T04:18:11.016000Z",
    "id": 35852,
    "inserted_at": "2019-11-13T04:02:50.850823Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "task_type": "pickup"
  },
  "event_type": "driver.departed",
  "id": "f17246f9-35ab-44c6-96c8-0061cb1dcf08",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: task.completed

```json
{
  "company_slug": "yojee",
  "created_at": 1573619045,
  "data": {
    "driver": {
      "id": 683,
      "name": "Nikola Test Driver"
    },
    "event_time": "2019-11-13T04:22:18.000000Z",
    "id": 35851,
    "inserted_at": "2019-11-13T04:02:50.850823Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "pod_url": "https://umbrella-dev.yojee.com/api/v3/public/pods/order_item/YOJ-VBBS9LFIOBG9",
    "task_type": "dropoff"
  },
  "event_type": "task.completed",
  "id": "4b067931-b83b-45f4-92ff-85c908a6f417",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: task.failed

```json
{
  "company_slug": "yojee",
  "created_at": 1573619526,
  "data": {
    "driver": {
      "id": 683,
      "name": "Nikola Test Driver"
    },
    "event_time": "2019-11-13T04:31:09.000000Z",
    "id": 35854,
    "inserted_at": "2019-11-13T04:29:51.353982Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "tracking_number": "YOJ-MDMSLHECUDWP"
    },
    "task_type": "pickup"
  },
  "event_type": "task.failed",
  "id": "730d3078-7c55-4afe-a90d-890f64545a9b",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

### Delivery Headers

HTTP POST payloads that are delivered to your webhook's configured URL endpoint will contain a header.

<table>
  <tr>
   <td><strong>Header</strong>
   </td>
   <td><strong>Description</strong>
   </td>
  </tr>
  <tr>
   <td>yojee-signature
   </td>
   <td>The HMAC hex digest of the response body.
<p>
The HMAC hex digest is generated using the SHA256 hash function and the secret as the HMAC key.
   </td>
  </tr>
  <tr>
   <td>yojee-request-timestamp
   </td>
   <td>Timestamp used as input for verifying signature
   </td>
  </tr>
</table>

#### Verifying Signatures

Yojee signs the webhook events it sends to your endpoints. We do so by including a signature using a hash-based message authentication code (HMAC) with SHA-256 in each event’s **yojee-signature** header. This allows you to validate that the events were sent by Yojee, not by a third party.

<!-- theme: info -->

> #### Note
>
> Before you can verify signatures, you need to retrieve your endpoint's secret from us. Each secret is unique to the endpoint to which it corresponds.

The following are the steps to verify the signature:

1. Retrieve **yojee-signature** and **yojee-request-timestamp** from header
2. Prepare signed payload string

   - Concatenate: yojee-request-timestamp (as string), the character ‘.’, and the JSON request body.

3. Determine Expected Signature

   - Compute an HMAC with the SHA256 hash function. Use the endpoint’s signing **secret** as the key, and use the prepared signed payload string as the message.

4. Compare signatures

   - Compare the signature in the HTTP Header to the Expected Signature. If the signatures match, compute the difference between the current timestamp and the received timestamp to decide if the difference is within the tolerance of your system.

###

### Responding to a Webhook

To acknowledge receipt of a webhook, your endpoint should return a **_2xx HTTP status code_**. All response codes outside this range, including **_3xx codes_**, will indicate to Yojee that you did not receive the webhook.

We will attempt to deliver your webhooks for up to five times with an exponential back off, according to the following table:

<table>
  <tr>
   <td><strong>Retry</strong>
   </td>
   <td><strong>Seconds</strong>
   </td>
  </tr>
  <tr>
   <td>1
   </td>
   <td>120
   </td>
  </tr>
  <tr>
   <td>2
   </td>
   <td>360
   </td>
  </tr>
  <tr>
   <td>3
   </td>
   <td>840
   </td>
  </tr>
  <tr>
   <td>4
   </td>
   <td>1800
   </td>
  </tr>
  <tr>
   <td>5
   </td>
   <td>3720
   </td>
  </tr>
</table>

#### Best Practice

If your webhook script performs complex logic, or makes network calls, it's possible that the script would time out before Yojee sees its complete execution. For that reason, you might want to have your webhook endpoint immediately acknowledge receipt by returning a **_2xx HTTP status code_**, and then perform the rest of its duties.
