
**Downstream Partner Integration Solution Guide**
## Overview

---


Yojee’s customers use our platform to track their transport orders. In some use cases, they transfer orders to their downstream partners (DSPs for abbreviation) for further order execution. This is achieved through creating a separate instance in Yojee (we refer to this as a ‘slug’) and providing the DSP to access this slug. The DSP will proceed to assign drivers to the transferred orders and their drivers will make use of Yojee’s mobile app to provide the status updates. The status updates will be visible in both the DSP and the upstream partner (USP), Yojee customer’s slugs.

**Section 1** in this document will outline the current operations for the above flow in further detail.

In certain cases, the DSPs have their own Transport Management Systems, and would like to integrate with Yojee. To achieve this, there is a need to call specific Yojee APIs. **Section 2** in this document will first outline the integration flow, and describe the individual API calls needed. It is also important to understand the flow in Section 1 to be able to understand how the integration flow will work.




### <span style="text-decoration:underline;">1. Current Operations</span>

The diagram below outlines the main components of current transfer order operations.

![main components](../../assets/images/dsp-int-guide/dsp-int-guide-image-01.png)

The components are:



* Order Transfer
    * where the order transfer is initiated by the USP to the DSP and DSP decides whether to accept the transfer
* Driver Management and Assignment
    * where the DSP manages its own drivers and assign the order tasks to the driver
* Task Status Tracking + POD updates
    * where the Driver assigned will use the Drivers’ Mobile App to mark the various stages of pickup/dropoff task completions, and upload signatures or pictures of the delivery as PODs

In the diagram above:



* **Company A** has access to the Dispatcher Portal for its own slug (green background with thick borders).
* **Company B** (downstream partner) has access to the Dispatcher for its own slug (green background).
* **Driver** has access to the Mobile App


#### 1.1 Order Transfer





##### Process Description



1. When Company A has an order it wishes to transfer to its DSP Company B, the Company A Dispatcher will go to Company A’s Yojee Dispatcher portal (in short, in **Co. A’s slug**) to select the order and initiate the transfer. The order will be in the status **Transferred - Pending** in Co. A’s slug.

    Company B Dispatcher will see an incoming order with options  to **Accept** or **Decline** in Company B’s Yojee Dispatcher portal (in short, in **Co. B’s slug**).

2. If Company B **declines** the transfer, the order will return as **unassigned** in Co’s A slug, and show as **Cancelled** in Co. B’s slug
3. If Company B **accepts** the transfer, the order will show as **Transferred - Unassigned** in Co. A’s slug and **Accepted** in Co. B’s slug. The order would now be transferred from Company A to Company B.


#### 1.2 Driver Management and Assignment






##### Process Description

After the order has been transferred to Co. B, it is the responsibility of Co. B to assign the order to a Driver for the execution of the order.

For Driver Management, the Co. B Dispatcher can go to Co. B’s slug to:



1. Create/Edit/Delete Drivers. Once drivers have been created in Co. B’s slug, the orders can be assigned.

For each order, there is a concept of tasks. In the most common use cases, an order will have a **pickup** task and a **dropoff** task. After the order has been transferred to Co. B, the tasks will also be visible in Co. B’s slug as **Unassigned**.

To assign the task(s) to a driver, in Co. B’s slug:



2. Assign the task(s) to a driver. The driver will see new tasks in **Incoming Assignments** on the mobile app. He can choose to Accept or Reject.
3. If the driver **rejects** the incoming assignment, the task will show as **Unassigned** again in Co. B’s slug
4. If the driver **accepts** the incoming assignment, the task will show as **Assigned** in Co. B’s slug. At the same time, the order will show as Transferred-Assigned in Co. A’s slug.


####


#### 1.3 Task Status Tracking  + POD updates


##### Process Description



1. When the driver arrives at the location, he has to first use the mobile app to **Mark as Arrived**. A message will be sent to mark the driver’s arrival and the driver’s arrival time can be viewed in the order’s item audit log in Co. B’s slug.

After that, depending on whether there is any task exception, meaning that whether it is possible to complete the task, it will either be **Completed** or **Reported**



2. If the task can be completed, the driver will proceed to scan the QR Code in the waybill.
3. The driver will take a photo or obtain the required signature. The rules for whether photo and/or signature are required is configurable. After the photo and/or signature are obtained, the mobile app will upload the images to the cloud.
4. Driver will confirm completion of the task, and
5. Mark departure from the location. At this point, messages will be sent to the Yojee backend to update the task as completed. In Co. B’s slug the task will show as **Completed** along with the POD link. In Co. A’s slug the task will show as **Transferred-Completed** along with the POD link.
6. If the task cannot be completed, the driver will click on the ‘report’ link and choose a Task Exception Reason Code, or enter a customer reason code.
7. The driver will take a photo or obtain the required signature. The rules for whether photo and/or signature are required is configurable. After the photo and/or signature are obtained, the mobile app will upload the images to the cloud.
8. Driver will confirm reporting of the task, and
9. Mark departure from the location. The task would then be marked as Reported. At this point, messages will be sent to the Yojee backend to update the task as reported. In Co. B’s slug the task will show as **Reported** along with the POD link. In Co. A’s slug the task will show as **Transferred-Reported** along with the POD link.


###


### <span style="text-decoration:underline;">2. Integration Solution</span>


#### Integration Solution Overview


With the understanding in Section 1 of how the Current Operations work, the solution to integrate Yojee with DSP’s TMS is designed as follows:



* Co. A will continue to use its slug in Yojee for its operations
* A slug will be created for Co. B (DSP) in Yojee. Co. A will continue to transfer orders to Co. B through Co. A’s slug.
* An **Integration Layer** will need to be introduced between Yojee’s backend and DSP’s TMS to be the bridge between both systems. The Integration Layer will communicate with Yojee using a set of standard Yojee APIs to perform the operations previously performed manually in Co. B’s slug and the drivers’ mobile app. Decision points in the Integration Layer, like whether the transfer order is accepted and whether the driver accepts the tasks, will likely be dependent on the information coming from the Co. B’s TMS.
* There will also be communications between the Integration Layer and Co. B’s TMS, but this is out of scope of this document.

The Integration Solution can also be broken down into the following components.



* Order Transfer
    * to retrieve incoming order details
    * to accept / decline the incoming orders
* Driver Management and Assignment
    * to manage drivers
    * to retrieve outstanding tasks to be worked on
    * to assign tasks to drivers
    * to accept / reject orders from the perspective of drivers
* Task Status Tracking + POD updates
    * to send status updates to Yojee backend
    * to notify Yojee backend of uploaded photo and/or signature images.




#### 2.1 Order Transfer




The API calls under **Order Transfer** in this solution will need to be called using Co. B’s Dispatcher credentials.


##### 2.1.1 Retrieve incoming order details

Incoming transfer orders will be in Co. B’s slug as orders with **created** status. To retrieve the order information and the order item information for these orders, we will need to make 2 calls:



* Dispatcher Get Orders with status **created**.
* Dispatcher Get OrderItems by retrieving the **order_id** from the call above.


######


###### **Dispatcher Get Orders**

This API call will retrieve orders matching the criteria provided in the parameters


<table>
  <tr>
   <td>GET
   </td>
   <td>[BASEURL]/api/v3/dispatcher/orders
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request Query Parameters


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
   <td>page_size
   </td>
   <td>N
   </td>
   <td>For pagination purposes
   </td>
  </tr>
  <tr>
   <td>page
   </td>
   <td>N
   </td>
   <td>For pagination purposes
   </td>
  </tr>
  <tr>
   <td>order
   </td>
   <td>N
   </td>
   <td>‘asc’ or ‘desc’
   </td>
  </tr>
  <tr>
   <td>status
   </td>
   <td>N
   </td>
   <td>An array containing either of the follow:
