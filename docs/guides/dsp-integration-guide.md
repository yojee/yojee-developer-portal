
**Downstream Partner Integration Solution Guide**
# Overview

Yojee’s customers use our platform to track their transport orders. In some use cases, they transfer orders to their downstream partners (DSPs for abbreviation) for further order execution. This is achieved through creating a separate instance in Yojee (we refer to this as a ‘slug’) and providing the DSP to access this slug. The DSP will proceed to assign drivers to the transferred orders and their drivers will make use of Yojee’s mobile app to provide the status updates. The status updates will be visible in both the DSP and the upstream partner (USP), Yojee customer’s slugs.

**Section 1** in this document will outline the current operations for the above flow in further detail.

In certain cases, the DSPs have their own Transport Management Systems, and would like to integrate with Yojee. To achieve this, there is a need to call specific Yojee APIs. **Section 2** in this document will first outline the integration flow, and describe the individual API calls needed. It is also important to understand the flow in Section 1 to be able to understand how the integration flow will work.


## Current Operations

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


### Order Transfer

#### Process Description

1. When Company A has an order it wishes to transfer to its DSP Company B, the Company A Dispatcher will go to Company A’s Yojee Dispatcher portal (in short, in **Co. A’s slug**) to select the order and initiate the transfer. The order will be in the status **Transferred - Pending** in Co. A’s slug.

    Company B Dispatcher will see an incoming order with options  to **Accept** or **Decline** in Company B’s Yojee Dispatcher portal (in short, in **Co. B’s slug**).

2. If Company B **declines** the transfer, the order will return as **unassigned** in Co’s A slug, and show as **Cancelled** in Co. B’s slug
3. If Company B **accepts** the transfer, the order will show as **Transferred - Unassigned** in Co. A’s slug and **Accepted** in Co. B’s slug. The order would now be transferred from Company A to Company B.


### Driver Management and Assignment

#### Process Description

After the order has been transferred to Co. B, it is the responsibility of Co. B to assign the order to a Driver for the execution of the order.

For Driver Management, the Co. B Dispatcher can go to Co. B’s slug to:

1. Create/Edit/Delete Drivers. Once drivers have been created in Co. B’s slug, the orders can be assigned.

For each order, there is a concept of tasks. In the most common use cases, an order will have a **pickup** task and a **dropoff** task. After the order has been transferred to Co. B, the tasks will also be visible in Co. B’s slug as **Unassigned**.

To assign the task(s) to a driver, in Co. B’s slug:

2. Assign the task(s) to a driver. The driver will see new tasks in **Incoming Assignments** on the mobile app. He can choose to Accept or Reject.
3. If the driver **rejects** the incoming assignment, the task will show as **Unassigned** again in Co. B’s slug
4. If the driver **accepts** the incoming assignment, the task will show as **Assigned** in Co. B’s slug. At the same time, the order will show as Transferred-Assigned in Co. A’s slug.


### Task Status Tracking  + POD updates
#### Process Description

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

## Integration Solution
### Integration Solution Overview

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


### Order Transfer

The API calls under **Order Transfer** in this solution will need to be called using Co. B’s Dispatcher credentials.

#### Retrieve incoming order details

Incoming transfer orders will be in Co. B’s slug as orders with **created** status. To retrieve the order information and the order item information for these orders, we will need to make 2 calls:
* Dispatcher Get Orders with status **created**.
* Dispatcher Get OrderItems by retrieving the **order_id** from the call above.


##### **Dispatcher Get Orders**


###### Sample Curl Command
```
curl --location -g --request GET '[BASEURL]/api/v3/dispatcher/orders?page_size=50&page=1&order=desc&status[]=created&from=2021-05-15T16:00:00.000Z&to=2021-05-16T16:00:00.000Z' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```

##### **Dispatcher Get OrderItems**

This API call will retrieve order items matching the criteria provided in the parameters. Use the order id from the result of the Dispatcher** Get Order** call.


###### Sample Curl Command
```
curl --location -g --request GET '[BASEURL]/api/v3/dispatcher/order_items?order_id=248602 \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


#### Accept the transfer order
##### **Dispatcher Accept Partner Transfer Order**

Call this API to **accept** the transfer order from upstream partner.



##### Sample Curl Command

```
curl --location --request PUT '[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/accept_order/O-8AWKFQYH168U' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'

