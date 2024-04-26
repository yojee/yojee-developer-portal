**API Common Error Guide**

# Overview

In this document, you could be able to find the common API error messages for different endpoints, the description of the message and steps on how to resolve those issues.

# Generic Errors

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "EC426": ["Resource not found!"]
  }
}
```

OR

```json
{
  "message": "Resource not found!"
}
```

</td>
  <td>This happens when system could not find the record based on the given identifier from the request payload.</td>
  <td>Ensure that identifier is valid.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "EC427": ["Unauthorized access!"]
  }
}
```

</td>
    <td>There are few scenarios that this error message will occur, when:</br>
      <ul>
        <li>System is unable to find record based on the given identifier.</li>
        <li>Your account do not have access to perform create/update action to certain resource. </br>
        For example, to create/update dispatcher record etc.</li>
      </ul>
    </td>
    <td>To resolve this, ensure that:</br>
      <ul>
        <li>Identifier is valid. OR</li>
        <li>If you want to perform create/update action, please contact system administrator to enable the access.</li>
      </ul>
    </td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "AC0015": ["Company does not exist."]
  }
}
```

</td>
    <td>This happens when "company_slug" value is invalid.</td>
    <td>Ensure that "company_slug" is valid or contact system administrator to get your company_slug.</td>
  </tr>
  <tr>
<td>

```json
{
  "message": "Unauthorized access!"
}
```

</td>
  <td>This happens when "access_token" value is invalid.</td>
  <td>Ensure that "access_token" is value or contact system administrator to get your access_token.</td>

  </tr>
</table>

# Other Errors

<!-- theme: info -->

> #### Note
>
> To find out more details on each endpoint, sample request payload and sample response, you can click on the individual title below :)

## [Authentication - Login](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-public-api.yaml/paths/~1api~1v3~1auth~1signin/post)

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "message": "Invalid or deactivated company"
}
```

</td>
    <td>This happens when "company_slug" value is invalid or it has been deactivated in the system.</td>
    <td>Please contact system administrator to get a new slug or reactivate your company slug.</td>
  </tr>
  <tr>
<td>

```json
{
  "message": "Unauthorized access!"
}
```

</td>
    <td>This happens when "email" and/or "password" value are/is invalid.</td>
    <td>Ensure that "email" and "password" are valid.</td>
  </tr>
  <tr>
<td>

```json
{
  "message": "Login with password is disabled"
}
```

</td>
    <td>This happens when login with password indicator is disabled in the system.</td>
    <td>Please contact system administrator to enable the indicator.</td>
  </tr>
</table>

## Order API

### [Dispatcher Order](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1orders~1create/post) and [Sender order creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-sender-api.yaml/paths/~1api~1v4~1sender~1orders~1create/post)

For order creation, the validation will vary depending on the given template id/template type id in the payload.

Let's take a look at the sample example of the common error messages that you might encounter and how to identity and resolve such errors.

<!-- theme: info -->

> #### Note
>
> If there are multiple orders in the request payload, please refer to "ref" field to identify which index of the items contain the invalid value.

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "errors": [
    {
      "code": "AC0014",
      "message": "No corporate and sender exists. please create a sender account before create a order.",
      "metadata": {
        "data_entity": "order",
        "field": "sender",
        "ref": "data[0].order_info"
      }
    }
  ],
  "request_id": "FwMaGv4SMgT2sAMAAGHB"
}
```

</td>
    <td>This happens when the given sender value is invalid in your dispatcher system.</td>
    <td>To resolve this, ensure that sender external id is valid.<br/> 
      <ul>
        <li>To get the value, navigate to Dispatcher UI, under Manage > Customers > Sender (Individual/Corporate). <br/>
        <strong>NOTE:</strong> please choose Corporate for integration purposes.
        </li>
        <li>Or contact system administrator to get the value.</li>
      </ul>
    </td>
  </tr>
  <tr>
<td>

```json
{
  "errors": [
    {
      "code": "EC424",
      "message": "order.external_id: INV350963 has already been taken",
      "metadata": {
        "field": "order.external_id",
        "value": "INV350963"
      }
    }
  ],
  "request_id": "Fv0Do0Ih1G8pH7oAANZR"
}
```

</td>
    <td>This happens when order external id is set to be unique in your dispatcher system.</td>
    <td>To resolve this, ensure that order external id value cannot be duplicated.</td>
  </tr>
  <tr>
<td>

```json
{
  "errors": [
    {
      "code": "BK0527",
      "message": "to_time End time cannot be before start time",
      "metadata": {
        "data_entity": "order_step",
        "field": "to_time",
        "ref": "data[0].order_steps[0]",
        "value": "2022-01-14T18:48:00"
      }
    }
  ],
  "request_id": "FwMap_eNfFxLIs0AAHFh"
}
```

</td>
    <td>This happens when "to_time" value is earlier than "from_time" value.</td>
    <td>To resolve this, ensure that "to_time" value should always be later than "from_time". <br/><br/>
    <strong>Note that</strong>, both "from_time" and "to_time" should be in the ISO8601 format.</td>
  </tr>
  <tr>
<td>

```json
{
  "errors": [
    {
      "code": "EC403",
      "message": "payload_type is invalid",
      "metadata": {
        "data_entity": "order_item",
        "field": "payload_type",
        "ref": "data[0].order_items[0]",
        "value": "import"
      }
    }
  ],
  "request_id": "FwMa7j-KLuf2hxsAAJyi"
}
```

</td>
    <td>This happens when payload_type = "import" is not exist in your dispatcher system.</td>
    <td>To resolve this, ensure that payload_type is valid.
      <ul>
        <li>To get the value, navigate to Dispatcher UI, under Manage > Orders > Item Types.</li>
        <li>Or contact system administrator to get the value.</li>
        <li>Or with our case, we can create a new payload_type record = "import".</li>
      </ul>
    </td>
  </tr>
  <tr>
<td>

```json
{
  "errors": [
    {
      "code": "EC403",
      "message": "service_type_name is invalid",
      "metadata": {
        "data_entity": "order",
        "field": "service_type_name",
        "ref": "data[0].order_info",
        "value": "service type test"
      }
    }
  ],
  "request_id": "FwMbWAt-0orl2lMAAJ7B"
}
```

</td>
     <td>This happens when service_type_name = "service type test" is not exist in your dispatcher system.</td>
    <td>To resolve this, ensure that service_type_name is valid.<br/> 
      <ul>
        <li>To get the value, navigate to Dispatcher UI, under Manage > Orders > Service Types.</li>
        <li>Or contact system administrator to get the value.</li>
        <li> Or with our case, we can create a new service_type_name record = "service type test".</li>
      </ul>
   </td>
  </tr>
</tr>
</table>

<!-- theme: info -->

> #### Note
>
> Validation(s) that take place in the order creation will also take place during updating of order record.

### [Order Cancellation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v4~1company~1order~1cancel/put)

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "errors": [
    {
      "code": "BK0510",
      "message": "Order not found",
      "metadata": {}
    }
  ],
  "request_id": "FwMmGZa5zAQStEwAABbS"
}
```

