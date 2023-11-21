**API Integration Introduction Guide**

## Revision

<table style="text-align: left;">
    <tr>
        <td><strong>Version</strong></td>
        <td><strong>Date</strong></td>
        <td><strong>Change</strong></td>
    </tr>
    <tr>
        <td>May 2021</td>
        <td>18 May 2021</td>
        <td>First Version</td>
    </tr>
    <tr>
        <td>Nov 2022</td>
        <td>28 Nov 2022</td>
        <td>
        - <strong>Updated V4 endpoints:</strong> for order creation and order cancellation<br/>
        - <strong>Updated webhook sample event payload</strong></td>
    </tr>
    <tr>
        <td>Nov 2022</td>
        <td>30 Nov 2022</td>
        <td><strong>Updated webhook details:</strong> specify the Sender with the sender_id to register this webhook for<br/> 
    </tr>
</table>

# Overview

The **three primary** RESTful API calls related to this project are:

1. [Order Creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1orders~1create/post)

2. [Order Cancellation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1order~1cancel/put)

3. [Webhook Registration](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-webhook-api.yaml/paths/~1api~1v3~1dispatcher~1webhooks/post)

## Basic information on APIs

### Base URL

In this document we will use `[BASEURL]` to represent the base URL for the calls.

For **development and testing purposes**, please use https://umbrella-staging.yojee.com.

The base URL for the **Production API** is https://umbrella.yojee.com.

### Authentication

Most of the API calls will require the following parameters in the header:

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

The Company Slug is a string to uniquely identify each instance of a customer’s company in Yojee. Each customer is assigned a slug which they will use as part of the authentication information.

#### Access Token

A long-lived Access Token is generated for the `Dispatcher` account. This token will only change upon a change in the password of the Dispatcher account.

Obtain this information from the Yojee team working with you.

In this document we will use `[SLUG]` and `[TOKEN]` to represent the `company_slug` and `access_token` respectively.

## Order Creation

This API call will create an order in Yojee.

For full details, please click [here](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1orders~1create/post).

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>POST</td>
    <td>[BASEURL]/api/v4/company/orders/create</td>
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
    <td>Access token</td>
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
      "order_step_groups": [{...}]
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
        For example: order number, order external id, sender information etc.</td>
    </tr>
    <tr>
        <td>order_items</td>
        <td>array</td>
        <td>Y</td>
        <td>A list of items that belongs to an order. Each item contains an order item information. <br/>
        For example: description, quantity, dimension, container details etc.</td>
    </tr>
    <tr>
        <td>order_steps</td>
        <td>array</td>
        <td>Y</td>
        <td>A list of steps which will contain information and time requirements for respective tasks.<br/>
        For example: address, pickup/dropoff time windows, contact information.</td>
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
</table>

### Sample Request Body Payload - Single Leg Order

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "EON-12345678",
        "packing_mode": "FCL",
        "sender": {
          "external_id": "SID-1136",
          "id": 123
        },
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
        "sender": {
          "external_id": "SID-1136",
          "id": 123
        },
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

## Order Cancellation

This API call will cancel an order in Yojee.

For full details, please click [here](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1order~1cancel/put).

<table style="text-align: left;">
  <tr>
    <td><strong>Method</strong></td>
    <td><strong>Endpoint</strong></td>
  </tr>
  <tr>
    <td>PUT</td>
    <td>[BASEURL]/api/v4/company/order/cancel</td>
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
    <td>Access token</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use ‘application/json’</td>
  </tr>
</table>

### Request Body - using order number

```json
{
  "number": "O-IBMHRWSUS4S4",
  "force_cancellation": true
}
```

### Request Body - using order external id

