**Sender API Guide**

## Revision

<table style="text-align: left;">
    <tr>
        <td><strong>Version</strong></td>
        <td><strong>Date</strong></td>
        <td><strong>Change</strong></td>
    </tr>
    <tr>
        <td>Jan 2021</td>
        <td>27 Jan 2021</td>
        <td>First Version</td>
    </tr>
    <tr>
        <td>Nov 2022</td>
        <td>30 Nov 2022</td>
        <td>
        - <strong>Updated V4 endpoints:</strong> for sender order creation<br/>
        - <strong>Updated reference link to:</strong> webhook section in API Integration Introduction Guide</td>
    </tr>
    <tr>
        <td>Jun 2023</td>
        <td>27 Jun 2023</td>
        <td> Add new section on POD
    </tr>
</table>

# Overview

The **three primary** RESTful API calls related to this guide are:

1. [Sender Order Creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-sender-api-v4.yaml/paths/~1api~1v4~1sender~1orders~1create/post)
2. [Sender Order Cancellation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-sender-api.yaml/paths/~1api~1v3~1sender~1orders~1cancel/post)
3. [Sender Webhook Registration](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-webhook-api.yaml/paths/~1api~1v3~1dispatcher~1webhooks/post) (for order status update)
   - For more information on [POD](#pod), please refer to last section of this guide.

## Basic information on APIs

### Base URL

In this document we will use `[BASEURL]` to represent the base URL for the calls.

For **development and testing purposes**, please use https://umbrella-staging.yojee.com.

The base URL for the **Production API** is https://umbrella.yojee.com.

### Authentication

Most of the API calls requires the following parameters in the header:

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

The Company Slug is a string to uniquely identify each instance of a customer’s company in Yojee. Each customer is assigned to a slug which they will use as part of the authentication information.

#### Access Token

A long-lived Access Token is generated for the `Sender` account. This token will only change upon a change in the password of the Sender account.

Obtain this information from the Yojee team working with you.

In this document we will use `[SLUG]` and `[TOKEN]` to represent the `company_slug` and `access_token` respectively.

## Sender Order Creation

This API call will create an order in Yojee using Sender credentials. We identify the Sender by the Access Token provided in the Request Header parameters.

For full details, please click [here](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-sender-api-v4.yaml/paths/~1api~1v4~1sender~1orders~1create/post).

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>POST</td>
    <td>[BASEURL]/api/v4/sender/orders/create</td>
  </tr>
</table>

### Request Headers

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
    <td>Access token <strong>(Sender Credentials)<strong></td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use ‘application/json’</td>
  </tr>
</table>

### Request Body

```json
{
  "data": [
    {
      "order_info": {},
      "order_items": [{...}],
      "order_steps": [{...}],
      "order_item_steps": [{...}],
      "order_step_groups": [{...}],
      "order_voyage_info": {}
    }
  ]
}
```

<table style="text-align: left;">
    <tr>
        <td><strong>Field</strong></td>
        <td><strong>Data Type</strong></td>
        <td><strong>Required? (Y/N)</strong></td>
        <td><strong>Description</strong></td>
    </tr>
    <tr>
        <td>order_info</td>
        <td>object</td>
        <td>Y</td>
        <td>Contains main information about an order.<br/>
        - For example: order number, order external id, booking template information etc.</td>
    </tr>
    <tr>
        <td>order_items</td>
        <td>array</td>
        <td>Y</td>
        <td>A list of items that belongs to an order. Each item contains an order item information. <br/>
        - For example: description, quantity, dimension, container details etc.</td>
    </tr>
    <tr>
        <td>order_steps</td>
        <td>array</td>
        <td>Y</td>
        <td>A list of steps which contains address information and time requirements for respective tasks.<br/>
        - For example: address, pickup/dropoff time windows, contact information etc.</td>
    </tr>
    <tr>
        <td>order_item_steps</td>
        <td>array</td>
        <td>Y</td>
        <td>A list of objects that link order_items to order_steps. In this section, it indicates the step_group and step_sequence of an order.</td>
    </tr>
    <tr>
        <td>order_step_groups</td>
        <td>array</td>
        <td>Y</td>
        <td>Contains grouping information of an order.</td>
    </tr>
    <tr>
        <td>order_voyage_info</td>
        <td>object</td>
        <td>N</td>
        <td>Contains voyage information of an order.</td>
    </tr>
</table>

### Sample Request Body Payload - Single Leg Order

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "EON-12345678",
        "packing_mode": "FCL",
        "service_type_name": "Same day",
        "template_type_id": 1
      },
      "order_item_steps": [
        {
          "order_item_index": 0,
          "order_step_group_index": 0,
          "order_step_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "order_item_index": 0,
          "order_step_group_index": 0,
          "order_step_index": 1,
          "step_sequence": 1,
          "type": "dropoff"
        },
        {
          "order_item_index": 1,
          "order_step_group_index": 0,
          "order_step_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "order_item_index": 1,
          "order_step_group_index": 0,
          "order_step_index": 1,
          "step_sequence": 1,
          "type": "dropoff"
        }
      ],
      "order_items": [
        {
          "description": "Item 1",
          "external_customer_id": "ecid-1",
          "external_customer_id2": "ecid-2",
          "external_customer_id3": "ecid-3",
          "external_tracking_number": "ETN-1234567891",
          "height": 8,
          "height_unit": "foot",
          "info": "Please call before delivery",
          "item_container": {
            "container_no": "759933",
            "description": "Maersk",
            "iso_type": "45G1",
            "seal_no": "123456",
            "slot_date": "2021-01-01T13:12:17.325Z",
            "slot_reference": "1",
            "tare_weight": 3870
          },
          "length": 20,
          "length_unit": "foot",
          "payload_type": "Package",
          "quantity": 1,
          "volume": 36.24556,
          "volume_unit": "cubic_meter",
          "volumetric_weight": 37.6272,
          "volumetric_weight_unit": "metric_ton",
          "weight": 5.2,
          "weight_unit": "metric_ton",
          "width": 8,
          "width_unit": "foot"
        },
        {
          "description": "Item 2",
          "external_customer_id": "ecid-1",
          "external_customer_id2": "ecid-2",
          "external_customer_id3": "ecid-3",
          "external_tracking_number": "ETN-1234567892",
          "height": 67,
          "height_unit": "centimeter",
          "info": "Please call before delivery",
          "item_container": null,
          "length": 52,
          "length_unit": "centimeter",
          "payload_type": "Package",
          "quantity": 1,
          "volume": 188136,
          "volume_unit": "cubic_centimeter",
          "volumetric_weight": 188.136,
          "volumetric_weight_unit": "kilogram",
          "weight": 100,
          "weight_unit": "kilogram",
          "width": 54,
          "width_unit": "centimeter"
        }
      ],
      "order_steps": [
        {
          "address": "Topaz Building Kamias Road, Malaya Quezon City",
          "address2": "Suite 200",
          "city": "Malaya Quezon City",
          "contact_company": "Gadgets Co",
          "contact_email": "johndoe@example.com",
          "contact_name": "John Doe",
          "contact_phone": "12345678",
          "country": "Philippines",
          "from_time": "2022-04-19T09:38:25.566831",
          "location": {
            "lat": 14.6332747,
            "lng": 121.052446
          },
          "postal_code": "1101",
          "state": "Luzon",
          "timezone": "Asia/Manila",
          "to_time": "2022-04-19T10:08:25.566831"
        },
        {
          "address": "Taguig, 1630 Metro Manila, Philippines",
          "address2": "Entrance A",
          "city": "Manila",
          "contact_company": "LZ Warehouse",
          "contact_email": "lzwarehouse@example.com",
          "contact_name": "LZ Warehouse",
          "contact_phone": "12345678",
          "country": "Philippines",
          "from_time": "2022-04-19T10:08:25.566831",
          "location": {
            "lat": 14.5126048,
            "lng": 120.9778036
          },
          "postal_code": "1630",
          "state": "Luzon",
          "timezone": "Asia/Manila",
          "to_time": "2022-04-19T10:38:25.566831"
        }
      ],
      "order_step_groups": [
        {
          "is_direct_shipment": true,
          "packing_mode": "FCL",
          "transport_mode": ["road", "ocean"]
        }
      ]
    }
  ]
}
```

### Sample Request Body Payload - Multi-Leg Order

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "EON-12345678",
        "packing_mode": "FCL",
        "service_type_name": "Same day",
        "template_type_id": 1
      },
      "order_item_steps": [
        {
          "order_item_index": 0,
          "order_step_group_index": 0,
          "order_step_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "order_item_index": 0,
          "order_step_group_index": 0,
          "order_step_index": 1,
          "step_sequence": 1,
          "type": "dropoff"
        },
        {
          "order_item_index": 0,
          "order_step_group_index": 1,
          "order_step_index": 2,
          "step_sequence": 2,
          "type": "pickup"
        },
        {
          "order_item_index": 0,
          "order_step_group_index": 1,
          "order_step_index": 3,
          "step_sequence": 3,
          "type": "dropoff"
        },
        {
          "order_item_index": 1,
          "order_step_group_index": 0,
          "order_step_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "order_item_index": 1,
          "order_step_group_index": 0,
          "order_step_index": 1,
          "step_sequence": 1,
          "type": "dropoff"
        },
        {
          "order_item_index": 1,
          "order_step_group_index": 1,
          "order_step_index": 2,
          "step_sequence": 2,
          "type": "pickup"
        },
        {
          "order_item_index": 1,
          "order_step_group_index": 1,
          "order_step_index": 3,
          "step_sequence": 3,
          "type": "dropoff"
        }
      ],
      "order_items": [
        {
          "description": "Item 1",
          "external_customer_id": "ecid-1",
          "external_customer_id2": "ecid-2",
          "external_customer_id3": "ecid-3",
          "external_tracking_number": "ETN-1234567891",
          "height": 8,
          "height_unit": "foot",
          "info": "Please call before delivery",
          "item_container": {
            "container_no": "759933",
            "description": "Maersk",
            "iso_type": "45G1",
            "seal_no": "123456",
            "slot_date": "2021-01-01T13:12:17.325Z",
            "slot_reference": "1",
            "tare_weight": 3870
          },
          "length": 20,
          "length_unit": "foot",
          "payload_type": "Package",
          "quantity": 1,
          "volume": 36.24556,
          "volume_unit": "cubic_meter",
          "volumetric_weight": 37.6272,
          "volumetric_weight_unit": "metric_ton",
          "weight": 5.2,
          "weight_unit": "metric_ton",
          "width": 8,
          "width_unit": "foot"
        },
        {
          "description": "Item 2",
          "external_customer_id": "ecid-1",
          "external_customer_id2": "ecid-2",
          "external_customer_id3": "ecid-3",
          "external_tracking_number": "ETN-1234567892",
          "height": 67,
          "height_unit": "centimeter",
          "info": "Please call before delivery",
          "item_container": null,
          "length": 52,
          "length_unit": "centimeter",
          "payload_type": "Package",
          "quantity": 1,
          "volume": 188136,
          "volume_unit": "cubic_centimeter",
          "volumetric_weight": 188.136,
          "volumetric_weight_unit": "kilogram",
          "weight": 100,
          "weight_unit": "kilogram",
          "width": 54,
          "width_unit": "centimeter"
        }
      ],
      "order_steps": [
        {
          "address": "Topaz Building Kamias Road, Malaya Quezon City",
          "address2": "Suite 200",
          "city": "Malaya Quezon City",
          "contact_company": "Gadgets Co",
          "contact_email": "johndoe@example.com",
          "contact_name": "John Doe",
          "contact_phone": "12345678",
          "country": "Philippines",
          "from_time": "2022-04-19T09:38:25.566831",
          "location": {
            "lat": 14.6332747,
            "lng": 121.052446
          },
          "postal_code": "1101",
          "state": "Luzon",
          "timezone": "Asia/Manila",
          "to_time": "2022-04-19T10:08:25.566831"
        },
        {
          "address": "Taguig, 1630 Metro Manila, Philippines",
          "address2": "Entrance A",
          "city": "Manila",
          "contact_company": "LZ Warehouse",
          "contact_email": "lzwarehouse@example.com",
          "contact_name": "LZ Warehouse",
          "contact_phone": "12345678",
          "country": "Philippines",
          "from_time": "2022-04-19T10:08:25.566831",
          "location": {
            "lat": 14.5126048,
            "lng": 120.9778036
          },
          "postal_code": "1630",
          "state": "Luzon",
          "timezone": "Asia/Manila",
          "to_time": "2022-04-19T10:38:25.566831"
        },
        {
          "address": "Taguig, 1630 Metro Manila, Philippines",
          "address2": "Entrance A",
          "city": "Manila",
          "contact_company": "LZ Warehouse",
          "contact_email": "lzwarehouse@example.com",
          "contact_name": "LZ Warehouse",
          "contact_phone": "12345678",
          "country": "Philippines",
          "from_time": "2022-04-19T10:38:25.566831",
          "location": {
            "lat": 14.5126048,
            "lng": 120.9778036
          },
          "postal_code": "1630",
          "state": "Luzon",
          "timezone": "Asia/Manila",
          "to_time": "2022-04-19T11:38:25.566831"
        },
        {
          "address": "FinalWareHouse, 1234 Metro Manila, Philippines",
          "address2": null,
          "city": "Manila",
          "contact_company": "Final Warehouse",
          "contact_email": "finalwarehouse@example.com",
          "contact_name": "Warehouse Contact Person",
          "contact_phone": "12345678",
          "country": "Philippines",
          "from_time": "2022-04-19T11:38:25.566831",
          "postal_code": "1234",
          "state": "Luzon",
          "timezone": "Asia/Manila",
          "to_time": "2022-04-19T12:38:25.566831"
        }
      ],
      "order_step_groups": [
        {
          "is_direct_shipment": true,
          "packing_mode": "FCL",
          "transport_mode": ["road", "ocean"]
        },
        {
          "is_direct_shipment": true,
          "packing_mode": "FCL",
          "transport_mode": ["road", "ocean"]
        }
      ]
    }
  ]
}
```