```

#### Decline the transfer order

##### **Dispatcher Reject Partner Transfer Order**

###### Sample Curl Command


```
curl --location --request PUT '[BASEURL]/api/v3/dispatcher/partner_transfer/dispatcher/reject_order/O-8AWKFQYH168U' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'

```

### Driver Management and Assignment

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

<!-- theme: info -->
> ### Note
> See the section on Basic Information on APIs - Authentication at the end of this document for more information on authentication.

#### Driver Management - Create Driver
##### **Dispatcher Create Worker**

Call this API to **create** a new Driver the transfer order from upstream partner.

##### Sample Curl Command
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


#### Driver Management - Update Driver Information

##### **Dispatcher Update Worker**

Call this API to **update** a Driver’s details

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


#### Driver Management - Delete Driver Information
##### **Dispatcher Delete Worker**

Call this API to **delete** a Driver’s details

###### Sample Curl Command
```
curl --location --request DELETE '[BASEURL]/api/v3/dispatcher/workers/4231' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```


#### Get Tasks
##### **Dispatcher Get Tasks**

This API call will retrieve tasks under the slug.


<!-- theme:info -->
> ### Note
> Note: To retrieve tasks pending assignment, use the Request Query Parameters as listed below.


###### Sample Curl Command
```
curl --location -g --request GET '[BASEURL]/api/v3/dispatcher/tasks?from=2021-05-17T16:00:00.000Z&to=2021-05-18T16:00:00.000Z&task_group_states[]=unassigned' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]'
```



#### Assign Driver

Driver assignment is done by a background job. The first call is to send the parameters to the background job for execution, and the second call is to get the status of the background job to see the outcome.


##### **Dispatcher Assign Driver to tasks**

Call this API to assign a Driver to tasks


###### Sample Curl Command
```
curl --location --request POST '[BASEURL]/api/v3/dispatcher/tasks/quick_assign_bg' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN]' \
--header 'Content-Type: application/json' \
--data-raw '{"task_ids":[568259,568260],"worker_id":4233}'
```


###### Sample Curl Command
```
curl --location --request POST '[BASEURL]/api/v3/dispatcher/tasks/bg_status' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: [TOKEN] \
--header 'Content-Type: application/json' \
--data-raw '{"id":"c8cc52f50a7c4ef28249377a1645e113"}'

```


#### Driver Get Task Groups

##### **Driver Get TaskGroups**

This call will retrieve the task groups that have been assigned to the Driver, but pending Driver’s acceptance / rejection.

###### Sample Curl Command
```
curl --location -g --request GET '[BASEURL]/api/v3/worker/task_groups?status[]=assigned&status[]=broadcasted' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9' \
--data-raw ''
```


#### Driver Accepts Task

##### **Driver Accepts Task**

For the Driver to **accept** the task, call this API.



###### Sample Curl Command

```
curl --location --request PUT '[BASEURL]/api/v3/worker/task_groups/285642/accept' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9'
```


#### Driver Rejects Task
##### **Driver Rejects Task**

For the Driver to **reject** the task, call this API.



```
curl --location --request PUT '[BASEURL]/api/v3/worker/task_groups/285637/reject \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9'
```


### Task Status Tracking + POD updates

#### Driver Get Ongoing Tasks


##### **Driver Get Ongoing Tasks**

This call will list out the tasks that are currently on hand for the driver.

###### Sample Curl Command


```
curl --location --request GET '[BASEURL]/api/v3/worker/tasks/ongoing' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9'
```


#### Validate Completion

##### **Validate Completion**

Before updating status for a task, we need to call Validate Completion to ensure that the task can be acted upon.


###### Sample Curl Command
```
curl --location --request POST '[BASEURL]/api/v3/worker/tasks/validate_completion' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9' \
--header 'Content-Type: application/json' \
--data-raw '{
    "task_ids": [
        568271
    ]
}'
```


#### Driver Mark Arrival
##### **Driver Mark Arrival**

This call is to mark the arrival of the Drive at the location for a task



###### Sample Curl Command
```
curl --location --request POST '[BASEURL]/api/v3/worker/tasks/mark_arrival' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'ACCESS_TOKEN: NAN3acMWpyzLpJ7f2yNqt238e3aDxzfx0OjikfF2ORc=' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9' \
--header 'Content-Type: application/json' \
--data-raw '{
    "task_ids": [
        568271
    ],
    "arrival_time": "2021-05-17T15:16:37.240Z"
}'
```


#### Driver Generate Batch Upload Pre-signed URLs

In some sub-tasks, there is a need to upload POD/signature images to a AWS S3 Bucket. To perform this securely, a pre-signed URL is generated first, and the image is uploaded to AWS S3 via the methods described at [https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/PresignedUrlUploadObject.html).


##### **Generate Batch Upload Pre-signed URLs**

Generate AWS S3 pre-signed URL for uploading


###### Sample Curl Command
```
curl --location --request POST '[BASEURL]/api/v3/worker/generate_batch_upload_presigned_urls' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9' \
--header 'Content-Type: application/json' \
--data-raw '{
    "filenames": [
        "1_signature_1621265036.jpg"
    ]
}
'
```


```
Note: The step to upload the image to AWS S3 using the pre-signed URL is not part of the Yojee API calls. You will need to upload the file separately and the uploaded file will have the URL of https://[region].amazonaws.com/[bucket]/[name] (see "object" field above).

