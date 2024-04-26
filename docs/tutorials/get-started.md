**Get Started with Yojee APIs**

# Overview

In most use cases, the first thing when integrating with Yojee APIs is to flow orders into Yojee. To do this we will need the following steps:

1. Obtain the necessary authentication credentials
1. Make a call to create an order in Yojee

## Authentication

The Yojee API uses API Keys to authenticate requests in most cases. Most of these calls will require the credentials for a `Dispatcher` role. This is the same role as the user that logs into the `Dispatcher Portal` to perform most of the operations in Yojee through GUI.

### API Keys

To authenticate using API Keys, include the following parameters in the HTTP header.

```json json_schema
{
  "type": "object",
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

| HTTP Header  | Type   | Description                                       |
| ------------ | ------ | ------------------------------------------------- |
| company_slug | string | Company Slug                                      |
| access_token | string | Access Token for the user profile making the call |
| Content-Type | string | application/json                                  |

## Create an Order

We will be making the call to [**Create Order**](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1orders~1create/post) to create an order.

### Sample Payload

Use the following sample payload for your first `Create Order` call:

```json
{
  "data": [
    {
      "order_info": {
        "external_id": "EON-12345678",
        "packing_mode": "FCL",
        "sender": {
          "external_id": "SID-1136"
        },
        "service_type_name": "Same day",
        "template_type_id": 1
      },
      "order_items": [
        {
          "description": "Item 1",
          "payload_type": "Package",
          "quantity": 1,
          "external_customer_id": "ecid-1",
          "external_customer_id2": "ecid-2",
          "external_customer_id3": "ecid-3",
          "info": "Please call before delivery",
          "height": 8,
          "height_unit": "foot",
          "length": 20,
          "length_unit": "foot",
          "width": 8,
          "width_unit": "foot",
          "weight": 5.2,
          "weight_unit": "metric_ton",
          "volume": 36.24556,
          "volume_unit": "cubic_meter",
          "volumetric_weight": 37.6272,
          "volumetric_weight_unit": "metric_ton"
        },
        {
          "description": "Item 2",
          "payload_type": "Container",
          "quantity": 1,
          "external_customer_id": "ecid-1",
          "external_customer_id2": "ecid-2",
          "external_customer_id3": "ecid-3",
          "info": "Please call before delivery",
          "height": 67,
          "height_unit": "centimeter",
          "length": 52,
          "length_unit": "centimeter",
          "width": 54,
          "width_unit": "centimeter",
          "weight": 100,
          "weight_unit": "kilogram",
          "volume": 188136,
          "volume_unit": "cubic_centimeter",
          "volumetric_weight": 188.136,
          "volumetric_weight_unit": "kilogram",
          "item_container": {
            "container_no": "759933",
            "description": "my container",
            "iso_type": "45G1",
            "seal_no": "123456",
            "slot_date": "2021-01-01T13:12:17.325Z",
            "slot_reference": "1",
            "weight": 3870,
            "vgm": 3870,
            "vgm_unit": "kilogram",
            "vgm_date": "2021-01-01T13:12:17.325Z",
            "vgm_party": "shipper"
          }
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
      ]
    }
  ]
}
```

<!-- theme: info -->

> ### sender.external_id, template_type_id
>
> Note that both `sender.external_id` and `template_type_id` need to be a valid ID in your Yojee instance

### Making the call

There are a few ways you can make the call:

#### On the API Reference

- Locate the `Try-it Console` on the right of the [**Create Order**](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1orders~1create/post).
- Fill in your `company_slug` and `access_token`.
- Copy and paste the above sample payload in the **Body** section of the `Try-it Console`.
- Click `Send Request`.

#### On the command line

- Copy this command:

```shell
curl --location 'https://umbrella-staging.yojee.com/api/v4/company/orders/create' \
--header 'company_slug: [COMPANY_SLUG]' \
--header 'access_token: [ACCESS_TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
    "data": [
        {
            "order_info": {
                "external_id": "EON-12345678",
                "packing_mode": "FCL",
                "sender": {
                    "external_id": "SID-1136"
                },
                "service_type_name": "Same day",
                "template_type_id": 1
            },
            "order_items": [
                {
                    "description": "Item 1",
                    "payload_type": "Package",
                    "quantity": 1,
                    "external_customer_id": "ecid-1",
                    "external_customer_id2": "ecid-2",
                    "external_customer_id3": "ecid-3",
                    "info": "Please call before delivery",
                    "height": 8,
                    "height_unit": "foot",
                    "length": 20,
                    "length_unit": "foot",
                    "width": 8,
                    "width_unit": "foot",
                    "weight": 5.2,
                    "weight_unit": "metric_ton",
                    "volume": 36.24556,
                    "volume_unit": "cubic_meter",
                    "volumetric_weight": 37.6272,
                    "volumetric_weight_unit": "metric_ton"
                },
                {
                    "description": "Item 2",
                    "payload_type": "Container",
                    "quantity": 1,
                    "external_customer_id": "ecid-1",
                    "external_customer_id2": "ecid-2",
                    "external_customer_id3": "ecid-3",
                    "info": "Please call before delivery",
                    "height": 67,
                    "height_unit": "centimeter",
                    "length": 52,
                    "length_unit": "centimeter",
                    "width": 54,
                    "width_unit": "centimeter",
                    "weight": 100,
                    "weight_unit": "kilogram",
                    "volume": 188136,
                    "volume_unit": "cubic_centimeter",
                    "volumetric_weight": 188.136,
                    "volumetric_weight_unit": "kilogram",
                    "item_container": {
                        "container_no": "759933",
                        "description": "my container",
                        "iso_type": "45G1",
                        "seal_no": "123456",
                        "slot_date": "2021-01-01T13:12:17.325Z",
                        "slot_reference": "1",
                        "weight": 3870,
                        "vgm": 3870,
                        "vgm_unit": "kilogram",
                        "vgm_date": "2021-01-01T13:12:17.325Z",
                        "vgm_party": "shipper"
                    }
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
            ]
        }
    ]
}'
```

<!-- theme: info -->

> Remember to replace **[COMPANY_SLUG]** and **[ACCESS_TOKEN]** with your `company_slug` and `access_token`

## Troubleshooting

You may encounter the following messages when you make your first requests.

### Unauthorized

```json
{
  "message": "Unauthorized"
}
```

This means either your `company_slug` or your `access_token` is invalid. Check them and resend the request. If the problem persists, kindly check with the Yojee team that is working with you.

## Verify Order is created

### Success Response

If the API call is successful, you should receive a HTTP 200 response with a payload that looks like the following sample:

```json
{
  "data": [
    {
      "id": 3289092,
      "number": "O-ZKZ3AJNAFFXU",
      "external_id": "EON-12345678",
      "order_items": [
        {
          "id": 4414173,
          "tracking_number": "YOJ-IB2NJPEOVGLF",
          "external_tracking_number": null
        },
        {
          "id": 4414174,
          "tracking_number": "YOJ-IB2NJPFSEWRW",
          "external_tracking_number": null
        }
      ],
      "service_type_id": 5384,
      "invoice_ref": "IV-57SS4MRVCI26"
    }
  ],
  "message": "1 Orders created!"
}
```

### Checking in Portal

You can also login to the Yojee `Dispatcher Portal` with your `Dispatcher` credentials to check that the order has been created.
