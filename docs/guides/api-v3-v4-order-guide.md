**V3 to V4 Order Creation Mapping Guide**

# Overview

In V4 order api, we introduced `data` parameter to allow creation of multiple orders in a single payload.
On top of that, with `template_id` or `template_type_id`, user can now customize the booking template(s) to cater to different order.

<!-- theme: warning -->

> #### Note
>
> In V4, validation of order creation/updating will vary depending on the given **template_id** or **template_type_id**.
> Note that, we can use either one of these fields to create/update an order, but one of it must be provided in the payload.
>
> Refer to "Sample payload conversion" section to see how to convert to v4 payload and "Booking Template" section to find out more additional information on booking template.

Therefore, in this document, we will focus on the mapping of order payload from V3 single-leg order creation and V3 multi-leg order creation to V4 order creation.

<table style="text-align: left;">
    <tr>
        <td></td>
        <td><strong>Method</strong></td>
        <td><strong>Endpoint</strong></td>
    </tr>
    <tr>
        <td rowspan="2" style="vertical-align: center;"><strong>V3</strong></td>
        <td>POST</td>
        <td>[BASEURL]/api/v3/dispatcher/orders</td>
    </tr>
    <tr>
        <td>POST</td>
        <td>[BASEURL]/api/v3/dispatcher/orders_multi_leg</td>
    </tr>
    <tr>
        <td><strong>V4</strong></td>
        <td>POST</td>
        <td>[BASEURL]/api/v4/company/orders/create</td>
    </tr>
</table>

## Main Components

The table below shows the main components in both versions.

<table>
    <tr>
        <td colspan="4" style="text-align: center;"><strong>V3</strong></td>
    </tr>
    <tr>
        <td><strong>Field</strong></td>
        <td><strong>Data Type</strong></td>
        <td><strong>Required? (Y/N)</strong></td>
        <td><strong>Payload Schema</strong></td>
    </tr>
    <tr>
        <td>external_id</td>
        <td>string</td>
        <td>N</td>
<td rowspan="11" style="vertical-align: top;">

```json
{
    "external_id": "TEST-ORDER-001",
    "external_sender_id": "KCYCorp",
    "sender_id": 123,
    "sender_type": "organisation",
    "placed_by_user_profile_id": "123",
    "price_currency": "SGD",
    "price_amount": 100,
    "container_no": null,
    "items": [{...}],
    "steps": [{...}],
    "item_steps": [{...}]
}
```

</td>
    </tr>
    <tr>
        <td>external_sender_id</td>
        <td>string</td>
        <td>N</td>
    </tr>
    <tr>
        <td>sender_id</td>
        <td>integer</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>sender_type</td>
        <td>string</td>
        <td>N</td>
    </tr>
     <tr>
        <td>placed_by_user_profile_id</td>
        <td>string</td>
        <td>N</td>
    </tr>
    <tr>
        <td>price_currency</td>
        <td>string</td>
        <td>N</td>
    </tr>
    <tr>
        <td>price_amount</td>
        <td>float</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>container_no</td>
        <td>string</td>
        <td>N</td>
    </tr>
    <tr>
        <td>items</td>
        <td>array</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>steps</td>
        <td>array</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>item_steps</td>
        <td>array</td>
        <td>Y</td>
    </tr>
</table>

<table>
    <tr>
        <td colspan="5" style="text-align: center;"><strong>V4</strong></td>
    </tr>
    <tr>
        <td><strong>Field</strong></td>
        <td><strong>Data Type</strong></td>
        <td><strong>Required? (Y/N)</strong></td>
        <td><strong>Description</strong></td>
        <td><strong>Payload Schema</strong></td>
    </tr>
    <tr>
        <td>order_info</td>
        <td>object</td>
        <td>Y</td>
        <td>Contains main information about an order.<br/>
        For example: order number, order external id, sender information etc.</td>
<td rowspan="5" style="vertical-align: top;">

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

</td>
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
        <td>N</td>
        <td>Contains grouping information of an order.</td>
    </tr>
</table>