<p>
‘created’, ‘accepted’, ‘completed’, ‘cancelled’ to filter the results
   </td>
  </tr>
  <tr>
   <td>from
   </td>
   <td>N
   </td>
   <td>To filter the order creation datetime. In UTC format
   </td>
  </tr>
  <tr>
   <td>to
   </td>
   <td>N
   </td>
   <td>To filter the order creation datetime. In UTC format
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location -g --request GET '[BASEURL]/api/v3/dispatcher/orders?page_size=50&page=1&order=desc&status[]=created&from=2021-05-15T16:00:00.000Z&to=2021-05-16T16:00:00.000Z' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>information retrieved successfully
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": [
        {
            "allow_transfer": false,
            "cancelled_at": null,
            "completion_time": null,
            "container_no": null,
            "display_price": "SGD 0",
            "editable": true,
            "extension": {},
            "external_id": "INVKCY-202105-002",
            "id": 248602,
            "inserted_at": "2021-05-16T10:23:21.112422Z",
            "is_from_transfer": true,
            "number": "O-8AWKFQYH168U",
            "order_item_statuses": {
                "cancelled": 0,
                "completed": 0,
                "created": 1,
                "processing": 0
            },
            "paid": false,
            "placed_by_user_profile_id": 1540,
            "price": {
                "amount": "0.00000000",
                "currency": "SGD"
            },
            "sender": {
                "id": 1441,
                "name": "Yojee Development",
                "organisation_name": "Yojee",
                "type": "organisation"
            },
            "status": "created"
        }
    ],
    "message": "Got list of company orders.",
    "pagination": {
        "current_page": 1,
        "limit_value": 50,
        "total_count": 1,
        "total_pages": 1
    }
}
```



###### **Dispatcher Get OrderItems**

This API call will retrieve order items matching the criteria provided in the parameters. Use the order id from the result of the Dispatcher** Get Order** call.


<table>
  <tr>
   <td>GET
   </td>
   <td>[BASEURL]/api/v3/dispatcher/order_items
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request Query Parameters


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
   <td>page_size
   </td>
   <td>N
   </td>
   <td>For pagination purposes
   </td>
  </tr>
  <tr>
   <td>page
   </td>
   <td>N
   </td>
   <td>For pagination purposes
   </td>
  </tr>
  <tr>
   <td>order_id
   </td>
   <td>N
   </td>
   <td>order_id
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location -g --request GET '[BASEURL]/api/v3/dispatcher/order_items?order_id=248602 \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>information retrieved successfully
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": [
        {
            "price_currency": null,
            "external_id": "INVKCY-202105-002",
            "external_customer_id3": null,
            "service_type": "same_day",
            "price_amount": null,
            "tracking_number": "YOJ-MYL9KVCSYMLE",
            "container_no": null,
            "sender_id": 1441,
            "price_info": null,
            "to_lng": 103.8977016,
            "weight": null,
            "item_number": 271358,
            "external_customer_id2": null,
            "to_address": "Blk 266C Punggol Way",
            "volume": null,
            "sender_name": null,
            "description": "MOISTURIZER 200ml",
            "order_id": 248602,
            "sender_company": "Company A",
            "transfer_info": null,
            "recipient_name": "Mr Lim Kok Leong",
            "service_type_id": null,
            "status": "created",
            "allow_transfer": true,
            "from_lat": 1.3965495,
            "from_address": "Blk 330A Anchorvale Street",
            "distance": 2131,
            "recipient_company": null,
            "height": null,
            "quantity": 1,
            "sender_phone": null,
            "to_lat": 1.4050971,
            "id": 272061,
            "length": null,
            "task_group_state": "unassigned",
            "from_lng": 103.889886,
            "external_customer_id": null,
            "task_group_id": 285636,
            "created_at": "2021-05-16T10:23:21.118434Z",
            "width": null,
            "order_number": "O-8AWKFQYH168U",
            "vehicle_type": null,
            "info": null,
            "recipient_phone": "912345678",
            "is_from_transfer": true,
            "cancelled_at": null
        }
    ],
    "message": "Got list of company orders_items.",
    "pagination": {
        "current_page": 1,
        "limit_value": 25,
        "total_count": 1,
        "total_pages": 1
    }
}
```



##### 2.1.2 Accept the transfer order


###### **Dispatcher Accept Partner Transfer Order**

Call this API to **accept** the transfer order from upstream partner.


<table>
  <tr>
   <td>PUT
   </td>
   <td>[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/accept_order/{order_number}
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request URL Parameters


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
   <td>Use the number (which is the order number) returned in the Dispatcher Get Order call.
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request PUT '[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/accept_order/O-8AWKFQYH168U' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'

```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Accepted successfully
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessable Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "message": "Transfer Order accepted successfully"
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "00000": [
            "no_matching_order"
        ]
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "BK0543": [
            "Transfer already accepted"
        ]
    }
}
```



##### 2.1.3 Decline the transfer order


###### **Dispatcher Reject Partner Transfer Order**

Call this API to **reject** the transfer order from upstream partner.


<table>
  <tr>
   <td>PUT
   </td>
   <td>[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/reject_order/{order_number}
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request URL Parameters


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
   <td>Use the number (which is the order number) returned in the Dispatcher Get Order call.
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request PUT '[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/reject_order/O-8AWKFQYH168U' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'

```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Rejected successfully
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessable Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "message": "Transfer order rejected successfully"
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "00000": [
            "no_matching_order"
        ]
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "BK0512": [
            "Order already cancelled"
        ]
    }
}
```



####


#### 2.2 Driver Management and Assignment







There are two sets of API calls in **Driver Management and Assignment**. The first set are to achieve the following:



* Before assigning a driver to a task, we need to create drivers in the Yojee system. The **Dispatcher Create Worker**, **Dispatcher Update Worker** and **Dispatcher Delete Worker** API calls are used to manage Drivers in Yojee.
* After the Driver is created, we will need to call **Dispatcher Get Tasks** for the list of tasks that are pending assignment.
* With the list of the pending tasks and the list of drivers, the integration layer will need to decide which driver to assign the tasks to, and call **Dispatcher Assign Driver to Tasks.**

For the first set of API calls, the Integration Layer needs to authenticate as Dispatcher to Yojee. This is typically done by including the COMPANY_SLUG and the Dispatcher’s ACCESS_TOKEN in the HTTP header.

The second set of API calls are to achieve the following:



* Get list of tasks assigned to the particular driver, but pending his acceptance. This calls **Worker Get TaskGroups** with status **assigned** and **broadcast**.
* If the Driver rejects the assignment, **Worker TaskGroups Reject** will be called.
* If the Driver accepts the assignment, **Worker TaskGroups Accept** will be called.

For this second set of API calls, the Integration Layer needs to authenticate as a Driver to Yojee, as the calls will take the id of the authenticated account as part of the input to perform the operations. This is typically done through JWT tokens.


```
Note: See the section on Basic Information on APIs - Authentication at the end of this document for more information on authentication.
```



##### 2.2.1 Driver Management - Create Driver


###### **Dispatcher Create Worker**

Call this API to **create** a new Driver the transfer order from upstream partner.


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/dispatcher/workers
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request Body Parameters


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
   <td>phone
   </td>
   <td>Y
   </td>
   <td>Phone number of the driver. It is also used to authenticate the driver.
   </td>
  </tr>
  <tr>
   <td>vehicle_type_id
   </td>
   <td>Y
   </td>
   <td>Array of vehicle type ids that this driver can use.
   </td>
  </tr>
  <tr>
   <td>current_vehicle_type_id
   </td>
   <td>Y
   </td>
   <td>The current vehicle this driver is using.
   </td>
  </tr>
  <tr>
   <td>tester
   </td>
   <td>Y
   </td>
   <td>Use ‘true’. This is needed to allow the driver to be authenticated using the static_otp_token.
   </td>
  </tr>
  <tr>
   <td>static_otp_token
   </td>
   <td>Y
   </td>
   <td>Used to authenticate the driver.
   </td>
  </tr>
  <tr>
   <td>name
   </td>
   <td>Y
   </td>
   <td>Driver’s name
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/dispatcher/workers/' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
  "vehicle_type_ids": [
    1793
  ],
  "current_vehicle_type_id": 1793,
  "phone": "+6522222226",
  "tester": true,
  "static_otp_token": "112233",
  "name": "Driver KCY from API 4"
}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Created successfully
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessable Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
Note: The driver_id in this payload is highlighted in bold. This is the id to use if we are to assign the driver to tasks
```



```
{
    "data": {
        "external_id": null,
        "name": "Driver KCY from API 4",
        "vehicle_type_registers": [
            {
                "name": "Van",
                "vehicle_type_id": 1793
            }
        ],
        "start_address": null,
        "driver_license_photo_back": null,
        "inserted_at": "2021-05-16T14:56:49.050997Z",
        "active_worker_sequence_optimised_at": null,
        "approved_at": "2021-05-16T14:56:49.075277Z",
        "national_id_photo_front": null,
        "ongoing_tasks_count": 0,
        "end_location": null,
        "regions": [],
        "distance_away": null,
        "start_location": null,
        "driver_license_photo_front": null,
        "national_id_photo_back": null,
        "static_otp_token": "112233",
        "deleted_at": null,
        "status": "off_duty",
        "location": null,
        "extension": {},
        "end_address": null,
        "driver_license": null,
        "national_id": null,
        "avatar": null,
        "id": 4233,
        "assigned_vehicle": null,
        "phone": "+6522222226",
        "otp_token": null,
        "tags": [],
        "email": null,
        "metadata": {},
        "vehicle": null,
        "contract": "employee",
        "last_seen": null,
        "tester": true,
        "current_vehicle_type_id": 1793,
        "user_profile_id": 9876,
        "default_vehicle": null
    },
    "message": "Worker account created."
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "AC0007": [
            "The account already exists for this role"
        ]
    }
}
```



##### 2.2.2 Driver Management - Update Driver Information


###### **Dispatcher Update Worker**

Call this API to **update** a Driver’s details


<table>
  <tr>
   <td>PUT
   </td>
   <td>[BASEURL]/api/v3/dispatcher/workers/{id}
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request URL parameters


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
   <td>id
   </td>
   <td>Y
   </td>
   <td>id if the Driver
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Body Parameters


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
   <td>phone
   </td>
   <td>Y
   </td>
   <td>Phone number of the driver. It is also used to authenticate the driver.
   </td>
  </tr>
  <tr>
   <td>vehicle_type_id
   </td>
   <td>N
   </td>
   <td>Array of vehicle type ids that this driver can use.
   </td>
  </tr>
  <tr>
   <td>current_vehicle_type_id
   </td>
   <td>N
   </td>
   <td>The current vehicle this driver is using.
   </td>
  </tr>
  <tr>
   <td>tester
   </td>
   <td>N
   </td>
   <td>Use ‘true’. This is needed to allow the driver to be authenticated using the static_otp_token.
   </td>
  </tr>
  <tr>
   <td>static_otp_token
   </td>
   <td>N
   </td>
   <td>Used to authenticate the driver.
   </td>
  </tr>
  <tr>
   <td>name
   </td>
   <td>N
   </td>
   <td>Driver’s name
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request PUT '[BASEURL]/api/v3/dispatcher/workers/4232' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{
  "vehicle_type_ids": [
    1792
  ],
  "current_vehicle_type_id": 1792,
  "phone": "+6522222225",
  "static_otp_token": "1122334455",
}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Updated successfully
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessable Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
Note: The driver_id in this payload is highlighted in bold. This is the id to use if we are to assign the driver to tasks
```



