   

Section Staging Processing

If Is\_Deleted = true

-   Flag Is\_Deleted = true on Section record
-   Check for task sections where not cancelled and not deleted and 3 statuses are blank

-   Hard delete (delete task section record)



Task Staging Processing

-   If Task Section Action = Remove

-   Check if task sections Are not cancelled and not deleted and 3 statuses are blank

-   Hard Delete (delete the Task Section record)

-   Else

-   Soft delete (Remove\_From\_Task\_\_c = true)

#TaskIntegration #Sledgehammer 