<!-- theme: info -->

> #### Note
>
> In V4, order creation components will be wrapped as a single object (one order) inside <strong>"data"</strong> array.
>
> Therefore, to create multiple orders in the same payload, append the object inside the data array.

For example:

```json
{
  "data": [
    {
      "order_info": {},
      "order_items": [{...}],
      "order_steps": [{...}],
      "order_item_steps": [{...}],
      "order_step_groups": [{...}]
    },
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

## Sample payload conversion

Let's first take a look at sample payload of both versions before we move into field mapping table in the next section.

<table>
    <tr>
        <td colspan="2" style="text-align: center;"><strong>Sample Payload</strong></td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;"><strong>V3</strong></td>
    </tr>
    <tr>
        <td style="text-align: center;"><strong>Single-leg</strong></td>
        <td style="text-align: center;"><strong>Multi-leg</strong></td>
    </tr>
    <tr style="vertical-align: top;">
<td>

```json
{
  "external_id": "TEST-ORDER-001",
  "external_sender_id": "KCYCorp",
  "sender_id": 123,
  "sender_type": "organisation",
  "placed_by_user_profile_id": "123",
  "price_currency": "SGD",
  "price_amount": 100,
  "container_no": "CONTAINER-TEST",
  "items": [
    {
      "description": "Laptop Computer",
      "width": 0.11,
      "length": 0.12,
      "height": 0.044,
      "weight": 334,
      "volume": 1000,
      "volumetric_weight": 1,
      "quantity": 4,
      "info": "Item Info",
      "external_customer_id": "TN-001",
      "external_customer_id2": "CUSTOMER-INFO-001",
      "external_customer_id3": "",
      "payload_type": "Package",
      "service_type": "express",
      "price_info": "",
      "price_amount": 0
    }
  ],
  "steps": [
    {
      "quantity": 4,
      "address": "20 Pasir Pajang Road",
      "address2": "",
      "city": "",
      "country": "Singapore",
      "state": "",
      "postal_code": "117439",
      "contact_company": "S Company",
      "contact_name": "John Lim",
      "contact_phone": 62010000,
      "contact_email": "john@company1.example.com",
      "from_time": "2021-08-01T08:59:22.813Z",
      "to_time": "2021-08-02T07:59:59.813Z",
      "lat": 14.6332747,
      "lng": 121.052446
    },
    {
      "quantity": 4,
      "address": "1 Changi Business Park",
      "address2": "Avenue 1",
      "city": "",
      "country": "Singapore",
      "state": "",
      "postal_code": "486058",
      "contact_company": "C Company",
      "contact_name": "Peter Tan",
      "contact_phone": 62010001,
      "contact_email": "peter@company2.example.com",
      "from_time": "2021-08-03T08:59:22.813Z",
      "to_time": "2021-08-04T07:59:59.813Z",
      "lat": 14.5126048,
      "lng": 120.9778036
    }
  ],
  "item_steps": [
    {
      "item_id": 0,
      "order_step_id": 0,
      "type": "pickup"
    },
    {
      "item_id": 0,
      "order_step_id": 1,
      "type": "dropoff"
    }
  ]
}
```

</td>
<td>

```json
{
  "external_id": "TEST-ORDER-001",
  "external_sender_id": "KCYCorp",
  "sender_id": 123,
  "sender_type": "organisation",
  "placed_by_user_profile_id": "123",
  "price_currency": "SGD",
  "price_amount": 100,
  "container_no": "CONTAINER-TEST",
  "items": [
    {
      "description": "Laptop Computer",
      "width": 0.11,
      "length": 0.12,
      "height": 0.044,
      "weight": 334,
      "volume": 1000,
      "volumetric_weight": 1,
      "quantity": 4,
      "info": "Item Info",
      "external_customer_id": "TN-001",
      "external_customer_id2": "CUSTOMER-INFO-001",
      "external_customer_id3": "",
      "payload_type": "Package",
      "service_type": "express",
      "price_info": "",
      "price_amount": 0
    }
  ],
  "steps": [
    {
      "quantity": 4,
      "address": "20 Pasir Pajang Road",
      "address2": "",
      "city": "",
      "country": "Singapore",
      "state": "",
      "postal_code": "117439",
      "contact_company": "S Company",
      "contact_name": "John Lim",
      "contact_phone": "62010000",
      "contact_email": "john@company1.example.com",
      "from_time": "2021-08-01T08:59:22.813Z",
      "to_time": "2021-08-02T07:59:59.813Z",
      "lat": 14.6332747,
      "lng": 121.052446
    },
    {
      "quantity": 4,
      "address": "1 Changi Business Park",
      "address2": "Avenue 1",
      "city": "",
      "country": "Singapore",
      "state": "",
      "postal_code": "486058",
      "contact_company": "C Company",
      "contact_name": "Peter Tan",
      "contact_phone": "62010001",
      "contact_email": "peter@company2.example.com",
      "from_time": "2021-08-03T08:59:22.813Z",
      "to_time": "2021-08-04T07:59:59.813Z",
      "lat": 14.5126048,
      "lng": 120.9778036
    }
  ],
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
  ]
}
```

</td>
    </tr>
</table>

<table>
    <tr>
        <td colspan="2" style="text-align: center;"><strong>Sample Payload</strong></td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;"><strong>V4</strong></td>
    </tr>
    <tr>
        <td style="text-align: center;"><strong>Using template_id</strong></td>
        <td style="text-align: center;"><strong>Using template_type_id</strong></td>
    </tr>
    <tr style="vertical-align: top;">
<td>

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "TEST-ORDER-001",
        "sender": {
          "external_id": "KCYCorp",
          "id": 123,
          "sender_type": "organisation",
          "user_profile_id": 123
        },
        "service_type_name": "express",
        "template_id": 1 //required field
      },
      "order_items": [
        {
          "description": "Laptop Computer",
          "width": 0.11,
          "length": 0.12,
          "height": 0.044,
          "weight": 334,
          "volume": 1000,
          "volumetric_weight": 1,
          "quantity": 4,
          "info": "Item Info",
          "external_customer_id": "TN-001",
          "external_customer_id2": "CUSTOMER-INFO-001",
          "external_customer_id3": "",
          "payload_type": "Package",
          "item_container": {
            "container_no": "CONTAINER-TEST"
          }
        }
      ],
      "order_steps": [
        {
          "address": "20 Pasir Pajang Road",
          "address2": "",
          "city": "",
          "country": "Singapore",
          "state": "",
          "postal_code": "117439",
          "contact_company": "S Company",
          "contact_name": "John Lim",
          "contact_phone": "62010000",
          "contact_email": "john@company1.example.com",
          "from_time": "2021-08-01T08:59:22.813Z",
          "to_time": "2021-08-02T07:59:59.813Z",
          "location": {
            "lat": 14.6332747,
            "lng": 121.052446
          }
        },
        {
          "address": "1 Changi Business Park",
          "address2": "Avenue 1",
          "city": "",
          "country": "Singapore",
          "state": "",
          "postal_code": "486058",
          "contact_company": "C Company",
          "contact_name": "Peter Tan",
          "contact_phone": "62010001",
          "contact_email": "peter@company2.example.com",
          "from_time": "2021-08-03T08:59:22.813Z",
          "to_time": "2021-08-04T07:59:59.813Z",
          "location": {
            "lat": 14.5126048,
            "lng": 120.9778036
          }
        }
      ],
      "order_item_steps": [
        {
          "order_item_index": 0,
          "order_step_index": 0,
          "order_step_group_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "order_item_index": 0,
          "order_step_index": 1,
          "order_step_group_index": 0,
          "step_sequence": 1,
          "type": "dropoff"
        }
      ],
      "order_step_groups": [{}] //required field
    }
  ]
}
```

