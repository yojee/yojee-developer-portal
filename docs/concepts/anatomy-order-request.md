# Anatomy of An Order Request
## How to structure the JSON payload to create an order

<!-- theme: success -->

> 💡 For a quick background on the terminology used in this document, check out [Yojee Terminology](./yojee-terminology.md)

To create an order via the Yojee API, we make a call to
[**Create Order**](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders/post) with a JSON payload that adheres to the following schema.

This document will detail the fields and nested fields required in the JSON payload.

```json json_schema
title: 'Create Order Request'
required:
  - item_steps
  - items
  - sender_type
  - steps
type: object
properties:
  external_id:
    type: string
    description: For entering any tracking number from partner system
  external_sender_id:
    type: string
    description: External Sender ID. Either this or `Sender ID` should be provided
  sender_id:
    type: integer
    description: Sender ID. Either this or `External Sender ID` should be provided
  sender_type:
    type: string
    description: Sender Type. For most integration this is `organisation`
    enum:
      - organisation
      - individual
  items:
    type: array
    description: List of items to be sent
    items:
      type: object
      properties:
        description:
          type: string
        width:
          type: number
        length:
          type: number
        height:
          type: number
        weight:
          type: number
        quantity:
          type: number
        info:
          type: string
        external_customer_id:
          type: string
          description: For storing external reference
        external_customer_id2:
          type: string
          description: For storing external reference
        external_customer_id3:
          type: string
          description: For storing external reference
        payload_type:
          type: string
          description: Item Type defined in Yojee Dispatcher
        price_info:
          type: string
        service_type:
          type: string
          description: Service Type defined in Yojee Dispatcher
        service_type_id:
          type: integer
          description: Service Type ID of the Service Type for this order. Not needed if `service_type` is provided
        volume:
          type: number
        volumetric_weight:
          type: number
        price_amount:
          type: number
      required:
        - payload_type
        - service_type
  steps:
    type: array
    description: List of delivery details for pick up and drop off steps
    items:
      type: object
      properties:
        quantity:
          type: number
        address:
          type: string
        address2:
          type: string
        country:
          type: string
        state:
          type: string
        postal_code:
          type: string
        contact_company:
          type: string
        contact_name:
          type: string
        contact_phone:
          type: number
          description: The phone number if SMS notification is required. The corresponding notification setting also needs to be enabled in Yojee Dispatcher
        contact_email:
          type: string
          description: The email address if email notification is required. The corresponding notification setting also needs to be enabled in Yojee Dispatcher
        from_time:
          type: string
          description: Start of time slot for this step in ISO8601 format
        to_time:
          type: string
          description: End of time slot for this step in ISO8601 format
        lat:
          type: number
          description: If lat/lng for the location is not provided, Yojee will attempt to geocode using the address
        lng:
          type: number
          description: If lat/lng for the location is not provided, Yojee will attempt to geocode using the address
      required:
        - address
        - country
        - from_time
        - to_time
  item_steps:
    type: array
    description: List of item steps
    items:
      type: object
      properties:
        item_id:
          type: number
        order_step_id:
          type: number
        step_group:
          type: number
        step_sequence:
          type: number
        type:
          type: string
          description: This can be `pickup` or `dropoff`
      required:
        - item_id
        - order_step_id
        - type
  price_currency:
    type: string
    description: Order currency
  price_amount:
    type: number
    description: Order price
  placed_by_user_profile_id:
    type: string
    description: Only in cases where there are Corporate Users created under Corporate Sender
  container_no:
    type: string
    description: Used when order is tied to a container
```

## First Level Required parameters

### 

## First Level Optional parameters

## Single-Leg Order
In the most simple case, an `Order` will have a single leg as follows:

```mermaid
  graph LR
    A[Source] -->|Leg 1| B[Destination]
```