### Responses

<table style="text-align: left;">
  <tr>
    <td><strong>HTTP Response Code</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>200</td>
    <td>Order creation is successful</td>
  </tr>
  <tr>
    <td>422</td>
    <td>Order creation failed. Reason is included in the payload</td>
  </tr>
</table>

#### Sample Success Response: 200

```json
{
  "data": [
    {
      "external_id": "EON-12345678",
      "id": 435523,
      "invoice_ref": "IV-KGSN5FR7PXTT",
      "number": "O-IBMHRWSUS4S4",
      "order_items": [
        {
          "external_tracking_number": "ETN-1234567891",
          "id": 461412,
          "tracking_number": "YOJ-FERK2RV8C0SW"
        },
        {
          "external_tracking_number": "ETN-1234567892",
          "id": 461413,
          "tracking_number": "YOJ-FERK2R98C0WE"
        }
      ]
    }
  ],
  "message": "1 Orders created!"
}
```

#### Sample Failure Response: 422

```json
{
  "errors": [
    {
      "code": "EC403",
      "message": "sender is invalid",
      "metadata": {
        "data_entity": "order",
        "field": "sender",
        "ref": "data[0].order_info",
        "value": null
      }
    }
  ],
  "request_id": "FyuzpAr_riwCJ1IAH2Gy"
}
```