</td>
   <td>This happens when system could not find the order record based on the given identifier from the request payload.</td>
  <td>Ensure that identifier is valid.</td>
</table>

## Task

### [Mark as failed](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-worker-api.yaml/paths/~1api~1v3~1worker~1task~1{id}~1mark_as_failed/post), [Mark as Completed](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-worker-api.yaml/paths/~1api~1v3~1worker~1tasks~1{id}~1complete/put), [Mark Arrival](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-worker-api.yaml/paths/~1api~1v3~1worker~1tasks~1mark_arrival/post), [Mark Departure](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-worker-api.yaml/paths/~1api~1v3~1worker~1tasks~1mark_departure/post)

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "WO1022": ["The task's state cannot transit."]
  }
}
```

</td>
  <td>This happens when the current task state is already in final state (completed/invalidated/failed).</td>
  <td><strong>Note that</strong>, you can only mark task as failed or complete when the current task status is in initial state which is created.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "WO1015": ["One or more sub tasks have not been completed"]
  }
}
```

</td>
  <td>This happens when there is/are compulsory subtask(s) which linked to the current task that you want to mark task as failed and those subtasks are incomplete.</td>
  <td>To resolve this, complete those compulsory subtasks first before you can proceed to mark task as failed.</br>
  To complete subtask, you can explore this endpoint <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-task-api.yaml/paths/~1api~1v3~1dispatcher~1sub_tasks/put">Complete Subtask</a>.
  </td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "MA1305": ["Invalid geographic coordinates"]
  }
}
```

</td>
  <td>This happens when "lng" and/or "lat" value are/is invalid.</td>
  <td>Ensure that both "lng" and "lat" are valid before sending the request.</td>
  </tr>
</table>

## Task Group

### [Accept](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-worker-api.yaml/paths/~1api~1v3~1worker~1task_groups~1{id}~1accept/put), [Reject](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-worker-api.yaml/paths/~1api~1v3~1worker~1task_groups~1{id}~1reject/put)

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "WO1017": ["Task Group no longer available. Please refresh"]
  }
}
```

</td>
  <td>This happens when the Task Group record has already been accepted or broadcasted.</td>
  <td rowspan="2">Ensure that Task Group ID is valid. You can explore this endpoint <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-user-api.yaml/paths/~1api~1v3~1dispatcher~1task_groups/get">Get list of Task Groups</a>.</td>
  </tr>
    <tr>
<td>

```json
{
  "data": {
    "AC0305": ["The broadcasted taskgroup has expired."]
  }
}
```

</td>
  <td>This happens when Task Group record has already been timeout.</td>
  </tr>
</table>

## [Complete Subtask](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-task-api.yaml/paths/~1api~1v3~1dispatcher~1sub_tasks/put)

<table style="table-layout: fixed; width: 100%">
  <tr>
    <td><strong>Error Message</strong></td>
    <td><strong>Explanation</strong></td>
    <td><strong>How to resolve?</strong></td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": ["the sub task does not exist"]
  }
}
```

</td>
  <td>This happens when "task_id" and/or "sub_task_rule_id" are/is invalid.</td>
  <td>
    <ul>
      <li>To ensure that "task_id" is valid, please check this endpoint <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-task-api.yaml/paths/~1api~1v3~1dispatcher~1tasks/get">Search Tasks</a>.</li>
      <li>To ensure that "sub_task_rule_id" is valid, please check this endpoint <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-task-api.yaml/paths/~1api~1v3~1dispatcher~1sub_task_rules/get">Get list of Sub Task Rules</a>.</li>
  <ul>
  </td>
  </tr>
    <tr>
<td>

```json
{
  "data": {
    "WO1020": [
      "Arrival time is required for capture arrival time subtask completion"
    ]
  }
}
```

</td>
  <td>This happens when "arrival_time" is empty or invalid.</td>
  <td><strong>Note that</strong>, "arrival_time" is a required field and it should be in the ISO8601 format.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "WO1019": ["Photo is required for subtask completion"]
  }
}
```

</td>
    <td>This happens when "photos" is an empty array.</td>
    <td><strong>Note that</strong>, "photos" is a required field and it cannot be an empty array.</td>
  </tr>
</table>
