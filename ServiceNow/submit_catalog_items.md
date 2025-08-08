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
