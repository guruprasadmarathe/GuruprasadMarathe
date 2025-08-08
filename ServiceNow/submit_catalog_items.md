# Submit Catalog Items API

## Overview

This ServiceNow Scripted REST API enables a single API call to submit one or multiple catalog items on behalf of a user, streamlining the standard multi-step process.

Charlie (a conversational bot) currently cannot orchestrate multiple sequential API calls required by ServiceNow (resolve user sys_id, add item to cart, submit order). This API handles all these steps internally to provide a seamless conversational experience.

---

## Why Use This API?

- Simplifies multiple API calls into one.
- Enables Charlie to submit catalog item requests directly.
- Supports multiple catalog items with varying variables.
- Handles dynamic catalog items without breaking.
- Improves end-user experience by removing manual steps.

---

## API Details

| Attribute         | Value                                                                  |
|-------------------|------------------------------------------------------------------------|
| **Method**        | `POST`                                                                |
| **Endpoint**      | `/api/x_custom/submit_catalog_items`                                 |
| **Authentication**| Basic Auth with a ServiceNow integration user having `rest_service` and `catalog_admin` roles |
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

---

requester: User’s email address to resolve sys_user.sys_id.

items: Array of catalog items to submit.

catalog_id: Sys ID of the catalog item.

quantity: Number of items to request (default 1).

variables: Key-value pairs matching catalog item variables.

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


status can be:

submitted: Successfully placed order.

failed_to_add_to_cart: Could not add catalog item.

failed_to_submit_order: Cart order submission failed.

Scripted REST API Script


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



Testing Instructions
Using Postman or any REST client:
Set POST method to https://<your_instance>.service-now.com/api/x_custom/submit_catalog_items

Use Basic Auth with a valid ServiceNow integration user.

Set Content-Type header to application/json.

Paste request JSON (see above) in the request body.

Send the request.

Review the response for success or error statuses.

Notes for Developers
The API automatically resolves the user email to sys_id.

The Cart API handles variable mapping; unsupported variables are ignored.

You can submit multiple items at once, each with its own variables.

Handle error statuses gracefully in your integration.

Log request and response for troubleshooting.

Future Enhancements
Add OAuth 2.0 support.

Return RITM (Requested Item) numbers.

Validate variable keys before adding to cart.

Retry failed submissions automatically.

Support attachments for catalog items.