```
{
    "data": {
        "external_id": null,
        "name": "Driver Tan",
        "vehicle_type_registers": [
            {
                "name": "Car",
                "vehicle_type_id": 1792
            }
        ],
        "start_address": null,
        "driver_license_photo_back": null,
        "inserted_at": "2021-05-16T14:53:31.676384Z",
        "active_worker_sequence_optimised_at": null,
        "approved_at": "2021-05-16T14:53:31.696382Z",
        "national_id_photo_front": null,
        "ongoing_tasks_count": 0,
        "end_location": null,
        "regions": [],
        "distance_away": null,
        "start_location": null,
        "driver_license_photo_front": null,
        "national_id_photo_back": null,
        "static_otp_token": "11223344",
        "deleted_at": null,
        "status": "off_duty",
        "location": null,
        "end_address": null,
        "driver_license": null,
        "national_id": null,
        "avatar": null,
        "id": 4232,
        "assigned_vehicle": null,
        "phone": "+6522222225",
        "otp_token": "112233",
        "tags": [],
        "email": null,
        "metadata": {},
        "vehicle": null,
        "contract": "employee",
        "last_seen": null,
        "tester": true,
        "current_vehicle_type_id": 1792,
        "user_profile_id": 9875,
        "default_vehicle": null
    },
    "message": "Worker account updated."
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "EC401": [
            "phone can't be blank"
        ]
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "EC424": [
            "phone has already been taken"
        ]
    }
}
```



##### 2.2.3 Driver Management - Delete Driver Information


###### **Dispatcher Delete Worker**

Call this API to **delete** a Driver’s details


<table>
  <tr>
   <td>DELETE
   </td>
   <td>[BASEURL]/api/v3/dispatcher/workers/{id}
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request URL parameters


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
   <td>id
   </td>
   <td>Y
   </td>
   <td>id if the Driver
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request DELETE '[BASEURL]/api/v3/dispatcher/workers/4231' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Deleted successfully
   </td>
  </tr>
  <tr>
   <td>403
   </td>
   <td>Forbidden - either the worker with the id is not found or the worker is not under the current slug
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "message": "Worker account deleted."
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 403


```
{
    "message": "Unauthorized access!"
}
```



##### 2.2.4 Get Tasks


###### **Dispatcher Get Tasks**

This API call will retrieve tasks under the slug.


```
Note: To retrieve tasks pending assignment, use the Request Query Parameters as listed below.
```



<table>
  <tr>
   <td>GET
   </td>
   <td>[BASEURL]/api/v3/dispatcher/tasks
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request Query Parameters


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
   <td>task_group_states[]
   </td>
   <td>N
   </td>
   <td>Use ‘unassigned’ to filter out unassigned tasks
   </td>
  </tr>
  <tr>
   <td>from
   </td>
   <td>N
   </td>
   <td rowspan="2" >To filter on the task datetime. In UTC format. The datetime range covers the “to” and “from” time of the tasks
   </td>
  </tr>
  <tr>
   <td>to
   </td>
   <td>N
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location -g --request GET '[BASEURL]/api/v3/dispatcher/tasks?from=2021-05-17T16:00:00.000Z&to=2021-05-18T16:00:00.000Z&task_group_states[]=unassigned' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>information retrieved successfully
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
Note: The task_ids of the tasks are highlighted in bold in the payload below. They will be used in assigning the driver to the tasks.
```



```
{
    "data": [
        {
            "lng": 103.889886,
            "type": "pickup",
            "state": "created",
            "order_item": {
                "cod_price": null,
                "consolidation_metadata": {},
                "external_customer_id": null,
                "external_customer_id2": null,
                "external_customer_id3": null,
                "id": 272065,
                "isolate_tasks": false,
                "service_type": "same_day",
                "service_type_id": null,
                "state": "created",
                "tracking_number": "YOJ-ZSUITFSQF9G8"
            },
            "associated_task_id": null,
            "order_item_id": 272065,
            "order": {
                "container_no": null,
                "external_id": "INVKCY-202105-003",
                "id": 248605,
                "number": "O-EIJXCMDXIWIJ",
                "price": {
                    "amount": "0.00000000",
                    "currency": "SGD"
                },
                "sender": {
                    "id": 1441,
                    "name": "Yojee Development",
                    "organisation_name": "Yojee",
                    "sender_type": "organisation"
                },
                "sender_name": "Yojee Development",
                "sender_organisation_name": "Yojee",
                "status": "accepted"
            },
            "order_step_id": 491954,
            "task_group": {
                "accepted_time": null,
                "broadcast_expires_at": null,
                "id": 285641,
                "price": null,
                "state": "unassigned"
            },
            "contact": {
                "company": "Company A",
                "name": null,
                "phone": null
            },
            "inserted_at": null,
            "step_group_optimizable": true,
            "weight": null,
            "start_time": null,
            "worker": {
                "id": null,
                "name": null
            },
            "step_group": 1,
            "sequence_id": null,
            "description": "MOISTURIZER 200ml",
            "step_sequence": 1,
            "item": {
                "description": "MOISTURIZER 200ml",
                "global_tracking_number": "Y-D7HNULQOJZMS",
                "height": null,
                "length": null,
                "payload_type": "package",
                "quantity": null,
                "volume": 1.0,
                "volumetric_weight": null,
                "weight": null,
                "width": null
            },
            "sequence_group": 1,
            "has_complete_info": true,
            "volumetric_weight": null,
            "eta": null,
            "location": {
                "address": "Blk 330A Anchorvale Street",
                "address2": null,
                "country": "Singapore",
                "from": "2021-05-18T02:00:00.000000Z",
                "lat": 1.3965495,
                "lng": 103.889886,
                "postal_code": null,
                "state": null,
                "to": "2021-05-18T04:30:00.000000Z"
            },
            "position": null,
            "to": "2021-05-18T04:30:00.000000Z",
            "service_by_hub_id": null,
            "lat": 1.3965495,
            "height": null,
            "worker_id": null,
            "quantity": 1,
            "sequence": {
                "created_at": null,
                "eta": null,
                "id": null,
                "position": null,
                "trip_id": null,
                "vehicle": null
            },
            "address": "Blk 330A Anchorvale Street",
            "id": 568259,
            "stop_id": null,
            "length": null,
            "consolidation_metadata": {},
            "completion_time": null,
            "step_group_size": 2,
            "task_group_id": 285641,
            "width": null,
            "completion_quantity": null,
            "order_item_step_id": 531458,
            "step_quantity": 1,
            "from": "2021-05-18T02:00:00.000000Z"
        },
        {
            "lng": 103.8977016,
            "type": "dropoff",
            "state": "created",
            "order_item": {
                "cod_price": null,
                "consolidation_metadata": {},
                "external_customer_id": null,
                "external_customer_id2": null,
                "external_customer_id3": null,
                "id": 272065,
                "isolate_tasks": false,
                "service_type": "same_day",
                "service_type_id": null,
                "state": "created",
                "tracking_number": "YOJ-ZSUITFSQF9G8"
            },
            "associated_task_id": null,
            "order_item_id": 272065,
            "order": {
                "container_no": null,
                "external_id": "INVKCY-202105-003",
                "id": 248605,
                "number": "O-EIJXCMDXIWIJ",
                "price": {
                    "amount": "0.00000000",
                    "currency": "SGD"
                },
                "sender": {
                    "id": 1441,
                    "name": "Yojee Development",
                    "organisation_name": "Yojee",
                    "sender_type": "organisation"
                },
                "sender_name": "Yojee Development",
                "sender_organisation_name": "Yojee",
                "status": "accepted"
            },
            "order_step_id": 491955,
            "task_group": {
                "accepted_time": null,
                "broadcast_expires_at": null,
                "id": 285641,
                "price": null,
                "state": "unassigned"
            },
            "contact": {
                "company": null,
                "name": "Mr Lim Kok Leong",
                "phone": "912345678"
            },
            "inserted_at": null,
            "step_group_optimizable": true,
            "weight": null,
            "start_time": null,
            "worker": {
                "id": null,
                "name": null
            },
            "step_group": 1,
            "sequence_id": null,
            "description": "MOISTURIZER 200ml",
            "step_sequence": 2,
            "item": {
                "description": "MOISTURIZER 200ml",
                "global_tracking_number": "Y-D7HNULQOJZMS",
                "height": null,
                "length": null,
                "payload_type": "package",
                "quantity": null,
                "volume": 1.0,
                "volumetric_weight": null,
                "weight": null,
                "width": null
            },
            "sequence_group": 1,
            "has_complete_info": true,
            "volumetric_weight": null,
            "eta": null,
            "location": {
                "address": "Blk 266C Punggol Way",
                "address2": null,
                "country": "Singapore",
                "from": "2021-05-18T06:00:00.000000Z",
                "lat": 1.4050971,
                "lng": 103.8977016,
                "postal_code": null,
                "state": null,
                "to": "2021-05-18T08:30:00.000000Z"
            },
            "position": null,
            "to": "2021-05-18T08:30:00.000000Z",
            "service_by_hub_id": null,
            "lat": 1.4050971,
            "height": null,
            "worker_id": null,
            "quantity": 1,
            "sequence": {
                "created_at": null,
                "eta": null,
                "id": null,
                "position": null,
                "trip_id": null,
                "vehicle": null
            },
            "address": "Blk 266C Punggol Way",
            "id": 568260,
            "stop_id": null,
            "length": null,
            "consolidation_metadata": {},
            "completion_time": null,
            "step_group_size": 2,
            "task_group_id": 285641,
            "width": null,
            "completion_quantity": null,
            "order_item_step_id": 531459,
            "step_quantity": 1,
            "from": "2021-05-18T06:00:00.000000Z"
        }
    ],
    "pagination": {
        "current_page": 1,
        "limit_value": 30,
        "total_count": 2,
        "total_pages": 1
    }
}
```



##### 2.2.5 Assign Driver

Driver assignment is done by a background job. The first call is to send the parameters to the background job for execution, and the second call is to get the status of the background job to see the outcome.


###### **Dispatcher Assign Driver to tasks**

Call this API to assign a Driver to tasks


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/dispatcher/tasks/quick_assign_bg
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request Body parameters


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
   <td>task_ids
   </td>
   <td>Y
   </td>
   <td>Array of task_ids retrieved from <strong>Dispatcher Get Tasks</strong> call
   </td>
  </tr>
  <tr>
   <td>worker_id
   </td>
   <td>Y
   </td>
   <td>Driver_id
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/dispatcher/tasks/quick_assign_bg' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{"task_ids":[568259,568260],"worker_id":4233}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Quick assign background job triggered successfully
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
Note: The id of the background job is highlighted in bold below. This is the id to use in the Get Background Job Status call.
```



