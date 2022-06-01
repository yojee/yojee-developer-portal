**API Common Error Guide**

# Overview

In this document, you can find the common API error messages for different endpoints, the description of the message and steps on how to resolve those issues.

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

## [Multi-leg](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders_multi_leg/post) and [Single-leg order creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders/post)

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
    "AC0014": [
      "No corporate and sender exists. please create a sender account before create a order."
    ]
  }
}
```

</td>
    <td>This happens when "sender_id" value is invalid.</td>
    <td>Ensure that "sender_id" exists in our Dispatcher System. You can either create a new sender via Dispatcher UI (Manage > Senders) or via our API <a href="https://yojee.stoplight.io/docs/yojee-api/publish/yojee-user-api.yaml/paths/~1api~1v3~1dispatcher~1senders/post">Create Sender.</a>
    </td>
  </tr>
</table>

## [Multi-leg order creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders_multi_leg/post)

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
    "00000": ["there are time conflicts in the input"]
  }
}
```

</td>
    <td>
        This happens when "to_time" value is earlier than "from_time".
    </td>
    <td>
        Ensure that both "from_time" and "to_time" value are valid and "to_time" value should always be later than "from_time" value. <br/><br/>
        <strong>Note that</strong>, both "from_time" and "to_time" should be in the ISO8601 format.
    </td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": ["item array is empty"]
  }
}
```

OR

```json
{
  "data": {
    "00000": ["steps array is empty"]
  }
}
```

</td>
    <td>This happens when "items" or "steps" is empty.</td>
    <td>Ensure that this parameter should NOT be an empty array in the request payload.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": ["from_time or to_time is not valid"]
  }
}
```

</td>
    <td>This happens when "from_time" or "to_time" value is not in the ISO8601 format.</td>
    <td>Ensure that both "from_time" and "to_time" value are in the correct format of ISO8601 format.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": [
      "item_steps is referring the items or steps which doesn't exist or steps or items has extra entires that item_steps is not referring to"
    ]
  }
}
```

</td>
    <td>
    This happens when the total count of "items", "steps" and "item_steps" do not tally. </br></br>
    There are few scenarios that this error message will occur: </br>
      <ul>
        <li>"item_steps" is empty when both "items" and "steps" are not empty.</li>
        <li>"items" or "steps" has extra entries that "item_steps" is not referring to.</li>
      </ul>
    </td>
    <td>
    Ensure that "items", "steps", "item_steps" total count should be tally. </br></br>
    For example:
      <ul>
        <li>If total count of "items" = 2 and total count of "steps" = 2. </br>
        In "item_steps" object, the distinct count of "item_id" and "order_step_id" should also be equal to 2.
        </li>
      </ul>
    </td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": ["step_group or step_sequence is not valid"]
  }
}
```

</td>
    <td>There are few scenarios that this error message will occur, when: <br/>
      <ul>
        <li>"step_group" and "step_sequence" are not found in the payload.</li>
        <li>"step_group" or "step_sequence" start with index = 0.</li>
      </ul>
    </td>
    <td>To resolve different scenarios that we've mentioned, ensure that:
      <ul>
        <li>Both "step_group" and "step_sequence" are present in the "item_steps" object.</li>
        <li>"step_group" or "step_sequence" should start from 1 (index + 1).</li>
      </ul>
    </td>
  </tr>
</table>

## Order cancellation - via [order number](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders~1cancel/put) or [other parameters](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders~1cancel/post)

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
    "BK0569": ["Sender cannot cancel order"]
  }
}
```

</td>
    <td>This happens when an access for sender role to cancel an order is disabled.</td>
    <td>Contact your system administrator to enable "allow_sender_cancel_order" indicator.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0504": ["Cannot cancel an order (or item) with started tasks"]
  }
}
```

</td>
    <td>This happens when task is already assigned to worker/driver.</td>
    <td>To resolve this, please cancel (mark as failed) the assigned task. Once task status changes to "unassigned", you can proceed to cancel the order.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0545": ["Cannot cancel a pending or transferred order"]
  }
}
```

</td>
    <td>This happens when order is in transferred status to your downstream partner.</td>
    <td>To resolve this, downstream partner needs to first decline/reject that transferred order. Once task status changes to "unassigned", you can proceed to cancel the order.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0510": ["Order not found"]
  }
}
```

</td>
    <td>This happens when system could not find the order record based on the given identifier value.</td>
    <td>Ensure that the parameter value (order number/order external id) is valid.</td>
  </tr>