```json
{
  "external_id": "EON-12345678",
  "force_cancellation": true
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
    <td>Order cancellation failed. Reason is included in the payload</td>
  </tr>
</table>

#### Sample Success Response: 200

```json
{
  "data": {
    "cancelled_at": "2022-11-28T08:59:39.206221Z",
    "external_id": "TB00000087",
    "number": "O-IBMHRWSUS4S4"
  }
}
```

#### Sample Failure Response: 422

```json
{
  "errors": [
    {
      "code": "BK0510",
      "message": "Order not found",
      "metadata": {}
    }
  ],
  "request_id": "Fyu0BG6uReUqWvUAJ4Mh"
}
```

## Webhooks

Webhook URLs can be registered with Yojee and the Yojee system will make HTTP calls to the URL with the relevant payload when a status update of the Order is triggered.

<!-- theme:info -->

> ### Note
>
> Webhooks registered with `sender_id` will only be triggered for Orders created by the Sender with the sender_id (Sender Level).

### Events Supported

Events currently being supported are:

<table style="text-align: left;">
  <tr>
    <td rowspan="2"><strong>Event Name</strong></td>
    <td rowspan="2"><strong>Description</strong></td>
    <td colspan="2" style="text-align: center;"><strong>Level</strong></td>
  </tr>
  <tr>
    <td><strong>Company</strong></td>
    <td><strong>Sender</strong></td>
  </tr>
  <tr>
    <td><a href="#event-sendercreated">sender.created</a></td>
    <td>When a Sender is created</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">N</td>
  </tr>
  <tr>
    <td><a href="#event-ordercreated">order.created</a></td>
    <td>When an Order is created</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-orderupdated">order.updated</a></td>
    <td>When an Order is updated</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-taskaccepted-pickup">task.accepted</a></td>
    <td>When a task (assigned to a driver - pickup / drop-off) is accepted</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-taskcompleted">task.completed</a></td>
    <td>When a task (assigned to a driver - pickup / drop-off) is completed</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-taskfailed">task.failed</a></td>
    <td>When a task (assigned to a driver - pickup / drop-off) is reported</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-taskreassigned">task.reassigned</a></td>
    <td>When a task (assigned to a driver & not accepted yet) is reassigned</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-tasktransferred">task.transferred</a></td>
    <td>When a task is transferred to an upstream / downstream partner</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">N</td>
  </tr>
  <tr>
    <td><a href="#event-order_itemcancelled">order_item.cancelled</a></td>
    <td>When an Order Item is cancelled</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-driverarrived">driver.arrived</a></td>
    <td>When a driver has arrived at the pickup / drop-off destination point</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-driverdeparted">driver.departed</a></td>
    <td>When a driver has departed from the pickup / drop-off destination point</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">Y</td>
  </tr>
  <tr>
    <td><a href="#event-ordertransferrejected">order.transfer.rejected</a></td>
    <td>When a transferred Order is rejected by the downstream partner</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">N</td>
  </tr>
  <tr>
    <td><a href="#event-paymentcompleted">payment.completed</a></td>
    <td>When a Payment is completed</td>
    <td style="text-align: center;">Y</td>
    <td style="text-align: center;">N</td>
  </tr>
</table>

To register a webhook, use the following API endpoint:

For full details, please click [here](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-webhook-api.yaml/paths/~1api~1v3~1dispatcher~1webhooks/post).

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
    <td>Access token <strong>(Dispatcher Credentials)</strong></td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>Y</td>
    <td>Use ‘application/json’</td>
  </tr>
</table>

### Request Body

<!-- theme: info -->

> #### Note
>
> You can send as a form data or send as JSON in the POST request body

#### Send as form data

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
    <td>events[]</td>
    <td>Y</td>
    <td>The event to register the webhook with. For multiple events, repeat with another events[] parameter</td>
  </tr>
  <tr>
    <td>sender_id</td>
    <td>N</td>
    <td>Specify the Sender with the sender_id to register this webhook for</td>
  </tr>
</table>

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
--form 'events[]="order.updated"' \
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
    "order.updated",
    "order_item.cancelled"
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
      "sender.created",
      "task.accepted",
      "task.reassigned",
      "task.transferred",
      "task.completed",
      "task.failed",
      "driver.arrived",
      "driver.departed",
      "order.created",
      "order.updated",
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

### Request Body

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
    <td>Yojee API environment</td>
  </tr>

</table>

#### Event: sender.created

```json
{
  "company_slug": "theary-test",
  "created_at": 1663039247,
  "data": {
    "billing_address": "International financial center",
    "business_reg_no": "98sdf89j23",
    "email": "testing@gmail.com",
    "external_id": "IDTEST1",
    "gst_no": "0923i4s0du3",
    "id": 4084,
    "inserted_at": "2022-09-13T03:20:47.718944",
    "name": "testing",
    "organisation": {
      "city": null,
      "country": null,
      "default_dropoff_address": null,
      "default_pickup_address": null,
      "name": "Testing booking",
      "phone": "+627128787217",
      "postal_code": null,
      "reg_address": "International financial center",
      "sender_organisation_slug": "testing-booking-ORG-1233"
    },
    "payment_option": "monthly_billing",
    "phone": "+6281238372",
    "sender_organisation_slug": "testing-booking-ORG-1233",
    "sender_type": "organisation",
    "updated_at": "2022-09-13T03:20:47.718944"
  },
  "event_type": "sender.created",
  "id": "5fdd1d4a-f103-4a6a-afad-c69563c53940",
  "version": "2",
  "webhook_id": 130,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

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
  "version": "2",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: order.updated

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
  "event_type": "order.updated",
  "id": "72574e3c-77e1-445f-805a-5c246d4dda55",
  "version": "2",
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
    "eta": "2023-11-21T09:09:00.000000",
    "event_time": "2023-11-21T09:28:44.957776Z",
    "id": 14299954,
    "inserted_at": "2023-11-21T08:49:29.149877Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "state": "processing",
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "sender": {"id": 15399},
    "step_sequence": 1,
    "task_type": "pickup"
  },
  "event_type": "task.accepted",
  "id": "4f8d2be4-1306-465e-8651-1e0d5b86c8e6",
  "version": "2",
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
    "eta": "2023-11-21T09:29:01.000000",
    "event_time": "2023-11-21T09:28:44.957776Z",
    "id": 14299955,
    "inserted_at": "2023-11-21T08:49:29.149877Z",
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "state": "processing",
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "sender": {"id": 15399},
    "step_sequence": 1,
    "task_type": "dropoff"
  },
  "event_type": "task.accepted",
  "id": "16f40019-424e-4a29-baa2-a7886c661921",
  "version": "2",
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
    "order": {"external_id": null, "number": "O-ELIDYWA9X0ED"},
    "order_item": {
      "external_customer_id": null,
      "external_customer_id2": null,
      "external_customer_id3": null,
      "state": "completed",
      "tracking_number": "YOJ-VBBS9LFIOBG9"
    },
    "sender": {"id": 15399},
    "step_sequence": 1,
    "pod_url": "https://umbrella-dev.yojee.com/api/v3/public/pods/order_item/YOJ-VBBS9LFIOBG9",
    "task_type": "dropoff"
  },
  "event_type": "task.completed",
  "id": "4b067931-b83b-45f4-92ff-85c908a6f417",
  "version": "2",
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
      "state": "processing",
      "sender": {"id": 15399},
      "step_sequence": 0,
      "tracking_number": "YOJ-MDMSLHECUDWP"
    },
    "reasons": ["package damage"],
    "task_type": "pickup"
  },
  "event_type": "task.failed",
  "id": "730d3078-7c55-4afe-a90d-890f64545a9b",
  "version": "2",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: task.reassigned

