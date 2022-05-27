**API Common Error Guide**

# Overview

In this document, you can find the common API error messages for different endpoints, the description of the message and steps on how to resolve those issues.

### [Multi-leg order creation](https://yojee.stoplight.io/docs/yojee-api/publish/yojee-order-api.yaml/paths/~1api~1v3~1dispatcher~1orders_multi_leg/post)

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
            This happens when the value of "to_time" is earlier than "from_time".
        </td>
        <td>
            Ensure that both "from_time" and "to_time" value are valid and "to_time" value should always be later than "from_time" value.
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
        <td>Ensure that this parameter should not be an empty array in the request payload.</td>
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
        <td>This happens when "from_time" or "to_time" value is not in ISO8601 format.</td>
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
            This happens when the total count of "items", "steps" and "item_steps" do not tally. </br>
            There are few scenarios that this error message will occur: </br>
            <ul>
                <li>"item_steps" is empty when both "items" and "steps" are not empty.</li>
                <li>"items" or "steps" has extra entries that "item_steps" is not referring to.</li>
            </ul>
        </td>
        <td>
            Ensure that "items", "steps", "item_steps" total count should be tally. </br>
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
        <td>There are few scenarios that this error message will occur: <br/>
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