## Sender Order Cancellation

This API call will cancel an order in Yojee using Sender credentials. We identify the Sender by the Access Token provided in the Request Header parameters.

For full details, please click [here](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-sender-api.yaml/paths/~1api~1v3~1sender~1orders~1cancel/post).

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>POST</td>
    <td>[BASEURL]/api/v3/sender/orders/cancel</td>
  </tr>
</table>
</table>

### Request Headers

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
    <td>Access token <strong>(Sender Credentials)<strong></td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use ‘application/json’</td>
  </tr>
</table>

<!-- theme:info -->

> ### Note
>
> None of the parameters here are required, but at least one is required for a successful cancellation.

### Request Body - using order number

```json
{
  "order_number": "O-IBMHRWSUS4S4"
}
```

### Request Body - using order external id

```json
{
  "order_external_id": "EON-12345678"
}
```

### Request Body - using tracking number

```json
{
  "tracking_number": "YOJ-FERK2R98C0WE"
}
```

### Request Body - using external customer id

```json
{
  "external_customer_id": "ecid-1"
}
```

### Responses

<table style="text-align: left;">
  <tr>
    <td><strong>HTTP Response Code</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>200</td>
    <td>Order cancellation is successful</td>
  </tr>
  <tr>
    <td>422</td>
    <td>Order cancellation failed. There are a number of reasons for this</td>
  </tr>
  <tr>
    <td>404</td>
    <td>Resource not found</td>
  </tr>
