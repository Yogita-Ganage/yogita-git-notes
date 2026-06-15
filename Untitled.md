# Manual SharePoint List Item Cleanup Flow - SOP

## Document Control

| Field         | Details                                          |
| ------------- | ------------------------------------------------ |
| Document Name | Manual SharePoint List Item Cleanup Flow - SOP   |
| Flow Name     | Manual - Delete All Items From SharePoint List   |
| Owner         | Data Engineering / [Your Name]                   |
| Created Date  | 15 June 2026                                     |
| Version       | 1.0                                              |
| Status        | Draft / Ready for Review                         |
| Tested On     | RDM - Form Question Test                         |
| Test Result   | Successfully deleted 1,340 SharePoint list items |
| Tool Used     | Microsoft Power Automate                         |
| Trigger Type  | Manual trigger                                   |

---

## 1. Purpose

This document explains how to use the reusable manual Power Automate flow created to clear existing items from an RDM SharePoint list.

The flow is intended for scenarios where incorrect or test records have already been inserted into an original RDM SharePoint list and the corrected ingestion needs to be re-run.

The flow only deletes list items/records. It does not delete the SharePoint list itself and does not change the list structure, columns, lookup settings, permissions, views, or formatting.

---

## 2. Background

While working on RDM SharePoint list ingestion, we normally test using a duplicate or test copy of the original SharePoint list first.

However, we still come across scenarios where data has already been inserted into the original SharePoint list and existing records need to be cleared before re-running the corrected ingestion.

Previously, when the list contained a large number of records, the workaround was to duplicate the original list again, rename the old list, and use the new copy for the corrected run.

This cleanup flow avoids that workaround by safely clearing only the existing items from the selected SharePoint list.

---

## 3. Scope

This SOP applies to RDM SharePoint lists where existing list items need to be cleared before re-running corrected ingestion.

This flow is designed to delete all items from the selected SharePoint list.

It does not delete:

* The SharePoint list
* Columns
* Lookup configuration
* Views
* Permissions
* Formatting
* List settings

---

## 4. Flow Summary

| Area          | Details                                                            |
| ------------- | ------------------------------------------------------------------ |
| Flow Type     | Instant cloud flow                                                 |
| Trigger       | Manually trigger a flow                                            |
| Schedule      | None                                                               |
| Main Purpose  | Delete all items from a selected SharePoint list                   |
| Input Method  | User enters Site URL, List Name, and confirmation text at run time |
| Safety Check  | Flow only proceeds when ConfirmDelete = DELETE                     |
| Delete Method | Deletes each item using SharePoint internal item ID                |

---

## 5. Flow Inputs

The flow requires the following inputs at run time.

### 5.1 SiteUrl

The SharePoint site URL.

Example:

`https://vitahealthgroup.sharepoint.com/sites/DataAnalyticsPhaseI`

### 5.2 ListName

The exact SharePoint list name to clear.

Example:

`RDM - Form Question Test`

### 5.3 ConfirmDelete

Safety confirmation input.

The flow will only proceed if the value entered is exactly:

`DELETE`

If any other value is entered, the flow will stop and no records will be deleted.

---

## 6. Flow Logic

The flow follows this structure:

1. Manual trigger receives `SiteUrl`, `ListName`, and `ConfirmDelete`.
2. Condition checks whether `ConfirmDelete` is equal to `DELETE`.
3. If the confirmation is not valid, the flow terminates.
4. If the confirmation is valid, the flow retrieves items from the selected SharePoint list.
5. The flow loops through each returned item.
6. Each item is deleted using the SharePoint internal item ID.

Flow structure:

`Manual Trigger → Condition → Get items → Apply to each → Delete item`

---

## 7. Power Automate Configuration

### 7.1 Manual Trigger

Inputs configured:

* `SiteUrl`
* `ListName`
* `ConfirmDelete`

[Screenshot: Manual trigger inputs]

---

### 7.2 Condition

Condition:

`ConfirmDelete is equal to DELETE`

If false, the flow terminates.

If true, the flow continues to retrieve and delete list items.

[Screenshot: Condition step]

---

### 7.3 Get Items

Action used:

`SharePoint - Get items`

Configuration:

| Field        | Value    |
| ------------ | -------- |
| Site Address | SiteUrl  |
| List Name    | ListName |
| Top Count    | 5000     |
| Pagination   | On       |
| Threshold    | 100000   |

This step retrieves the items from the selected SharePoint list.

[Screenshot: Get items configuration]

---

### 7.4 Apply to Each

The `Apply to each` action loops through the output from `Get items`.

Input used:

`body/value`

This represents the list of SharePoint items returned by the `Get items` action.

[Screenshot: Apply to each configuration]

---

### 7.5 Delete Item

Action used:

`SharePoint - Delete item`

Configuration:

| Field        | Value                       |
| ------------ | --------------------------- |
| Site Address | SiteUrl                     |
| List Name    | ListName                    |
| Id           | SharePoint internal item ID |

Expression used for Id:

`items('Apply_to_each')?['ID']`

This deletes the current SharePoint item inside the loop.

[Screenshot: Delete item configuration]

---

## 8. Important Configuration Notes

Pagination must be enabled in the `Get items` action.

Recommended configuration:

* Top Count: `5000`
* Pagination: `On`
* Threshold: `100000`

If Pagination is off, Power Automate may retrieve only a limited number of records. In that case, the flow may appear successful, but some records may remain in the SharePoint list.

---

## 9. Why SharePoint Internal Item ID Is Used

The `Delete item` action requires the SharePoint internal item ID.

This is the unique ID automatically assigned by SharePoint to each list item. The flow does not use business columns, lookup values, source IDs, or source names to delete records.

This makes the flow reusable across different RDM SharePoint lists, even when the list columns are different.

Lookup columns do not need separate handling because the full SharePoint list item is deleted using its internal ID.

---

## 10. Permissions

The SharePoint connection used by the flow must have permission to read and delete items from the selected SharePoint list.

Required access:

* Read access for `Get items`
* Delete/Edit access for `Delete item`

If the flow fails with `Forbidden` or `UnauthorizedAccessException`, check the SharePoint connection account and confirm it has permission on the target list.

Both `Get items` and `Delete item` should use a SharePoint connection that has the required permissions on the target list.

---

## 11. How to Use the Flow

1. Open the manual cleanup flow in Power Automate.
2. Click Run.
3. Enter the SharePoint `SiteUrl`.
4. Enter the exact `ListName`.
5. Enter `DELETE` in the `ConfirmDelete` field.
6. Run the flow.
7. Wait until the flow completes successfully.
8. Refresh the SharePoint list and confirm that the items have been deleted.
9. Re-run the corrected ingestion if required.

[Screenshot: Run flow input screen]

---

## 12. Validation Before Running

Before running the flow:

* Confirm that the correct SharePoint site URL is being used.
* Confirm that the correct SharePoint list name is being used.
* Confirm that the list is the intended list to clear.
* Check the current item count in the SharePoint list.
* Confirm that the cleanup is required before re-running ingestion.
* Confirm that the connection account has permission to delete list items.

---

## 13. Validation After Running

After the flow completes:

* Confirm that the flow status is `Succeeded`.
* Check that `Get items` retrieved the expected number of records.
* Check that `Apply to each` completed successfully.
* Refresh the SharePoint list.
* Confirm that the existing items have been cleared.
* Re-run the corrected ingestion if required.

---

## 14. When to Use This Flow

Use this flow when:

* Incorrect records have already been inserted into an RDM SharePoint list.
* Existing records need to be cleared before re-running corrected ingestion.
* The SharePoint list structure should be preserved.
* Creating a duplicate list and renaming the old list should be avoided.
* The list contains a large number of records and manual deletion would be inefficient.

---

## 15. When Not to Use This Flow

Do not use this flow when:

* The SharePoint list itself needs to be deleted.
* Only specific filtered records need to be deleted.
* The target list name is not confirmed.
* The flow connection account does not have delete permission.
* Business approval is required before clearing the list.
* There is uncertainty about whether the records should be removed.

---

## 16. Safety Notes

This flow is manual only and should not be scheduled.

The `DELETE` confirmation step is included to reduce the risk of accidental deletion.

The flow should be tested on a test SharePoint list before being used on an original or production list.

Always confirm the target list name and item count before running the cleanup.

---

## 17. Test Evidence

Test SharePoint list:

`RDM - Form Question Test`

Test result:

The flow successfully retrieved and deleted 1,340 SharePoint list items.

[Screenshot: Successful flow run]

[Screenshot: SharePoint list after cleanup]

---

## 18. Known Notes and Limitations

If Pagination is off, Power Automate may retrieve only a limited number of records. In that case, the flow may appear successful but some records may remain in the SharePoint list.

For large lists, the flow may take several minutes because each item is deleted individually inside an `Apply to each` loop.

The flow currently clears all items from the selected SharePoint list. It is not designed for filtered or partial deletion.

---

## 19. Future Considerations

Possible future improvements:

* Add an optional filter condition for partial cleanup.
* Restrict the flow to specific approved users.
* Replace free-text ListName input with a controlled list selection if required.
* Add email or Teams notification after successful cleanup.
* Add logging of run details, including list name, deleted count, run user, and timestamp.

---

## 20. Summary

This manual Power Automate cleanup flow provides a safer and reusable way to clear records from an RDM SharePoint list without deleting or recreating the list.

It helps avoid the previous workaround of duplicating the original list, renaming the old list, and re-running ingestion into a new copy.

The flow preserves the SharePoint list structure and only removes the existing list items, allowing the corrected ingestion to be re-run against the same list.