</td>
<td>

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "TEST-ORDER-001",
        "sender": {
          "external_id": "KCYCorp",
          "id": 123,
          "sender_type": "organisation",
          "user_profile_id": 123
        },
        "service_type_name": "express",
        "template_type_id": 123 //required field
      },
      "order_items": [
        {
          "description": "Laptop Computer",
          "width": 0.11,
          "length": 0.12,
          "height": 0.044,
          "weight": 334,
          "volume": 1000,
          "volumetric_weight": 1,
          "quantity": 4,
          "info": "Item Info",
          "external_customer_id": "TN-001",
          "external_customer_id2": "CUSTOMER-INFO-001",
          "external_customer_id3": "",
          "payload_type": "Package",
          "item_container": {
            "container_no": "CONTAINER-TEST"
          }
        }
      ],
      "order_steps": [
        {
          "address": "20 Pasir Pajang Road",
          "address2": "",
          "city": "",
          "country": "Singapore",
          "state": "",
          "postal_code": "117439",
          "contact_company": "S Company",
          "contact_name": "John Lim",
          "contact_phone": "62010000",
          "contact_email": "john@company1.example.com",
          "from_time": "2021-08-01T08:59:22.813Z",
          "to_time": "2021-08-02T07:59:59.813Z",
          "location": {
            "lat": 14.6332747,
            "lng": 121.052446
          }
        },
        {
          "address": "1 Changi Business Park",
          "address2": "Avenue 1",
          "city": "",
          "country": "Singapore",
          "state": "",
          "postal_code": "486058",
          "contact_company": "C Company",
          "contact_name": "Peter Tan",
          "contact_phone": "62010001",
          "contact_email": "peter@company2.example.com",
          "from_time": "2021-08-03T08:59:22.813Z",
          "to_time": "2021-08-04T07:59:59.813Z",
          "location": {
            "lat": 14.5126048,
            "lng": 120.9778036
          }
        }
      ],
      "order_item_steps": [
        {
          "order_item_index": 0,
          "order_step_index": 0,
          "order_step_group_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "order_item_index": 0,
          "order_step_index": 1,
          "order_step_group_index": 0,
          "step_sequence": 1,
          "type": "dropoff"
        }
      ],
      "order_step_groups": [{}] //required field
    }
  ]
}
```

</td>
    </tr>
</table>

### Note

- The 2 main difference in V3 single-leg and V3 multi-leg are:

<table>
    <tr>
        <td></td>
        <td><strong>Single-leg</strong></td>
        <td><strong>Multi-leg</strong></td>
    </tr>
    <tr>
        <td>contact_phone</td>
        <td>Data type = "number"</td>
        <td>Data type = "string"</td>
    </tr>
    <tr>
        <td>step_group</td>
        <td rowspan="2">non-mandatory field</td>
        <td rowspan="2">mandatory field</td>
    </tr>
     <tr>
        <td>step_sequence</td>
    </tr>

</table>

## Fields Mapping Table

<table>
    <tr style="text-align: left;">
        <td><strong>V3 fields</strong></td>
        <td><strong>V4 fields</strong></td>
        <td><strong>Required? (Y/N/*)</strong></td>
        <td><strong>Description</strong></td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">external_id</td>
        <td>N</td>
<td>It is a nested parameter inside <strong>"order_info"</strong> object. <br/>
For example:

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "TEST-ORDER-001"
      }
    }
  ]
}
```

