**V3 to V4 Order Creation Mapping Guide**

# Overview

In V4 order api, we introduced `data` parameter to allow creation of multiple orders in a single payload. On top of that, with `template_id`, user can now customize the booking template(s) to cater to different order.

<!-- theme: warning -->

> #### Note
>
> In V4, validation of order creation/updating will vary depending on the given template_id.

Therefore, in this document, we will focus on the mapping of order payload from [V3 order creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v3.yaml/paths/~1api~1v3~1idspatcher~1orders_multi_leg/post) to [V4 order creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1orders~1create/post).

<table style="text-align: left;">
    <tr>
        <td></td>
        <td><strong>Method</strong></td>
        <td><strong>Endpoint</strong></td>
    </tr>
    <tr>
        <td><strong>V3</strong></td>
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
        <td colspan="4" style="text-align: center;"><strong>V4</strong></td>
    </tr>
    <tr>
        <td><strong>Field</strong></td>
        <td><strong>Data Type</strong></td>
        <td><strong>Required? (Y/N)</strong></td>
        <td><strong>Payload Schema</strong></td>
    </tr>
    <tr>
        <td>order_info</td>
        <td>object</td>
        <td>Y</td>
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
    </tr>
    <tr>
        <td>order_steps</td>
        <td>array</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>order_item_steps</td>
        <td>array</td>
        <td>Y</td>
    </tr>
    <tr>
        <td>order_step_groups</td>
        <td>array</td>
        <td>Y</td>
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

Let's first take a look sample payload of both versions before we move into field mapping table in the next section.

<table>
    <tr>
        <td colspan="2" style="text-align: center;"><strong>Sample Payload</strong></td>
    </tr>
    <tr style="text-align: center;">
        <td><strong>V3</strong></td>
        <td><strong>V4</strong></td>
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
          "contact_phone": 62010000,
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
          "contact_phone": 62010001,
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
          "quantity": 4,
          "order_item_index": 0,
          "order_step_index": 0,
          "order_step_group_index": 0,
          "step_sequence": 0,
          "type": "pickup"
        },
        {
          "quantity": 4,
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

There are more available parameters for container related information. Please refer to [V4 - Order Creation Doc](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1orders~1create/post).

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

Please refer to [V4 - Order Creation Doc](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1orders~1create/post) for more information.

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
          "contact_phone": 62010000,
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
        <td>*</td>
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
        <td>*</td>
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
          "quantity": 4, //moved from v3 "steps"
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

<strong>Note:</strong>

<ul>
    <li>In V3, "quantity" param is inside "steps" object, however, in V4, this param is moved to "order_item_steps" object.</li>
</ul>

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
> For those parameters that marked with '\*', we can customize whether those are required or optional field in the booking template.

## List of available options

<!-- theme: info -->

> #### Note
>
> Refer to table below for list of available options for new parameters.
> Sample options are also available at [V4 - Order Creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api-v4.yaml/paths/~1api~1v4~1company~1orders~1create/post)

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
        <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-public-api.yaml/paths/~1api~1v4~1public~1templates~1active/get">Get active templates</a>. To know which item is a default template, its "default_at" will not be null.
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
        <td>To get service type name, refer to this endpoint <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-configuration-api.yaml/paths/~1api~1v3~1dispatcher~1service_types/get">Get Service Types</a>
        </td>
    </tr>
</table>