```
{
    "cancelled_at": null,
    "completed_at": null,
    "failed_at": null,
    "id": "c8cc52f50a7c4ef28249377a1645e113",
    "inserted_at": "2021-05-16T17:02:23.787793",
    "processed": null,
    "progress": null,
    "result": null,
    "total": null,
    "type": "quick_assign",
    "updated_at": "2021-05-16T17:02:23.787793"
}
```



###### **Get Background Job status**

Call this API to view the status of the background job


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/dispatcher/tasks/bg_status
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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


<!--H6 not demoted to H7. -->


###### Request Body parameters


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
   <td>id
   </td>
   <td>Y
   </td>
   <td>id returned by the <strong>quick_assign_bg (Assign Driver)</strong> call
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/dispatcher/tasks/bg_status' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN] \
--header 'Content-Type: application/json' \
--data-raw '{"id":"c8cc52f50a7c4ef28249377a1645e113"}'

```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Background job status returned successfully
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "cancelled_at": null,
    "completed_at": "2021-05-16T17:02:24.340561",
    "failed_at": null,
    "id": "c8cc52f50a7c4ef28249377a1645e113",
    "inserted_at": "2021-05-16T17:02:23.787793",
    "processed": null,
    "progress": "Update tasks distance",
    "result": {
        "message": "Allocation successful",
        "status": 200
    },
    "total": null,
    "type": "quick_assign",
    "updated_at": "2021-05-16T17:02:24.340561"
}
```



##### 2.2.5 Driver Get Task Groups


###### **Driver Get TaskGroups**

This call will retrieve the task groups that have been assigned to the Driver, but pending Driver’s acceptance / rejection.