```json
{
  "company_slug": "yojee",
  "created_at": 1662702819,
  "data": {
    "driver": { "id": 5102, "name": "Driver test" },
    "eta": null,
    "event_time": "2022-09-09T05:53:39.416353Z",
    "id": 1933847,
    "inserted_at": "2022-09-09T04:20:59.232506Z",
    "order": { "external_id": null, "number": "O-MQVIDX44ATAX" },
    "order_item": {
      "external_customer_id": "YOJ-1234",
      "external_customer_id2": null,
      "external_customer_id3": null,
      "state": "created",
      "tracking_number": "YOJ-GGQCJRAVE0EZ"
    },
    "order_item_step": { "metadata": {} },
    "reasons": [],
    "sender": { "id": 1792 },
    "step_sequence": 0,
    "task_type": "pickup"
  },
  "event_type": "task.reassigned",
  "id": "b92bd595-b626-4181-8ceb-0a5bb4ff3f9d",
  "version": "2",
  "webhook_id": 116,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

#### Event: task.transferred

```json
{
  "company_slug": "yojee",
  "created_at": 1582704492,
  "data": [
    {
      "event_time": "2020-02-26T08:08:11.174241Z",
      "id": 42185,
      "inserted_at": "2020-02-26T08:07:01.025972Z",
      "order_item": {
        "external_customer_id": null,
        "external_customer_id2": null,
        "external_customer_id3": null,
        "tracking_number": "YOJ-X6L7COIZ5IFC"
      },
      "task_type": "dropoff"
    },
    {
      "event_time": "2020-02-26T08:08:11.171353Z",
      "id": 42186,
      "inserted_at": "2020-02-26T08:07:01.025972Z",
      "order_item": {
        "external_customer_id": null,
        "external_customer_id2": null,
        "external_customer_id3": null,
        "tracking_number": "YOJ-X6L7COIZ5IFC"
      },
      "task_type": "pickup"
    }
  ],
  "event_type": "task.transferred",
  "id": "9601c6c8-9b17-442a-be6f-307399f7533a",
  "version": "2",
  "webhook_id": 103,
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
    "order": {
      "container_no": null,
      "external_id": "testing",
      "id": 7665,
      "number": "O-SGAODV1ZMTRC",
      "status": "cancelled"
    },
    "sender": {
      "id": 24
    },
    "tracking_number": "YOJ-ZWRW1ZBWF5MZ"
  },
  "event_type": "order_item.cancelled",
  "id": "1c0d2750-b75b-4831-b7d2-2c79a1ef36ce",
  "version": "2",
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
  "version": "2",
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
  "version": "2",
  "webhook_id": 96,
  "yojee_instance": "https://umbrella-dev.yojee.com"
}
```

#### Event: order.transfer.rejected

```json
{
  "company_slug": "theary-test",
  "created_at": 1646126351,
  "data": {
    "company_id": 872,
    "downstream_company_id": 1137,
    "rejected_order": {
      "cancelled_at": "2022-03-01T09:19:11.477130Z",
      "completion_time": null,
      "container_no": null,
      "display_price": "SGD 0",
      "external_id": "LGL20220216031_FCL Export",
      "id": 664564,
      "inserted_at": "2022-02-24T08:09:19.065521Z",
      "number": "O-PO5RZVH1X6XV",
      "order_items": [
        {
          "external_customer_id": null,
          "external_customer_id2": "COSCO",
          "external_customer_id3": null,
          "id": 924019,
          "inserted_at": "2022-02-24T08:09:19.068975Z",
          "item": {
            "description": null,
            "global_tracking_number": "Y-0IYPWYWXSNZI",
            "height": null,
            "id": 923527,
            "length": null,
            "payload_type": "40 HIGH",
            "quantity": 1,
            "volume": "0.000",
            "volumetric_weight": "0.00",
            "weight": null,
            "width": null
          },
          "price": null,
          "service_type": "fcl_export",
          "status": "cancelled",
          "tracking_number": "YOJ-FHHKYT3WQBHW",
          "transfer_info": null
        }
      ],
      "price": { "amount": "0.00000000", "currency": "SGD" },
      "sender": {
        "id": 4012,
        "name": null,
        "organisation_name": "Test",
        "type": "organisation"
      },
      "status": "cancelled"
    },
    "upstream_company_id": 872
  },
  "event_type": "order.transfer.rejected",
  "id": "3814906a-9c62-4937-83d9-f337add13e5e",
  "version": "2",
  "webhook_id": 130,
  "yojee_instance": "https://umbrella-staging.yojee.com"
}
```

#### Event: payment.completed

```json
{
  "company_slug": "yojee",
  "created_at": 1642652192,
  "data": {
    "order": {
      "cancelled_at": null,
      "completion_time": null,
      "container_no": null,
      "display_price": "SGD 19",
      "external_id": null,
      "id": 662942,
      "inserted_at": "2022-01-20T04:16:30.949838Z",
      "number": "O-L3BIUN0W3W28",
      "order_items": [
        {
          "external_customer_id": null,
          "external_customer_id2": null,
          "external_customer_id3": null,
          "id": 919999,
          "inserted_at": "2022-01-20T04:16:30.972115Z",
          "item": {
            "description": null,
            "global_tracking_number": "Y-7TJ2HGTVGTWZ",
            "height": "2",
            "id": 919746,
            "length": "2",
            "payload_type": "Document",
            "quantity": 1,
            "volume": "8",
            "volumetric_weight": "1",
            "weight": "1",
            "width": "2"
          },
          "price": { "amount": "19.00000000", "currency": "SGD" },
          "service_type": "same_day",
          "status": "created",
          "tracking_number": "YOJ-P1RYF8RMLNXC",
          "transfer_info": null
        }
      ],
      "price": { "amount": "19.00000000", "currency": "SGD" },
      "sender": {
        "id": 3389,
        "name": null,
        "organisation_name": null,
        "type": "individual"
      },
      "status": "accepted"
    },
    "payment_data": {
      "shipping": null,
      "id": "ch_3KJs4ZJaYXslwhTn0hBYCTWQ",
      "transfer_data": null,
      "statement_descriptor_suffix": null,
      "application_fee_amount": null,
      "disputed": false,
      "transfer_group": null,
      "status": "succeeded",
      "source_transfer": null,
      "destination": null,
      "dispute": null,
      "created": 1642652191,
      "currency": "sgd",
      "refunded": false,
      "amount_refunded": 0,
      "captured": true,
      "source": {
        "address_city": "",
        "address_country": "",
        "address_line1": "44",
        "address_line1_check": "pass",
        "address_line2": "",
        "address_state": null,
        "address_zip": "109704",
        "address_zip_check": "pass",
        "brand": "Visa",
        "country": "US",
        "customer": null,
        "cvc_check": "pass",
        "dynamic_last4": null,
        "exp_month": 4,
        "exp_year": 2022,
        "fingerprint": "fcirVwe4pLmPVMRF",
        "funding": "credit",
        "id": "card_1KJs4YJaYXslwhTnfKeIio2q",
        "last4": "4242",
        "metadata": {},
        "name": "Name",
        "object": "card",
        "tokenization_method": null
      },
      "billing_details": {
        "address": {
          "city": "",
          "country": "",
          "line1": "44",
          "line2": "",
          "postal_code": "109704",
          "state": null
        },
        "email": null,
        "name": "Name",
        "phone": null
      },
      "order": null,
      "amount_captured": 1900,
      "object": "charge",
      "failure_code": null,
      "receipt_number": null,
      "receipt_email": null,
      "application": null,
      "balance_transaction": "txn_3KJs4ZJaYXslwhTn0BwvAD8N",
      "statement_descriptor": null,
      "payment_method_details": {
        "card": {
          "brand": "visa",
          "checks": {
            "address_line1_check": "pass",
            "address_postal_code_check": "pass",
            "cvc_check": "pass"
          },
          "country": "US",
          "exp_month": 4,
          "exp_year": 2022,
          "fingerprint": "fcirVwe4pLmPVMRF",
          "funding": "credit",
          "installments": null,
          "last4": "4242",
          "network": "visa",
          "three_d_secure": null,
          "wallet": null
        },
        "type": "card"
      },
      "invoice": null,
      "outcome": {
        "network_status": "approved_by_network",
        "reason": null,
        "risk_level": "normal",
        "risk_score": 49,
        "seller_message": "Payment complete.",
        "type": "authorized"
      },
      "amount": 1900,
      "fraud_details": {},
      "customer": null,
      "on_behalf_of": null,
      "refunds": {
        "data": [],
        "has_more": false,
        "object": "list",
        "total_count": 0,
        "url": "/v1/charges/ch_3KJs4ZJaYXslwhTn0hBYCTWQ/refunds"
      },
      "payment_intent": null,
      "review": null,
      "failure_message": null,
      "application_fee": null,
      "paid": true,
      "description": "Payment for O-L3BIUN0W3W28",
      "metadata": {},
      "calculated_statement_descriptor": "Stripe",
      "livemode": false,
      "payment_method": "card_1KJs4YJaYXslwhTnfKeIio2q",
      "receipt_url": "https://test.com"
    }
  },
  "event_type": "payment.completed",
  "id": "f1b5da54-f75d-40b9-9647-2a4fdf7dc835",
  "version": "2",
  "webhook_id": 75,
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

Yojee signs the webhook events it sends to your endpoints. We do so by including a signature using a hash-based message authentication code (HMAC) with SHA-256 in each event’s **yojee-signature** header. This allows you to validate that the events were sent by Yojee, not by a third party.

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
     Use the endpoint’s signing **secret** as the key, and use the prepared signed payload string as the message.

4. Compare signatures
   - Compare the signature in the HTTP Header to the Expected Signature. If the signatures match, compute the difference between the current timestamp and the received timestamp to decide if the difference is within the tolerance of your system.

### Responding to a Webhook

To acknowledge receipt of a webhook, your endpoint should return a **_2xx HTTP status code_**. All response codes outside this range, including **_3xx codes_**, will indicate to Yojee that you did not receive the webhook.

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

If your webhook script performs complex logic, or makes network calls, it's possible that the script would time out before Yojee sees its complete execution. For that reason, you might want to have your webhook endpoint immediately acknowledge receipt by returning a **_2xx HTTP status code_**, and then perform the rest of its duties.