</td>
    </tr>
    <tr>
        <td>external_sender_id</td>
        <td>external_id</td>
        <td>N</td>
<td rowspan="4">It is a nested parameter inside <strong>"sender"</strong> object and "sender" is nested inside "order_info" object. <br/>
For example:

```json
{
  "data": [
    {
      "order_info": {
        "sender": {
          "external_id": "KCYCorp",
          "id": 123,
          "sender_type": "organisation",
          "user_profile_id": 123
        }
      }
    }
  ]
}
```

<strong>Note that</strong>, "sender" is a required field.

</td>
    </tr>
    <tr>
        <td>sender_id</td>
        <td>id</td>
        <td>Y</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">sender_type</td>
        <td>N</td>
    </tr>
    <tr>
        <td>placed_by_user_profile_id</td>
        <td>user_profile_id</td>
        <td>N</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">container_no</td>
        <td>*</td>
<td>It is nested parameter inside <strong>"item_container"</strong> object and "item_container" is nested inside "order_items" array.<br/>
For example:

```json
{
  "data": [
    {
      "order_items": [
        {
          "item_container": {
            "container_no": "CONTAINER-TEST"
          }
        }
      ]
    }
  ]
}
```

There are more available parameters for container related information. Please refer to [V4 - Order Creation Doc](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1orders~1create/post).

