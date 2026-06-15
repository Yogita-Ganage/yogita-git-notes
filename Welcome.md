Hi all, while working on RDM SharePoint list ingestion, we normally test using a duplicate/test copy of the original list first. However, there can still be situations where data has already been inserted into the original SharePoint list and we need to clear the existing records before re-running the corrected ingestion.

When the list contains a large number of records, the previous workaround was to duplicate the original list again, rename the old list, and use the new copy for the corrected run.

To avoid this, I’ve created and tested a reusable manual Power Automate cleanup flow. It clears only the items/records from the selected SharePoint list and does not delete the list itself or change its structure, columns, lookup settings, permissions, or formatting.

The flow is dynamic, takes Site URL and List Name at run time, and includes a `DELETE` confirmation step for safety. I’ll document the usage steps and safety checks as an SOP.