</table>

#### Sample Success Response: 200

If **either** `order_number` **or** `order_external_id` is supplied in the Request Body, the following is the successful response:

```json
{
  "message": "Order cancelled."
}
```

If **either** `tracking_number` **or** `external_customer_id` is supplied in the Request Body, the following is the successful response:

```json
{
  "data": {},
  "message": "Order Item(s) cancelled successfully."
}
```

#### Sample Failure Response: 422

There is a new setting `allow_sender_cancel_order` which is **defaulted to false**.

If this setting is false, the following message will be returned when attempting to cancel an order using Sender credentials.

Please get in touch with your Yojee Customer Success representative if you encounter this message and would like to set the setting to **true**.

```json
{
  "data": {
    "BK0569": ["Sender cannot cancel order"]
  }
}
```

There is another new `setting allow_sender_cancel_assigned_task` which is **defaulted to false**.

If this setting is false, the following message will be returned when attempting to cancel an order that has already been assigned.

Please get in touch with your Yojee Customer Success representative if you encounter this message and would like to set the setting to **true**.

```json
{
  "data": {
    "BK0504": ["Cannot cancel an order (or item) with started tasks"]
  }
}
```

In the case where the Sender credentials used to make the API does not match the Sender of the Order it is trying to cancel, the following message will be returned.