</table>

## [Multi-leg](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1order_items~1{id}~1update_multi/put) and [Single-leg order item update](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1order_items~1{id}/put)

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
    "BK0511": ["Cannot edit order item"]
  }
}
```

</td>
    <td>There are few scenarios that this error message will occur, when:</br>
      <ul>
        <li>order item status is already in "completed" or "cancelled".</li>
        <li>order item information is already broadcasted to worker/driver.</li>
        <li>order item is transferred from upstream partner and "allow_downstream_edit_transferred_items" is disabled.</li>
      </ul>
    </td>
    <td>For upstream and downstream partner scenario, to edit/update a transferred order item, please contact system administrator to enable the "allow_downstream_edit_transferred_items" indicator.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0547": [
      "Unable to change service type. Please cancel this order and create new order for a new service type to apply."
    ]
  }
}
```

</td>
    <td>This happens when "service_type" is invalid.</td>
    <td>To resolve this, please cancel this order and create a new order.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0517": ["Dropoff time cannot be before pickup time"]
  }
}
```

</td>
    <td>This happens when the "dropoff from_time" value is earlier than the "pickup to_time" value.</td>
    <td>Ensure that "dropoff from_time" should be later than "pickup to_time".</td>
  </tr>
    <tr>
<td>

```json
{
  "data": {
    "BK0511": ["Cannot edit order item"]
  }
}
```

</td>
    <td>This happens when there was a failed task during the execution of order item update.</td>
    <td>Please retry your request and if error still persists, please contact system administrator to retrieve error log file to identify the root cause.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": ["trying to delete or update invalid tasks"]
  }
}
```

</td>
  <td>This happens when "update_steps" and/or "delete_steps" IDs are invalid.</td>
  <td>Ensure that those IDs exists in the current order steps before you can use those IDs to update or remove steps.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "00000": ["Cannot update the completed tasks"]
  }
}
```

</td>
  <td>This happens when you try to update tasks (the IDs in the "update_steps") which are/is already completed.</td>
  <td><strong>Note that</strong>, you can only update task(s) that has not completed.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0590": ["Cannot remove tasks"]
  }
}
```

</td>
  <td>This happens when IDs in "delete_steps" are invalid.</td>
  <td><strong>Note that</strong>, you can only remove tasks that exist in your current order record.</td>
  </tr>
</table>

<!-- theme: info -->

> #### Note
>
> Other validation(s) that take place in the order creation will also take place during updating of order record.

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

## Partner

### Invite - [Send](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partners~1{cip}~1send_invite/put), [Cancel](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partners~1{cip}~1cancel_invite/put), [Accept](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partners~1{cip}~1accept_invite/put), [Reject](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partners~1{cip}~1reject_invite/put)

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
    "PT1801": ["Invalid CIP"]
  }
}
```

</td>
    <td>This happens when "cip" value is invalid.</td>
    <td>Ensure that "cip" value is valid, you can get this value from Manage > Company Setting.</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "PT1802": ["Invalid partner request"]
  }
}
```

</td>
    <td>This happens when there is no such upstream and downstream partner request within your company slug.</td>
    <td><strong>Note that</strong>, "cip" is a required field, therefore, there should be a defined relationship between 2 entities (your company with your downstream partner) in order to perform these actions. (send, accept, reject or cancel an invite).</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "PT1803": ["Cannot add current company as a partner"]
  }
}
```

</td>
    <td>This happens when you try to send an invite with your own company cip.</td>
    <td><strong>Note that</strong>, "cip" refers to your downstream partner CIP.</td>
  </tr>
</table>

### Transfer Order - [Create](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partner_transfer~1sender~1create_order/post), [Cancel](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partner_transfer~1sender~1withdraw_order~1{order_number}/put), [Accept](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partner_transfer~1dispatcher~1accept_order~1{order_number}/put), [Reject](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-dispatcher-api.yaml/paths/~1api~1v3~1dispatcher~1partner_transfer~1dispatcher~1reject_order~1{order_number}/put)

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
    "BK0543": ["Transfer already accepted"]
  }
}
```

</td>
    <td>This happens when order is already in accepted state.</td>
    <td rowspan="2">Ensure that order record should be in "unassigned" state before it can be "accepted" or "cancelled".</td>
  </tr>
  <tr>
<td>

```json
{
  "data": {
    "BK0544": ["Transfer is already cancelled"]
  }
}
```

</td>
    <td>This happens when order has already been cancelled.</td>
  </tr>
</table>
