# One Step: Submit Catalog Items API (Single API Call)

## Overview

This ServiceNow Scripted REST API enables a single API call to submit one or multiple catalog items on behalf of a user, streamlining the standard multi-step process.

Bots (a conversational bot) currently cannot orchestrate multiple sequential API calls required by ServiceNow (resolve user sys_id, add item to cart, submit order). This API handles all these steps internally to provide a seamless conversational experience.

---

## Why Use This API?

- Simplifies multiple API calls into one.
- Enables BOTs to submit catalog item requests directly.
- Supports multiple catalog items with varying variables.
- Handles dynamic catalog items without breaking.
- Improves end-user experience by removing manual steps.

---

## API Details

| Attribute         | Value                                                                  |
|-------------------|------------------------------------------------------------------------|
| **Method**        | `POST`                                                                |
| **Endpoint**      | `/api/x_custom/submit_catalog_items`                                 |
| **Authentication**| OAuth 2.0 a ServiceNow integration user having `rest_service` and `catalog_admin` roles |
| **Content-Type**  | `application/json`                                                     |

---

## Request Format

```json
{
  "requester": "user@example.com",
  "items": [
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_1",
      "quantity": 1,
      "variables": {
        "variable_name1": "value1",
        "variable_name2": "value2"
      }
    },
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_2",
      "variables": {
        "variable_nameA": "valueA"
      }
    }
  ]
}

```

requester: User’s email address to resolve sys_user.sys_id.

items: Array of catalog items to submit.

catalog_id: Sys ID of the catalog item.

quantity: Number of items to request (default 1).

variables: Key-value pairs matching catalog item variables.
```json
{
  "requester": "user@example.com",
  "results": [
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_1",
      "status": "submitted",
      "request_id": "REQ0012345"
    },
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_2",
      "status": "failed_to_add_to_cart"
    }
  ]
}

```

status can be:

submitted: Successfully placed order.

failed_to_add_to_cart: Could not add catalog item.

failed_to_submit_order: Cart order submission failed.

## Scripted REST API Script
```json

(function process(request, response) {
  try {
    var requestBody = request.body.data;
    var requesterEmail = requestBody.requester;
    var catalogItems = requestBody.items;

    var userGr = new GlideRecord('sys_user');
    userGr.addQuery('email', requesterEmail);
    userGr.query();

    if (!userGr.next()) {
      response.setStatus(400);
      response.setBody({ error: 'Invalid requester email' });
      return;
    }

    var userSysId = userGr.getValue('sys_id');
    var submittedRequests = [];

    for (var i = 0; i < catalogItems.length; i++) {
      var item = catalogItems[i];
      var cart = new Cart();
      cart.setRequester(userSysId);

      var cartItemId = cart.addItem(item.catalog_id, item.quantity || 1, item.variables);
      if (!cartItemId) {
        submittedRequests.push({
          catalog_id: item.catalog_id,
          status: 'failed_to_add_to_cart'
        });
        continue;
      }

      var cartOrderId = cart.placeOrder();
      if (!cartOrderId) {
        submittedRequests.push({
          catalog_id: item.catalog_id,
          status: 'failed_to_submit_order'
        });
        continue;
      }

      submittedRequests.push({
        catalog_id: item.catalog_id,
        status: 'submitted',
        request_id: cartOrderId
      });
    }

    response.setStatus(200);
    response.setBody({
      requester: requesterEmail,
      results: submittedRequests
    });

  } catch (err) {
    response.setStatus(500);
    response.setBody({ error: err.message });
  }
})(request, response);


```

## Example Request Payload
```json

{
  "requester": "john.doe@example.com",
  "items": [
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_1",
      "quantity": 1,
      "variables": {
        "request_type": "New Laptop",
        "urgency": "High"
      }
    },
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_2",
      "variables": {
        "comments": "Need external monitor",
        "approval_required": "true"
      }
    }
  ]
}

```
## Example Response
```json
CopyEdit
{
  "requester": "john.doe@example.com",
  "results": [
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_1",
      "status": "submitted",
      "request_id": "REQ0012345"
    },
    {
      "catalog_id": "CATALOG_ITEM_SYS_ID_2",
      "status": "submitted",
      "request_id": "REQ0012346"
    }
  ]
}

```

## Postman Test Case
•	Method: POST
•	URL: https://your-instance.service-now.com/api/x_custom/submit_catalog_items
•	Auth: Basic Auth
•	Headers:
o	Content-Type: application/json
•	Body: Raw JSON (see request payload above)

## Why It Supports Variable Catalog Items
•	Each catalog item may have different variable names and count.
•	The addItem() method in ServiceNow automatically maps provided variables.
•	Missing variables fall back to defaults defined in the item definition.
•	Excess variables are safely ignored.
•	This ensures flexibility and robustness for any number of items or variable structures.

## Error Handling
Condition	Status Code	Response Status
Invalid requester email	400	{ "error": "Invalid requester email" }
System or scripting error	500	{ "error": "error message" }
Item failed to add to cart	200	status: failed_to_add_to_cart
Cart order failed	200	status: failed_to_submit_order
Success	200	status: submitted

## Developer Tips
•	Use scoped API names (e.g., x_custom.submit_catalog_items) for isolation.
•	Consider validating variable keys against the sc_item_option_mtom table.
•	For traceability, log requester email, catalog item ID, and status.
•	Use gs.info() in dev/test environments to trace issues.

## Future Enhancements
•	✅ RITM numbers in response
•	✅ Retry logic for failed items
•	✅ Attachment support
•	✅ Validation of variable sets before submission
•	✅ Notify requesters via email or MS Teams