<table>
  <tr>
   <td>GET
   </td>
   <td>[BASEURL]/api/v3/worker/task_groups
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request Body parameters


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
   <td>status[]
   </td>
   <td>Y
   </td>
   <td>Array of status to search for. Use ‘assigned’ and ‘broadcast’
   </td>
  </tr>
  <tr>
   <td>page
   </td>
   <td>N
   </td>
   <td>For pagination
   </td>
  </tr>
  <tr>
   <td>page_size
   </td>
   <td>N
   </td>
   <td>For pagination
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location -g --request GET '[BASEURL]/api/v3/worker/task_groups?status[]=assigned&status[]=broadcasted' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaWauc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjM1MTk0LCJpYXQiOjE2MjEyMzQyOTQsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2c2cwMnM5ODF1NWxiNHAwMDAwZzczIiwibmJmIjoxNjIxMjM0Mjk0LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.n8xHAVHC86nloGSKAmC0SCJ2HsHBdv8Z7zEj5HIekL8' \
--data-raw ''
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Task groups retrieved successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": {
        "entries": [
            {
                "accepted_time": null,
                "assigned_time": "2021-05-16T16:53:41.529262Z",
                "id": 285642,
                "price": "SGD 0",
                "state": "assigned",
                "tasks": [
                    {
                        "external_id": "INVKCY-202105-003",
                        "lng": 103.889886,
                        "type": "pickup",
                        "state": "created",
                        "external_customer_id3": null,
                        "order_item_tracking_number": "YOJ-ZSUITFSQF9G8",
                        "service_type": "same_day",
                        "order_item_id": 272065,
                        "arrival_time": null,
                        "postal_code": null,
                        "order_step_id": 491954,
                        "container_no": null,
                        "contact": {
                            "contact_company": "Company A",
                            "email": null,
                            "name": null,
                            "phone": null
                        },
                        "cod_price_amount": null,
                        "cod_price_currency": null,
                        "start_time": null,
                        "item": {
                            "global_tracking_number": "Y-D7HNULQOJZMS",
                            "height": null,
                            "item_container": null,
                            "length": null,
                            "payload_type": "package",
                            "quantity": null,
                            "volume": "1",
                            "volumetric_weight": null,
                            "weight": null,
                            "width": null
                        },
                        "external_customer_id2": null,
                        "additional_info": null,
                        "description": "MOISTURIZER 200ml",
                        "departure_time": null,
                        "address_state": null,
                        "cancelled_time": null,
                        "unassign_time": null,
                        "address2": null,
                        "to": "2021-05-18T04:30:00.000000Z",
                        "lat": 1.3965495,
                        "country": "Singapore",
                        "quantity": 1,
                        "cod_price": null,
                        "address": "Blk 330A Anchorvale Street",
                        "id": 568259,
                        "stop_id": 38088,
                        "external_customer_id": null,
                        "completion_time": null,
                        "task_group_id": 285642,
                        "from": "2021-05-18T02:00:00.000000Z"
                    },
                    {
                        "external_id": "INVKCY-202105-003",
                        "lng": 103.8977016,
                        "type": "dropoff",
                        "state": "created",
                        "external_customer_id3": null,
                        "order_item_tracking_number": "YOJ-ZSUITFSQF9G8",
                        "service_type": "same_day",
                        "order_item_id": 272065,
                        "arrival_time": null,
                        "postal_code": null,
                        "order_step_id": 491955,
                        "container_no": null,
                        "contact": {
                            "contact_company": null,
                            "email": null,
                            "name": "Mr Lim Kok Leong",
                            "phone": "912345678"
                        },
                        "cod_price_amount": null,
                        "cod_price_currency": null,
                        "start_time": null,
                        "item": {
                            "global_tracking_number": "Y-D7HNULQOJZMS",
                            "height": null,
                            "item_container": null,
                            "length": null,
                            "payload_type": "package",
                            "quantity": null,
                            "volume": "1",
                            "volumetric_weight": null,
                            "weight": null,
                            "width": null
                        },
                        "external_customer_id2": null,
                        "additional_info": null,
                        "description": "MOISTURIZER 200ml",
                        "departure_time": null,
                        "address_state": null,
                        "cancelled_time": null,
                        "unassign_time": null,
                        "address2": null,
                        "to": "2021-05-18T08:30:00.000000Z",
                        "lat": 1.4050971,
                        "country": "Singapore",
                        "quantity": 1,
                        "cod_price": null,
                        "address": "Blk 266C Punggol Way",
                        "id": 568260,
                        "stop_id": 38089,
                        "external_customer_id": null,
                        "completion_time": null,
                        "task_group_id": 285642,
                        "from": "2021-05-18T06:00:00.000000Z"
                    }
                ]
            },
            {
                "accepted_time": null,
                "assigned_time": "2021-05-16T16:03:29.934364Z",
                "id": 285637,
                "price": "SGD 0",
                "state": "assigned",
                "tasks": [
                    {
                        "external_id": "INVKCY-202105-002",
                        "lng": 103.889886,
                        "type": "pickup",
                        "state": "created",
                        "external_customer_id3": null,
                        "order_item_tracking_number": "YOJ-MYL9KVCSYMLE",
                        "service_type": "same_day",
                        "order_item_id": 272061,
                        "arrival_time": null,
                        "postal_code": "541330",
                        "order_step_id": 491952,
                        "container_no": null,
                        "contact": {
                            "contact_company": "Company A",
                            "email": null,
                            "name": null,
                            "phone": null
                        },
                        "cod_price_amount": null,
                        "cod_price_currency": null,
                        "start_time": null,
                        "item": {
                            "global_tracking_number": "Y-LT2XQFSPJPGI",
                            "height": null,
                            "item_container": null,
                            "length": null,
                            "payload_type": "Package",
                            "quantity": 1,
                            "volume": null,
                            "volumetric_weight": null,
                            "weight": null,
                            "width": null
                        },
                        "external_customer_id2": null,
                        "additional_info": null,
                        "description": "MOISTURIZER 200ml",
                        "departure_time": null,
                        "address_state": null,
                        "cancelled_time": null,
                        "unassign_time": null,
                        "address2": null,
                        "to": "2021-05-17T04:30:00.000000Z",
                        "lat": 1.3965495,
                        "country": "Singapore",
                        "quantity": 1,
                        "cod_price": null,
                        "address": "Blk 330A Anchorvale Street",
                        "id": 568251,
                        "stop_id": 38086,
                        "external_customer_id": null,
                        "completion_time": null,
                        "task_group_id": 285637,
                        "from": "2021-05-17T02:00:00.000000Z"
                    },
                    {
                        "external_id": "INVKCY-202105-002",
                        "lng": 103.8977016,
                        "type": "dropoff",
                        "state": "created",
                        "external_customer_id3": null,
                        "order_item_tracking_number": "YOJ-MYL9KVCSYMLE",
                        "service_type": "same_day",
                        "order_item_id": 272061,
                        "arrival_time": null,
                        "postal_code": "823266",
                        "order_step_id": 491953,
                        "container_no": null,
                        "contact": {
                            "contact_company": null,
                            "email": null,
                            "name": "Mr Lim Kok Leong",
                            "phone": "912345678"
                        },
                        "cod_price_amount": null,
                        "cod_price_currency": null,
                        "start_time": null,
                        "item": {
                            "global_tracking_number": "Y-LT2XQFSPJPGI",
                            "height": null,
                            "item_container": null,
                            "length": null,
                            "payload_type": "Package",
                            "quantity": 1,
                            "volume": null,
                            "volumetric_weight": null,
                            "weight": null,
                            "width": null
                        },
                        "external_customer_id2": null,
                        "additional_info": null,
                        "description": "MOISTURIZER 200ml",
                        "departure_time": null,
                        "address_state": null,
                        "cancelled_time": null,
                        "unassign_time": null,
                        "address2": null,
                        "to": "2021-05-17T08:30:00.000000Z",
                        "lat": 1.4050971,
                        "country": "Singapore",
                        "quantity": 1,
                        "cod_price": null,
                        "address": "Blk 266C Punggol Way",
                        "id": 568252,
                        "stop_id": 38087,
                        "external_customer_id": null,
                        "completion_time": null,
                        "task_group_id": 285637,
                        "from": "2021-05-17T06:00:00.000000Z"
                    }
                ]
            }
        ],
        "pagination": {
            "current_page": 1,
            "limit_value": 10,
            "total_count": 2,
            "total_pages": 1
        }
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```



##### 2.2.6 Driver Accepts Task


###### **Driver Accepts Task**

For the Driver to **accept** the task, call this API.


<table>
  <tr>
   <td>PUT
   </td>
   <td>[BASEURL]/api/v3/worker/task_groups/{id}/accept
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request URL parameters


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
   <td>id
   </td>
   <td>Y
   </td>
   <td>task_group_id retrieved from call to <strong>Driver Get TaskGroups</strong>
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request PUT '[BASEURL]/api/v3/worker/task_groups/285642/accept' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjM1MTk0LCJpYXQiOjE2MjEyMzQyOTQsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2c2cwMnM5ODF1NWxiNHAwMDAwZzczIiwibmJmIjoxNjIxMjM0Mjk0LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.n8xHAVHC86nloGSKAmC0SCJ2HsHBdv8Z7zEj5HIekL8'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Accepted successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessed Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": {
        "accepted_time": "2021-05-17T07:05:18.569846Z",
        "assigned_time": "2021-05-16T16:53:41.529262Z",
        "id": 285642,
        "price": "SGD 0",
        "state": "assigned",
        "tasks": [
            {
                "external_id": "INVKCY-202105-003",
                "lng": 103.889886,
                "type": "pickup",
                "state": "created",
                "external_customer_id3": null,
                "order_item_tracking_number": "YOJ-ZSUITFSQF9G8",
                "service_type": "same_day",
                "order_item_id": 272065,
                "arrival_time": null,
                "postal_code": null,
                "order_step_id": 491954,
                "container_no": null,
                "contact": {
                    "contact_company": "Company A",
                    "email": null,
                    "name": null,
                    "phone": null
                },
                "cod_price_amount": null,
                "cod_price_currency": null,
                "start_time": null,
                "item": {
                    "global_tracking_number": "Y-D7HNULQOJZMS",
                    "height": null,
                    "item_container": null,
                    "length": null,
                    "payload_type": "package",
                    "quantity": null,
                    "volume": "1",
                    "volumetric_weight": null,
                    "weight": null,
                    "width": null
                },
                "external_customer_id2": null,
                "additional_info": null,
                "description": "MOISTURIZER 200ml",
                "departure_time": null,
                "address_state": null,
                "cancelled_time": null,
                "unassign_time": null,
                "address2": null,
                "to": "2021-05-18T04:30:00.000000Z",
                "lat": 1.3965495,
                "country": "Singapore",
                "quantity": 1,
                "cod_price": null,
                "address": "Blk 330A Anchorvale Street",
                "id": 568259,
                "stop_id": 38088,
                "external_customer_id": null,
                "completion_time": null,
                "task_group_id": 285642,
                "from": "2021-05-18T02:00:00.000000Z"
            },
            {
                "external_id": "INVKCY-202105-003",
                "lng": 103.8977016,
                "type": "dropoff",
                "state": "created",
                "external_customer_id3": null,
                "order_item_tracking_number": "YOJ-ZSUITFSQF9G8",
                "service_type": "same_day",
                "order_item_id": 272065,
                "arrival_time": null,
                "postal_code": null,
                "order_step_id": 491955,
                "container_no": null,
                "contact": {
                    "contact_company": null,
                    "email": null,
                    "name": "Mr Lim Kok Leong",
                    "phone": "912345678"
                },
                "cod_price_amount": null,
                "cod_price_currency": null,
                "start_time": null,
                "item": {
                    "global_tracking_number": "Y-D7HNULQOJZMS",
                    "height": null,
                    "item_container": null,
                    "length": null,
                    "payload_type": "package",
                    "quantity": null,
                    "volume": "1",
                    "volumetric_weight": null,
                    "weight": null,
                    "width": null
                },
                "external_customer_id2": null,
                "additional_info": null,
                "description": "MOISTURIZER 200ml",
                "departure_time": null,
                "address_state": null,
                "cancelled_time": null,
                "unassign_time": null,
                "address2": null,
                "to": "2021-05-18T08:30:00.000000Z",
                "lat": 1.4050971,
                "country": "Singapore",
                "quantity": 1,
                "cod_price": null,
                "address": "Blk 266C Punggol Way",
                "id": 568260,
                "stop_id": 38089,
                "external_customer_id": null,
                "completion_time": null,
                "task_group_id": 285642,
                "from": "2021-05-18T06:00:00.000000Z"
            }
        ]
    }
}

```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "WO1017": [
            "Task Group no longer available. Please refresh"
        ]
    }
}
```



##### 2.2.7 Driver Rejects Task


###### **Driver Rejects Task**

For the Driver to **reject** the task, call this API.


<table>
  <tr>
   <td>PUT
   </td>
   <td>[BASEURL]/api/v3/worker/task_groups/{id}/reject
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request URL parameters


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
   <td>id
   </td>
   <td>Y
   </td>
   <td>task_group_id retrieved from call to <strong>Driver Get TaskGroups</strong>
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request PUT '[BASEURL]/api/v3/worker/task_groups/285637/reject \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjM1MTk0LCJpYXQiOjE2MjEyMzQyOTQsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2c2cwMnM5ODF1NWxiNHAwMDAwZzczIiwibmJmIjoxNjIxMjM0Mjk0LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.n8xHAVHC86nloGSKAmC0SCJ2HsHBdv8Z7zEj5HIekL8'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Rejected successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessed Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "message": "Reject a task group successfully."
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "WO1017": [
            "Task Group no longer available. Please refresh"
        ]
    }
}
```



####


#### 2.3 Task Status Tracking + POD updates



##### 2.3.1 Driver Get Ongoing Tasks


###### **Driver Get Ongoing Tasks**

This call will list out the tasks that are currently on hand for the driver.