```json
{
  "data": {
    "BK0571": ["The order(s) are not belong to the sender"]
  }
}
```

#### Sample Failure Response: 404

The following message is returned if the data supplied does not match any Order in the system.

```json
{
  "message": "Resource not found!"
}
```

## Sender Webhooks - Order Status Update

To get a status update of an order, we need to register to Yojee webhook event(s).

- For example, when order is assigned to a driver, when order is completed and when order is cancelled etc.
  - Please refer to this list of supported events which can be found [here.](./api-integration-guide.md#events-supported)

The Sender webhook api endpoint is the same as Dispatcher webhook (Company Level). The only difference being that Webhooks registered with `sender_id` will only be triggered for Orders created by the Sender with the sender_id (Sender Level).

For more information on Yojee webhook related, please refer to `webhooks section` which can be found at [API Integration Introduction Guide.](./api-integration-guide.md#webhooks)

## POD

In this section, we will focus more on:

1. how to identify if an order is completed
2. how to retrieve a POD

### How to identify if order is completed?

When an order is marked as **completed**, a webhook event = `task.completed` will be triggered and Yojee platform will send a post request to the registered url.

Below shows a sample of webhook event = `task.completed` when order is completed.

```json
{
  "company_slug": "yojee",
  "created_at": 1679017779,
  "data": {
    "driver": {
      "id": 20496,
      "name": "Driver 1"
    },
    "eta": "2023-03-17T20:39:13.543787",
    "event_time": "2023-03-17T01:49:39.009000Z",
    "id": 17683868,
    "inserted_at": "2023-03-17T01:29:13.001319Z",
    "order": {
      "external_id": "YOJ01234567",
      "number": "O-CQDSIW0ETB1O"
    },
    "order_item": {
      "external_customer_id": "108E23",
      "external_customer_id2": "108E23C",
      "external_customer_id3": null,
      "state": "completed",
      "tracking_number": "YOJ-7LK7R3EF2GCA"
    },
    "order_item_step": {
      "metadata": { "sequence": "2" }
    },
    "pod_url": "https://umbrella-staging.yojee.com/api/v3/public/pods/order_item/YOJ-7LK7R3EF2GCA",
    "reasons": [],
    "sender": {
      "id": 20794
    },
    "step_sequence": 1,
    "task_type": "dropoff"
  },
  "event_type": "task.completed",
  "id": "b1485b39-f598-4dca-8f56-a766a741454d",
  "webhook_id": 37,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

To identify a completed order, there are three points to notice:

1. event_type = `'task.completed'`
2. data.task_type = `'dropoff'`
3. data.order_item.state = `'completed'`

### How to retrieve POD?

There are two ways to retrieve a POD.

1. [by order number](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-public-api.yaml/paths/~1api~1v3~1public~1pods~1order~1{order_number}/get)
2. [by tracking number](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-public-api.yaml/paths/~1api~1v3~1public~1pods~1order_item~1{tracking_number}/get)