</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">description</td>
        <td>*</td>
<td rowspan="13">It is a nested parameter inside <strong>"order_items"</strong> array.<br/>
For example:

```json
{
  "data": [
    {
      "order_items": [
        {
          "description": "Laptop Computer",
          "width": 0.11,
          "length": 0.12,
          "height": 0.044,
          "weight": 334,
          "volume": 1000,
          "volumetric_weight": 1,
          "quantity": 4,
          "info": "Item Info",
          "external_customer_id": "TN-001",
          "external_customer_id2": "CUSTOMER-INFO-001",
          "external_customer_id3": "",
          "payload_type": "Package"
        }
      ]
    }
  ]
}
```

<strong>Note:</strong>

<ul>
    <li>In V4, we introduced new fields to store UOM of each variables, for example length_unit, weight_unit etc. </li>
    <li>If those fields are not found in the payload, system will use the default setup value.</li>

Please refer to [V4 - Order Creation Doc](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1orders~1create/post) for more information.

</ul>

</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">width</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">length</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">height</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">weight</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">volume</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">volumetric_weight</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">quantity</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">info</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">external_customer_id</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">external_customer_id2</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">external_customer_id3</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">payload_type</td>
        <td>*</td>
    </tr>
    <tr>
        <td>service_type</td>
        <td>service_type_name</td>
        <td>Y</td>
<td>It is a nested parameter inside <strong>"order_info"</strong> object.<br/>
For example:

```json
{
  "data": [
    {
      "order_info": {
        "service_type_name": "express"
      }
    }
  ]
}
```

</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">address</td>
        <td>Y</td>
<td rowspan="11">It is a nested parameter in <strong>"order_steps"</strong> array.<br/>
For example:

```json
{
  "data": [
    {
      "order_steps": [
        {
          "address": "20 Pasir Pajang Road",
          "address2": "",
          "city": "",
          "country": "Singapore",
          "state": "",
          "postal_code": "117439",
          "contact_company": "S Company",
          "contact_name": "John Lim",
          "contact_phone": "62010000",
          "contact_email": "john@company1.example.com",
          "from_time": "2021-08-01T08:59:22.813Z",
          "to_time": "2021-08-02T07:59:59.813Z"
        }
      ]
    }
  ]
}
```

</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">address2</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">country</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">state</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">postal_code</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">contact_company</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">contact_name</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">contact_phone</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">contact_email</td>
        <td>*</td>
    </tr>
    <tr>
        <td colspan="2" style="text-align: center;">from_time</td>
        <td>Y</td>
    </tr>
     <tr>
        <td colspan="2" style="text-align: center;">to_time</td>
        <td>Y</td>
    </tr>
     <tr>
        <td colspan="2" style="text-align: center;">lat</td>
        <td>N</td>
<td rowspan="2">It is a nested parameter insider <strong>"location"</strong> object and "location" is nested insider "order_steps" array. <br/>
For example:

```json
{
  "data": [
    {
      "order_steps": [
        {
          "location": {
            "lat": 14.6332747,
            "lng": 121.052446
          }
        }
      ]
    }
  ]
}
```

</td>
    </tr>
     <tr>
        <td colspan="2" style="text-align: center;">lng</td>
        <td>N</td>
    </tr>
     <tr>
        <td>item_id</td>
        <td>order_item_index</td>
        <td>Y</td>
<td rowspan="5">It is a nested parameter inside <strong>"order_item_steps"</strong> array.<br/>
For example:

```json
{
  "data": [
    {
      "order_item_steps": [
        {
          "order_item_index": 0,
          "order_step_index": 0,
          "order_step_group_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        }
      ]
    }
  ]
}
```

</td>
    </tr>
     <tr>
        <td>order_step_id</td>
        <td>order_step_index</td>
        <td>Y</td>
    </tr>
     <tr>
        <td>step_group</td>
        <td>order_step_group_index</td>
        <td>Y</td>
    </tr>
     <tr>
        <td colspan="2" style="text-align: center;">step_sequence</td>
        <td>Y</td>
    </tr>
     <tr>
        <td colspan="2" style="text-align: center;">type</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>service_type_id</td>
        <td colspan="2" style="text-align: center;">N/A</td>
        <td>This parameter is not necessary anymore as its information will be retrieved based on the "service_type_name".</td>
    </tr>
    <tr>
        <td>price_info</td>
        <td colspan="2" style="text-align: center;">N/A</td>
        <td rowspan="3">These parameters are not required as those information will be retrieved and calculated based on the charge rate.</td>
    </tr>
    <tr>
        <td>price_currency</td>
        <td colspan="2" style="text-align: center;">N/A</td>
    </tr>
    <tr>
        <td>price_amount</td>
        <td colspan="2" style="text-align: center;">N/A</td>
    </tr>
</table>

<!-- theme: info -->

> #### Note
>
> For those parameters that marked with '\*', it can be configured in the booking template, whether it should be a required or an optional field.

## Thing to take notes from v3 to v4 for an order

<table style="text-align: left;">
    <tr>
        <td><strong>Field name</strong></td>
        <td><strong>V3 <br/> old booking template</strong></td>
        <td><strong>V4 <br/> new booking template</strong></td>
        <td><strong>Remarks</strong></td>
    </tr>
    <tr>
        <td>step_group</td>
        <td>1</td>
        <td>0</td>
        <td rowspan="2">In the new booking template, the value will start from its index. </td>
    </tr>
    <tr>
        <td>step_sequence</td>
        <td>1</td>
        <td>0</td>
    </tr>
</table>

**Another thing to take note:**
In **v3**, specifically for **multi-leg orders**, all tasks (task 1, task 2, task 3, task 4) in an order are **grouped under one single group**. However, in **v4**, we **split the grouping into two groups**; where the first group consists of task 1 and task 2 and second group consists of task 3 and task 4.
With this grouping, users will have the flexibility to either assign each group to different drivers or all groups to the same driver.

Let’s take a look at the small portion of structure below:

<table style="text-align: left;">
    <tr>
        <td><strong>V3</strong></td>
        <td><strong>V4</strong></td>
        <td><strong>Remarks</strong></td>
    </tr>
    <tr>
<td>

```json
{
   "type": "pickup",
   "step_group": 1,
   "step_group_size": 1,
   "step_sequence": 1
},
{
   "type": "dropoff",
   "step_group": 1,
   "step_group_size": 1,
   "step_sequence": 2,

},
{
   "type": "pickup",
   "step_group": 1,
   "step_group_size": 1,
   "step_sequence": 3,

},
{
   "type": "dropoff",
   "step_group": 1,
   "step_group_size": 1,
   "step_sequence": 4,
}
```

</td>
<td>

```json
{
   "type": "pickup",
   "step_group": 0,
   "step_group_size": 2,
   "step_sequence": 0
},
{
   "type": "dropoff",
   "step_group": 0,
   "step_group_size": 2,
   "step_sequence": 1,

},
{
   "type": "pickup",
   "step_group": 1,
   "step_group_size": 2,
   "step_sequence": 2,

},
{
   "type": "dropoff",
   "step_group": 1,
   "step_group_size": 2,
   "step_sequence": 3,
}
```

</td>
        <td>Notice that:<br/>

- step_sequence in v3 is starting from its index+1, where in v4, it is starting from its index.

- In v3, there is only one group where step_group = 1. However, in v4, there are 2 groups:
  - Leg 1 (task 1 and task 2) belongs to step_group = 0
  - Leg 2 (task 3 and task 4) belongs to step_group = 1
  </td>
      </tr>
  </table>