This URL of the image is used in the call to Driver Bulk Actions.
```



#### Driver Bulk Actions

Driver Bulk Actions is where the main status updates take place. For each **pickup** task, there is a **pickup_completed** and a **pickup_failed** array of sub-tasks. For each dropoff task, there is a **dropoff_completed** and a **dropoff_failed** array of sub-tasks.

Each of these arrays of sub-tasks lists the sub-tasks that need to be completed for the respective scenario of **pickup_completed**, **pickup_failed**, **dropoff_completed** and **dropoff_failed**, before the task of **pickup** / **dropoff** can be completed. See the section for **Driver Get Ongoing Task** for the sub-tasks retrieved.

The Bulk Actions call combines the updates of the sub-tasks into one call with a number of action elements. Each Bulk Actions call will return a batch_id which will be used in **Get Driver Bulk Action Status** to check if the bulk action has completed.


##### **Driver Bulk Actions**

This call is to send multiple actions based on the sub-tasks to be completed.


###### Sample Curl Command

This sample Curl command is to illustrate a complete call to **Driver Bulk Actions**. The breakdown of the individual action elements will be described later in this document.


```
curl --location --request PUT '[BASEURL]/api/v3/worker/tasks/bulk_actions' \
--header 'COMPANY_SLUG: [SLUG]' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9' \
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



##### Bulk Actions Payload Format

Each of the Bulk Action payloads includes a number of actions. Each action is either:

* A sub-task completion action
* A task completion/report action
* A mark_arrival_departure action

The list of sub-task completion actions are listed when calling **Driver Get Ongoing Tasks**. The respective sub-task from the sample output above is reproduced here (note that the sub-tasks for different configurations might differ)


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



#### Get Driver Bulk Action Status


##### **Get Driver Bulk Action Status**

This call is to check the status of the bulk action called in **Driver Bulk Actions**.




###### Sample Curl Command


```
curl --location --request GET '[BASEURL]/api/v3/worker/tasks/bulk_actions/TlhhcEJ4WjR2ZE09/status' \
--header 'COMPANY_SLUG: [SLUG]'

```


#### JWT tokens

Another way of authentication is to use JWT tokens. To authenticate Drivers/Workers, we typically use JWT tokens.

To obtain the JWT Token for a Driver, use the **Verify Phone OTP** call.


```
After the Verify Phone OTP call succeeds, extract the access_token in the success payload, and include it in the Authorization Bearer Token calls that are to be made using the Driver's credentials.
```



##### **Verify Phone OTP**

Call this API to **get** a Driver’s JWT Token



##### Sample Curl Command


```
curl --location --request POST '[BASEURL]/api/v3/public/verify_phone_otp' \
--header 'Content-Type: application/json' \
--data-raw '{
    "phone": "+6522222226",
    "otp_code": "11223344"
}'
```