<table>
  <tr>
   <td>GET
   </td>
   <td>[BASEURL]/api/v3/worker/tasks/ongoing
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request GET '[BASEURL]/api/v3/worker/tasks/ongoing' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjY0MDU3LCJpYXQiOjE2MjEyNjMxNTcsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2dTRnM3NoY2JlYTRyMGlvMDAwNWgzIiwibmJmIjoxNjIxMjYzMTU3LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.ET_-vvCgB6QzfJNd5D3fzX30knMzrM6WCGkJNJqk-u4'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Retrieved successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
Note that there is one task for 'pickup' and one task for 'dropoff'. Also note that each of these tasks, there are sub-tasks for complete and fail. The sub-task information will be needed when formatting the payload for the Driver Bulk Actions call.
```



```
{
    "data": [
        {
            "external_id": "INV350963A",
            "lng": 103.8198,
            "type": "pickup",
            "state": "created",
            "external_customer_id3": null,
            "order_item_tracking_number": "YOJ-YFYSMXG8M33K",
            "service_type": "same_day",
            "order_item_id": 272070,
            "arrival_time": null,
            "postal_code": null,
            "order_step_id": 491956,
            "container_no": null,
            "contact": {
                "email": null,
                "name": null,
                "phone": null
            },
            "start_time": null,
            "external_customer_id2": null,
            "additional_info": null,
            "description": "PRO 95 X 500MM 40KGS KIT DRAWER SILK WHITE Qty 1\nPRO 178 X 500MM 40KGS KIT DRAWER SILK WHITE Qty 4\n",
            "departure_time": null,
            "address_state": "QLD",
            "all_tracking_numbers": [
                "YOJ-YREGBZWE66US",
                "YOJ-YFYSMXG8M33K"
            ],
            "eta": null,
            "address2": null,
            "sender": {
                "email": "yojee@development.com",
                "id": 1441,
                "name": "Yojee Development",
                "phone": "+6598775432"
            },
            "position": 0,
            "to": "2020-11-02T13:59:00.000000Z",
            "lat": 1.3521,
            "country": "Australia",
            "quantity": 5,
            "cod_price": null,
            "address": "19 WILLIAM FARRIOR PLACE EAGLE FARM",
            "id": 568271,
            "stop_id": 38090,
            "external_customer_id": null,
            "completion_time": null,
            "task_group_id": 285649,
            "start_lng": null,
            "assigned_time": "2021-05-17T14:51:45.298952Z",
            "vehicle": null,
            "order_number": "O-XER72YSBQVFV",
            "item": {
                "global_tracking_number": "YOJ-YFYSMXG8M33K",
                "height": null,
                "item_container": null,
                "length": null,
                "payload_type": "package",
                "quantity": null,
                "volume": "1",
                "volumetric_weight": null,
                "weight": null,
                "width": null
            },
            "start_lat": null,
            "from": "2020-11-01T14:00:00.000000Z",
            "sub_tasks": {
                "pickup_completed": [
                    {
                        "action": "capture_arrival_time",
                        "company_id": 997,
                        "event": "pickup_completed",
                        "id": 129598,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 226,
                        "task_id": 568271
                    },
                    {
                        "action": "upload_photo",
                        "company_id": 997,
                        "event": "pickup_completed",
                        "id": 129599,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 253,
                        "task_id": 568271
                    }
                ],
                "pickup_failed": [
                    {
                        "action": "upload_photo",
                        "company_id": 997,
                        "event": "pickup_failed",
                        "id": 129600,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 263,
                        "task_id": 568271
                    }
                ]
            }
        },
        {
            "external_id": "INV350963A",
            "lng": 153.4102,
            "type": "dropoff",
            "state": "created",
            "external_customer_id3": null,
            "order_item_tracking_number": "YOJ-YFYSMXG8M33K",
            "service_type": "same_day",
            "order_item_id": 272070,
            "arrival_time": null,
            "postal_code": null,
            "order_step_id": 491957,
            "container_no": null,
            "contact": {
                "email": null,
                "name": null,
                "phone": null
            },
            "start_time": null,
            "external_customer_id2": null,
            "additional_info": null,
            "description": "PRO 95 X 500MM 40KGS KIT DRAWER SILK WHITE Qty 1\nPRO 178 X 500MM 40KGS KIT DRAWER SILK WHITE Qty 4\n",
            "departure_time": null,
            "address_state": "QLD",
            "all_tracking_numbers": [
                "YOJ-YREGBZWE66US",
                "YOJ-YFYSMXG8M33K"
            ],
            "eta": null,
            "address2": null,
            "sender": {
                "email": "yojee@development.com",
                "id": 1441,
                "name": "Yojee Development",
                "phone": "+6598775432"
            },
            "position": 1,
            "to": "2020-11-02T13:59:00.000000Z",
            "lat": -28.1062,
            "country": "Australia",
            "quantity": 5,
            "cod_price": null,
            "address": "UNIT 2 / 29 ALEX FISHER DRIVE ATD BURLEIGH HEADS",
            "id": 568272,
            "stop_id": 38091,
            "external_customer_id": null,
            "completion_time": null,
            "task_group_id": 285649,
            "start_lng": null,
            "assigned_time": "2021-05-17T14:51:45.298952Z",
            "vehicle": null,
            "order_number": "O-XER72YSBQVFV",
            "item": {
                "global_tracking_number": "YOJ-YFYSMXG8M33K",
                "height": null,
                "item_container": null,
                "length": null,
                "payload_type": "package",
                "quantity": null,
                "volume": "1",
                "volumetric_weight": null,
                "weight": null,
                "width": null
            },
            "start_lat": null,
            "from": "2020-11-02T02:00:00.000000Z",
            "sub_tasks": {
                "dropoff_completed": [
                    {
                        "action": "upload_signature",
                        "company_id": 997,
                        "event": "dropoff_completed",
                        "id": 129602,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 255,
                        "task_id": 568272
                    }
                ],
                "dropoff_failed": [
                    {
                        "action": "upload_signature",
                        "company_id": 997,
                        "event": "dropoff_failed",
                        "id": 129601,
                        "meta": {
                            "signature_message": "Singature on task report"
                        },
                        "optional": false,
                        "sub_task_rule_id": 280,
                        "task_id": 568272
                    }
                ]
            }
        }
    ],
    "pagination": {
        "current_page": 1,
        "limit_value": 10,
        "total_count": 2,
        "total_pages": 1
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```



##### 2.3.2 Validate Completion


###### **Validate Completion**

Before updating status for a task, we need to call Validate Completion to ensure that the task can be acted upon.


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/worker/tasks/validate_completion
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request Body Parameters


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
   <td>task_ids
   </td>
   <td>Y
   </td>
   <td>Array containing a list of task_id for validation
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/worker/tasks/validate_completion' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjY0MDU3LCJpYXQiOjE2MjEyNjMxNTcsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2dTRnM3NoY2JlYTRyMGlvMDAwNWgzIiwibmJmIjoxNjIxMjYzMTU3LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.ET_-vvCgB6QzfJNd5D3fzX30knMzrM6WCGkJNJqk-u4' \
--header 'Content-Type: application/json' \
--data-raw '{
    "task_ids": [
        568271
    ]
}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Validation call executed successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200 - Validation passed, task can be acted upon


```
Note that the response will be 200 even if there are validation errors. The return payload needs to be checked for any messages in the "errors" field for any errors
```



```
{
    "data": {
        "errors": [],
        "ok": [
            568271
        ]
    },
    "message": "validate completion successfully"
}
```


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200 - Validation failed, check error message


```
{
    "data": {
        "errors": [
            {
                "message": "Previous tasks had not been completed",
                "task_ids": [
                    568272
                ],
                "type": "previous_task_not_completed"
            }
        ],
        "ok": []
    },
    "message": "validate completion successfully"
}
```


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200 - Validation failed, check error message


```
{
    "data": {
        "errors": [
            {
                "message": "Tasks do not exist",
                "task_ids": [
                    5682721
                ],
                "type": "task_not_found"
            }
        ],
        "ok": []
    },
    "message": "validate completion successfully"
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```



##### 2.3.4 Driver Mark Arrival


###### **Driver Mark Arrival**

This call is to mark the arrival of the Drive at the location for a task


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/worker/tasks/mark_arrival
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request Body Parameters


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
   <td>task_ids
   </td>
   <td>Y
   </td>
   <td>Array containing a list of task_id for validation
   </td>
  </tr>
  <tr>
   <td>arrival_time
   </td>
   <td>N
   </td>
   <td rowspan="2" >Time of arrival/departure in UTC. This API call is used for marking both, so at least one of the parameters is require
   </td>
  </tr>
  <tr>
   <td>departure_time
   </td>
   <td>N
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/worker/tasks/mark_arrival' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: NAN3acMWpyzLpJ7f2yNqt238e3aDxzfx0OjikfF2ORc=' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjY0OTc0LCJpYXQiOjE2MjEyNjQwNzQsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2dTY1Z252aDg3dnNnY3RvMDAwNXA2IiwibmJmIjoxNjIxMjY0MDc0LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.hLX1S96n-e2h1WdS4S7DqZWEu0SJzXbQee_4BqgtXkY' \
--header 'Content-Type: application/json' \
--data-raw '{
    "task_ids": [
        568271
    ],
    "arrival_time": "2021-05-17T15:16:37.240Z"
}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Arrival/Departure marked successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200 - Arrival


```
{
    "data": [
        {
            "arrival_lat": 1.3894008,
            "arrival_lng": 103.8867759,
            "arrival_time": "2021-05-17T15:16:37.240000Z",
            "completion_time": null,
            "departure_lat": null,
            "departure_lng": null,
            "departure_time": null,
            "id": 568271,
            "start_lat": null,
            "start_lng": null,
            "start_time": null,
            "state": "created",
            "task_group_id": 285649
        }
    ]
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```



##### 2.3.5 Driver Generate Batch Upload Pre-signed URLs

In some sub-tasks, there is a need to upload POD/signature images to a AWS S3 Bucket. To perform this securely, a pre-signed URL is generated first, and the image is uploaded to AWS S3 via the methods described at [https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html).


###### **Generate Batch Upload Pre-signed URLs**

Generate AWS S3 pre-signed URL for uploading


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/generate_batch_upload_presigned_urls
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request Body Parameters


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
   <td>filenames
   </td>
   <td>Y
   </td>
   <td>Array containing a list of filenames. Generate unique filenames as the files will be later uploaded to S3 bucket and non-unique filenames can get overwritten
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/worker/generate_batch_upload_presigned_urls' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjY1OTAzLCJpYXQiOjE2MjEyNjUwMDMsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2dTdyaHY1NHZia2tsb25nMDAwN3AyIiwibmJmIjoxNjIxMjY1MDAzLCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.ss9hcbqBTWM-1a_QBmINRi4B7UYTbQmRMnAWGLlS6gg' \
--header 'Content-Type: application/json' \
--data-raw '{
    "filenames": [
        "1_signature_1621265036.jpg"
    ]
}
'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Pre-signed URLs generated successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "1_signature_1621265036.jpg": {
        "object": {
            "bucket": "choongyong-yojee-umbrella",
            "name": "/uploads-presigned/kcy-ds/20210517/20210517152407-1_signature_1621265036.jpg",
            "region": "ap-southeast-1"
        },
        "presigned_url": "https://s3.ap-southeast-1.amazonaws.com/choongyong-yojee-umbrella/uploads-presigned/kcy-ds/20210517/20210517152407-1_signature_1621265036.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAJY4J5TPO3XC7JRAQ%2F20210517%2Fap-southeast-1%2Fs3%2Faws4_request&X-Amz-Date=20210517T152407Z&X-Amz-Expires=900&X-Amz-SignedHeaders=host&X-Amz-Signature=cb2f0eaefa0ebd2f9054f938a2ce9e6de5772fe75c6cf8c12f63cc21329bd129"
    }
}
```



```
Note: The step to upload the image to AWS S3 using the pre-signed URL is not part of the Yojee API calls. You will need to upload the file separately and the uploaded file will have the URL of https://[region].amazonaws.com/[bucket]/[name] (see "object" field above).

This URL of the image is used in the call to Driver Bulk Actions.
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```



##### 2.3.6 Driver Bulk Actions

Driver Bulk Actions is where the main status updates take place. For each **pickup** task, there is a **pickup_completed** and a **pickup_failed** array of sub-tasks. For each dropoff task, there is a **dropoff_completed** and a **dropoff_failed** array of sub-tasks.

Each of these arrays of sub-tasks lists the sub-tasks that need to be completed for the respective scenario of **pickup_completed**, **pickup_failed**, **dropoff_completed** and **dropoff_failed**, before the task of **pickup** / **dropoff** can be completed. See the section for **Driver Get Ongoing Task** for the sub-tasks retrieved.

The Bulk Actions call combines the updates of the sub-tasks into one call with a number of action elements. Each Bulk Actions call will return a batch_id which will be used in **Get Driver Bulk Action Status** to check if the bulk action has completed.


###### **Driver Bulk Actions**

This call is to send multiple actions based on the sub-tasks to be completed.


<table>
  <tr>
   <td>PUT
   </td>
   <td>[BASEURL]/api/v3/worker/tasks/bulk_actions
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request Body Parameters - See example payloads later for details


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
   <td>actions
   </td>
   <td>Y
   </td>
   <td>An array of actions to execute
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command

This sample Curl command is to illustrate a complete call to **Driver Bulk Actions**. The breakdown of the individual action elements will be described later in this document.


```
curl --location --request PUT '[BASEURL]/api/v3/worker/tasks/bulk_actions' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjIxMjY3MTAyLCJpYXQiOjE2MjEyNjYyMDIsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2dWExY2RlcTdwaHVqYjAwMDAwZWkxIiwibmJmIjoxNjIxMjY2MjAyLCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9._Oafyj-9Te6ujPukwYbaToM8Xq9Za-Jw6YKCTO8HGUI' \
--header 'Content-Type: application/json' \
--data-raw '{
  "actions": [
    {
      "id": "_subtask_0_0",
      "params": {
        "sub_task_params": {
          "completion_data":{
            "recipient_name": "Tan Ah Kow"
          },
          "completion_location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "completion_time": "2021-05-17T14:15:00Z",
          "photos": [
            "https://s3.ap-southeast-1.amazonaws.com/choongyong-yojee-umbrella/uploads-presigned/kcy-ds/20210517/20210517135718-1_signature_16208060794.jpg"
          ]
        },
        "sub_task_rule_id": 255
      },
      "tasks_ids": [
        568260
      ],
      "type": "complete_sub_task"
    },
    {
      "id": "_task_0",
      "params": {
        "568260": {
          "completion_data": {},
          "completion_time": "2021-05-17T14:15:10Z",
          "descriptions": "",
          "location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "role": "worker",
          "type": "dropoff"
        }
      },
      "tasks_ids": [
        568260
      ],
      "type": "complete"
    },
    {
      "id": "arrival_departure_time_0",
      "params": {
        "arrival_time": "2021-05-17T10:49:03.130Z",
        "departure_time": "2021-05-17T14:15:20.420Z"
      },
      "tasks_ids": [
        568260
      ],
      "type": "mark_arrival_departure"
    }
  ]
}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Batch submitted successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": {
        "batch_id": "VWxqTnJEUXRIUnM9"
    },
    "message": "Bulk actions in progress."
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```



##### Bulk Actions Payload Format

Each of the Bulk Action payloads includes a number of actions. Each action is either:



* A sub-task completion action
* A task completion/report action
* A mark_arrival_departure action

The list of sub-task completion actions are listed when calling **Driver Get Ongoing Tasks**. The respective sub-task from the sample output above is reproduced here (note that the sub-tasks for different configurations might differ)

<!--H6 not demoted to H7. -->


###### Sub-tasks for Pickup


```
             "sub_tasks": {
                "pickup_completed": [
                    {
                        "action": "capture_arrival_time",
                        "company_id": 997,
                        "event": "pickup_completed",
                        "id": 129598,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 226,
                        "task_id": 568271
                    },
                    {
                        "action": "upload_photo",
                        "company_id": 997,
                        "event": "pickup_completed",
                        "id": 129599,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 253,
                        "task_id": 568271
                    }
                ],
                "pickup_failed": [
                    {
                        "action": "upload_photo",
                        "company_id": 997,
                        "event": "pickup_failed",
                        "id": 129600,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 263,
                        "task_id": 568271
                    }
                ]
            }
```


From the above, we can see that for **Pickup Completed**, we need to complete a ‘capture_arrival_time’ sub-task and a ‘upload-photo’ sub-task. For **Pickup Failed**, we need to complete a ‘upload_photo’ sub-task.

<!--H6 not demoted to H7. -->


###### Sub-tasks for Dropoff


```
            "sub_tasks": {
                "dropoff_completed": [
                    {
                        "action": "upload_signature",
                        "company_id": 997,
                        "event": "dropoff_completed",
                        "id": 129602,
                        "meta": {},
                        "optional": false,
                        "sub_task_rule_id": 255,
                        "task_id": 568272
                    }
                ],
                "dropoff_failed": [
                    {
                        "action": "upload_signature",
                        "company_id": 997,
                        "event": "dropoff_failed",
                        "id": 129601,
                        "meta": {
                            "signature_message": "Singature on task report"
                        },
                        "optional": false,
                        "sub_task_rule_id": 280,
                        "task_id": 568272
                    }
                ]

```


From the above, we can see that for **Dropoff Completed**, we need to complete a ‘upload_signature’ sub-task. For **Dropoff Failed**, we need to complete a ‘upload_signature’ sub-task.


###### **Sample Payload for Pickup Completed**

To send bulk actions for a **Pickup Completed** task, we will need a payload with ‘capture_arrival_time’, ‘upload_photo’ sub-tasks actions (reference with the sub_task_rule_id), a **type:”pickup”, type:”complete”** task action, and a **mark_arrival_departure** action.


```
{
  "actions": [
    {
      "id": "_subtask_0_0",
      "params": {
        "sub_task_params": {
          "completion_data": {
            "arrival_time": "2021-05-12T06:50:28Z"
          },
          "completion_location": {
            "lat": 1.2778017,
            "lng": 103.8489061
          },
          "completion_time": "2021-05-12T06:50:44Z"
        },
        "sub_task_rule_id": 226
      },
      "tasks_ids": [
        567997
      ],
      "type": "complete_sub_task"
    },
    {
      "id": "_subtask_0_1",
      "params": {
        "sub_task_params": {
          "completion_location": {
            "lat": 1.2778017,
            "lng": 103.8489061
          },
          "completion_time": "2021-05-12T06:50:44Z",
          "photos": [
            "https://s3.ap-southeast-1.amazonaws.com/choongyong-yojee-umbrella/uploads-presigned/kcy-ds/20210512/20210512065050-1_pickup_1_completed_1620802244360.jpg"
          ]
        },
        "sub_task_rule_id": 253
      },
      "tasks_ids": [
        567997
      ],
      "type": "complete_sub_task"
    },
    {
      "id": "_task_0",
      "params": {
        "567997": {
          "completion_data": {},
          "completion_time": "2021-05-12T06:50:44Z",
          "descriptions": "",
          "location": {
            "lat": 1.2778017,
            "lng": 103.8489061
          },
          "role": "worker",
          "type": "pickup"
        }
      },
      "tasks_ids": [
        567997
      ],
      "type": "complete"
    },
    {
      "id": "arrival_departure_time_0",
      "params": {
        "arrival_time": "2021-05-12T06:38:37.240Z",
        "departure_time": "2021-05-12T06:50:44.323Z"
      },
      "tasks_ids": [
        567997
      ],
      "type": "mark_arrival_departure"
    }
  ]
}
```



###### **Sample Payload for Pickup Failed**

To send bulk actions for a **Pickup Failed** task, we will need a payload with ‘upload_photo’ sub-tasks actions (reference with the sub_task_rule_id), a **type:”pickup”, type:”report”** task action, and a **mark_arrival_departure** action.


```
{
  "actions": [
    {
      "id": "_subtask_0_0",
      "params": {
        "sub_task_params": {
          "completion_location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "completion_time": "2021-05-12T08:34:11Z",
          "photos": [
            "https://s3.ap-southeast-1.amazonaws.com/choongyong-yojee-umbrella/uploads-presigned/kcy-ds/20210512/20210512083413-1_pickup_1_completed_1620808451564.jpg"
          ]
        },
        "sub_task_rule_id": 263
      },
      "tasks_ids": [
        568015
      ],
      "type": "complete_sub_task"
    },
    {
      "id": "_task_0",
      "params": {
        "568015": {
          "completion_data": {},
          "completion_time": "2021-05-12T08:34:11Z",
          "descriptions": [
            "Cargo Damage"
          ],
          "location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "role": "worker",
          "type": "pickup"
        }
      },
      "tasks_ids": [
        568015
      ],
      "type": "report"
    },
    {
      "id": "arrival_departure_time_0",
      "params": {
        "arrival_time": "2021-05-12T08:32:30.020Z"
      },
      "tasks_ids": [
        568015
      ],
      "type": "mark_arrival_departure"
    }
  ]
}
```



###### **Sample Payload for Dropoff Completed**

To send bulk actions for a **Dropoff Completed** task, we will need a payload with ‘upload_signature’ sub-tasks actions (reference with the sub_task_rule_id), a **type:”dropoff”, type:”complete”** task action, and a **mark_arrival_departure** action.


```
{
  "actions": [
    {
      "id": "_subtask_0_0",
      "params": {
        "sub_task_params": {
          "completion_data": {
            "recipient_name": "Kcy"
          },
          "completion_location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "completion_time": "2021-05-12T07:54:39Z",
          "photos": [
            "https://s3.ap-southeast-1.amazonaws.com/choongyong-yojee-umbrella/uploads-presigned/kcy-ds/20210512/20210512075441-1_signature_1620806079457.jpg"
          ]
        },
        "sub_task_rule_id": 255
      },
      "tasks_ids": [
        567998
      ],
      "type": "complete_sub_task"
    },
    {
      "id": "_task_0",
      "params": {
        "567998": {
          "completion_data": {},
          "completion_time": "2021-05-12T07:54:39Z",
          "descriptions": "",
          "location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "role": "worker",
          "type": "dropoff"
        }
      },
      "tasks_ids": [
        567998
      ],
      "type": "complete"
    },
    {
      "id": "arrival_departure_time_0",
      "params": {
        "arrival_time": "2021-05-12T07:49:03.130Z",
        "departure_time": "2021-05-12T07:54:39.420Z"
      },
      "tasks_ids": [
        567998
      ],
      "type": "mark_arrival_departure"
    }
  ]
}
```



###### **Sample Payload for Dropoff Failed**

To send bulk actions for a **Dropoff Failed** task, we will need a payload with ‘upload_signaure’ sub-tasks actions (reference with the sub_task_rule_id), a **type:”dropoff”, type:”report”** task action, and a **mark_arrival_departure** action.


```
{
  "actions": [
    {
      "id": "_subtask_0_0",
      "params": {
        "sub_task_params": {
          "completion_data": {
            "recipient_name": ""
          },
          "completion_location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "completion_time": "2021-05-12T08:18:35Z",
          "photos": [
            "https://s3.ap-southeast-1.amazonaws.com/choongyong-yojee-umbrella/uploads-presigned/kcy-ds/20210512/20210512081838-1_signature_1620807515809.jpg"
          ]
        },
        "sub_task_rule_id": 280
      },
      "tasks_ids": [
        568018
      ],
      "type": "complete_sub_task"
    },
    {
      "id": "_task_0",
      "params": {
        "568018": {
          "completion_data": {},
          "completion_time": "2021-05-12T08:18:35Z",
          "descriptions": [
            "Wrong weather"
          ],
          "location": {
            "lat": 1.2777957,
            "lng": 103.848903
          },
          "role": "worker",
          "type": "dropoff"
        }
      },
      "tasks_ids": [
        568018
      ],
      "type": "report"
    },
    {
      "id": "arrival_departure_time_0",
      "params": {
        "arrival_time": "2021-05-12T08:16:36.361Z"
      },
      "tasks_ids": [
        568018
      ],
      "type": "mark_arrival_departure"
    }
  ]
}
```



##### 2.3.7 Get Driver Bulk Action Status


###### **Get Driver Bulk Action Status**

This call is to check the status of the bulk action called in **Driver Bulk Actions**.


<table>
  <tr>
   <td>GET
   </td>
   <td>[BASEURL]/api/v3/worker/tasks/bulk_actions/{batch_id}/status
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Authorization
   </td>
   <td>Y
   </td>
   <td>Bearer [jwt token]
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


<!--H6 not demoted to H7. -->


###### Request URL Parameters


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
   <td>batch_id
   </td>
   <td>Y
   </td>
   <td>The batch_id to query
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request GET '[BASEURL]/api/v3/worker/tasks/bulk_actions/TlhhcEJ4WjR2ZE09/status' \
--header 'COMPANY_SLUG: [SLUG]'

```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>Status query executed successfully
   </td>
  </tr>
  <tr>
   <td>401
   </td>
   <td>Unauthorized
   </td>
  </tr>
  <tr>
   <td>404
   </td>
   <td>Batch not found
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": {
        "processed": 3,
        "results": {
            "_subtask_0_0": {
                "success": true
            },
            "_task_0": {
                "success": true
            },
            "arrival_departure_time_0": {
                "success": true
            }
        },
        "status": "completed",
        "total": 3
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200 - With some errors, task is already completed


```
{
    "data": {
        "processed": 3,
        "results": {
            "_subtask_0_0": {
                "success": true
            },
            "_task_0": {
                "errors": [
                    {
                        "00000": [
                            "Tasks already completed"
                        ]
                    }
                ],
                "success": false
            },
            "arrival_departure_time_0": {
                "success": true
            }
        },
        "status": "completed",
        "total": 3
    }
}
```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 401


```
{
    "message": "Unauthorized"
}
```


<!--H6 not demoted to H7. -->


###### Sample Success Response: 404


```
{
    "data": {},
    "message": "Batch expired or not found!"
}
```



### <span style="text-decoration:underline;">3. Basic information on APIs</span>


#### Base URL

In this document we will use [BASEURL] to represent the base URL for the calls.

For **development and testing** purposes, please use [https://umbrella-staging.yojee.com](https://umbrella-staging.yojee.com).

The base URL for the **Production** API is [https://umbrella.yojee.com](https://umbrella.yojee.com).


#### Authentication

Most of the API calls will require the following parameters in the header:


<table>
  <tr>
   <td><strong>Parameter</strong>
   </td>
   <td><strong>Type</strong>
   </td>
  </tr>
  <tr>
   <td>company_slug
   </td>
   <td>string
   </td>
  </tr>
  <tr>
   <td>access_token
   </td>
   <td>string
   </td>
  </tr>
</table>



##### Company Slug

The Company Slug is a string to uniquely identify each instance of a customer’s company in Yojee. Each customer is assigned a slug which they will use as part of the authentication information.


##### Access token

A long-lived Access Token is generated for the Dispatcher account. This token will only change upon a change in the password of the Dispatcher account.

Obtain this information from the Yojee team working with you.

In this document we will use [SLUG] and [TOKEN] to represent the company_slug and access_token respectively.


##### JWT tokens

Another way of authentication is to use JWT tokens. To authenticate Drivers/Workers, we typically use JWT tokens.

To obtain the JWT Token for a Driver, use the **Verify Phone OTP** call.


```
After the Verify Phone OTP call succeeds, extract the access_token in the success payload, and include it in the Authorization Bearer Token calls that are to be made using the Driver's credentials.
```



###### **Verify Phone OTP**

Call this API to **delete** a Driver’s details


<table>
  <tr>
   <td>POST
   </td>
   <td>[BASEURL]/api/v3/public/verify_phone_otp
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Headers


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
   <td>Content-Type
   </td>
   <td>Y
   </td>
   <td>Use ‘application/json’
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Request Body parameters


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
   <td>phone
   </td>
   <td>Y
   </td>
   <td>phone number of the driver
   </td>
  </tr>
  <tr>
   <td>otp_code
   </td>
   <td>Y
   </td>
   <td>OTP for the driver. In our use case here, use the static otp set up during Driver creation
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/public/verify_phone_otp' \
--header 'Content-Type: application/json' \
--data-raw '{
    "phone": "+6522222226",
    "otp_code": "11223344"
}'
```


<!--H6 not demoted to H7. -->


###### Responses


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
   <td>OTP verified successfully
   </td>
  </tr>
  <tr>
   <td>422
   </td>
   <td>Unprocessable Entity - error
   </td>
  </tr>
</table>


<!--H6 not demoted to H7. -->


###### Sample Success Response: 200


```
{
    "data": {
        "kcy-ds": {
            "company_name": "KCY Downstream",
            "company_slug": "kcy-ds",
            "email": null,
            "jwt_tokens": {
                "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6Im1haW4iLCJwZXJtaXNzaW9uc191cGRhdGVkX2F0IjpudWxsLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlfaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIaoxNjIxMjM0MDc3LCJpYXQiOjE2MjEyMzMxNzcsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2c2R2MjZiZ3FoczM0YWlrMDAwYzE3IiwibmJmIjoxNjIxMjMzMTc3LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.S1dn7-h1_4hZxuIV10x5t27J1hzddatqj8qD3Ia7H74",
                "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NfdHlwZSI6InJlZnJlc2hfdG9rZW4iLCJhdWQiOiJKb2tlbiIsImNvbXBhbnlaaWQiOjk5NywiY29tcGFueV9zbHVnIjoia2N5LWRzIiwiZXhwIjoxNjUyMzM3MTc3LCJpYXQiOjE2MjEyMzMxNzcsImlzcyI6Ikpva2VuIiwianRpIjoiMnB2c2R2MjZidDE1czM0YWlrMDAwYzI3IiwibmJmIjoxNjIxMjMzMTc3LCJ1c2VyX3Byb2ZpbGVfaWQiOjk4NzZ9.W4rz5CUev_60nctMLyzvRuQbEFG2EA-yy5wPPQTC3sQ"
            },
            "name": "Driver KCY from API 4",
            "phone": "+6522222226",
            "user_profile_id": 9876
        }
    }
}

```


<!--H6 not demoted to H7. -->


###### Sample Error Response: 422


```
{
    "data": {
        "AC0002": [
            "OTP code expired or invalid"
        ]
    }
}
```