## List of available options

<!-- theme: info -->

> #### Note
>
> Refer to table below for list of available options for new parameters.
> Sample options are also available at [V4 - Order Creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1orders~1create/post)

<table>
    <tr style="text-align: left;">
        <td><strong>Name</strong></td>
        <td><strong>Description</strong></td>
    </tr>
    <tr>
        <td>packing_mode</td>
        <td> List of available packing modes:
            <ul>
                <li>FTL</li>
                <li>LTL</li>
                <li>CNT</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>template_id</td>
        <td>To get template_id, refer to this endpoint
        <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-public-api.yaml/paths/~1api~1v4~1public~1templates~1active/get">Get active templates</a>. <br/>
        To know which item is a default template, its "default_at" will not be null. <br/>
        <strong>NOTE:</strong> Please call the above mentioned endpoint to get the latest default template_id as this id will be updated when we make any changes to the template.
        </td>
    </tr>
    <tr>
        <td>volume_unit</td>
        <td>List of available volume UOM:
            <ul>
                <li>cubic_centimeter</li>
                <li>cubic_meter</li>
                <li>cubic_inch</li>
                <li>cubic_foot</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>volumetric_weight_unit</td>
        <td>List of available volumetric weight UOM:
            <ul>
                <li>kilogram</li>
                <li>pound</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>weight_unit</td>
        <td>List of available weight UOM: 
            <ul>
                <li>gram</li>
                <li>kilogram</li>
                <li>metric_ton</li>
                <li>ounce</li>
                <li>pound</li>
                <li>ton</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>length_unit</td>
        <td rowspan="3">List of available length, width, height UOM:
            <ul>
                <li>millimeter</li>
                <li>centimeter</li>
                <li>meter</li>
                <li>inch</li>
                <li>foot</li>
                <li>yard</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>width_unit</td>
    </tr>
    <tr>
        <td>height_unit</td>
    </tr>
    <tr>
        <td>transport_mode</td>
        <td>List of available transport modes:
            <ul>
                <li>road</li>
                <li>rail</li>
                <li>ocean</li>
                <li>air</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td>service_type_name</td>
        <td>service type name
        </td>
    </tr>
</table>

## Booking Template

There are few things to take note while customising your booking template.
Under `Template Properties`, there are 2 checkboxes:

<table>
  <tr>
    <td><strong>Name</strong></td>
    <td><strong>Description</strong></td>
  </tr>
  <tr>
    <td>"Allow missing info"</td>
    <td>If this checkbox is:<br/><br/>
      <ul>
        <li><strong>"checked":</strong> it means that system will allow order to be created even though any of the required field(s) is not present in the payload. The order record will just display with <strong>"missing info"</strong> status.<br/>
        <span style="text-decoration: underline;">For example:</span> In template ABC, we configured "description" and "postal_code" to be a required fields and "Allow missing info" is checked. Therefore, even if "description" or "postal_code" is not found in the payload, system will allow this order to be created successfully and attach this order record with "missing info" status.
        <br/><br/>
        </li>
        <li><strong>"unchecked":</strong> it means that order can only be created successfully if and only if ALL required fields are present in the payload.</li>
      </ul>
      <strong>Note:</strong> System will reject the order if "address" value is invalid. However, if "Allow missing info" is checked, system will accept the order and attach it with "missing info" status.
  </tr>
  <tr>
    <td>"Allow dispatcher to add more legs"</td>
    <td>if this checkbox is:<br/><br/>
      <ul>
        <li><strong>"checked":</strong> it means that system will allow order to be created with more legs.
        <br/>
        <br/>
        </li>
        <li><strong>"unchecked":</strong> it means that system will reject the order, if in the request payload, there are more legs than the given number of legs that defined in the template.</li>
      </ul>
      <strong>Note:</strong> This checkbox is NOT APPLICABLE for batch uploading. This means that for batch uploading, the number of legs must be the same as the number of legs that is defined in the template.
    </td>
  </tr>
</table>
