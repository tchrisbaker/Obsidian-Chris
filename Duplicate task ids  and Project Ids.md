

I found that we received 2 API calls for Task (same Task Id is on both) and they both happened at the same exact moment down to second

[https://dfsco--sledge.lightning.force.com/lightning/r/SH\_Task\_Staging\_\_c/a4X2h000000AtP7EAK/view](https://dfsco--sledge.lightning.force.com/lightning/r/SH_Task_Staging__c/a4X2h000000AtP7EAK/view "https://dfsco--sledge.lightning.force.com/lightning/r/sh_task_staging__c/a4x2h000000atp7eak/view")

[https://dfsco--sledge.lightning.force.com/lightning/r/SH\_Task\_Staging\_\_c/a4X2h000000AtPCEA0/view](https://dfsco--sledge.lightning.force.com/lightning/r/SH_Task_Staging__c/a4X2h000000AtPCEA0/view "https://dfsco--sledge.lightning.force.com/lightning/r/sh_task_staging__c/a4x2h000000atpcea0/view")

The time have the same Created Date in Salesforce database which is this 2021-06-16T 12:41:13

The Source Timestamp is the same too for both records - 2021-06-16 12:41:13

Because of this, the Salesforce system executed them at the same exact time. We are doing an upsert so normally one of these would insert the other would update. But since they both came at the same moment, they both inserted. And because they both inserted, we have 2 Tasks with the same TaskId

I was curious if that would be a common occurrence that we would get more than 1 request at the same moment for the same record?



\[4:42 PM\] Joshua Kendall

Henrikas Kapaciauskas Andrius Juozapaitis

It appears we have two different issues, tho might be related as it might be a source\_timestamp issue.

Issue 1)

Task is sending requests w same sourcetimestamp but different message ID and data. It appears that when the first update is made to a task, it is sending both the create task request and the field updated request at the same time. I checked and it does not appear to be a field specific issue so seems to just be related to the first update.

Example, two request sent at same time but have different data. https://dfsco--sledge.my.salesforce.com/a4X2h000000Atpa - does not have document format (believe this is the task create request) https://dfsco--sledge.my.salesforce.com/a4X2h000000Atpf - has document format (believe this is the first update to the task)

Issue 2)

For Project, this seems related to both project/sections being sent w same sourcetimestamp.  It seems to be that New AD is sending multiple requests at the exact same time, even from multiple objects (ie. project and sections with same sourcetimestamp). 

https://dfsco--sledge.my.salesforce.com/a4U2h0000004fpH?srPos=13&srKp=a4U  - Section

https://dfsco--sledge.my.salesforce.com/a4Q2h000000IVC6?srPos=1&srKp=a4Q - Project

I would assume that it is not possible to create a section before or at same time a project is created. 

Couple questions:

-   Is this correct, a project and section can be created at the same time in New AD? I wouldn't think sections would be sent same time as project create nor have same sourcetimestamp.

-   Can you confirm what value is being used for sourcetimestamp?  It should be the time the action occurred in New AD, not the time the request was sent to SF.

-   Are requests being sent after each field update? I thought there was a Save button that would be used to send all updates at once, in case of multiples fields being updated on record.  Or are these being queued to be sent to SF?


---
\[4:49 PM\] Andrius Juozapaitis

IIRC, when the project is being created, section entities are created first (which triggers the initial request to SF with section update). Then, once the project entity is created, an Upsert Project action is sent, and SF compares the source timestamp we send with the project with the last update timestamp in SF (which happened when the dummy project entry was created). Since those actions execute in parallel in New AD, SF refuses to process the new project staging request. 

<https://teams.microsoft.com/l/message/19:bf5c904aedff431a89da5c55091d039a@thread.tacv2/1623966584205?tenantId=64ebfaf9-be45-43c2-9e9c-fcb060bf234d&amp;groupId=71e86d43-01fd-490c-8eb4-df52bffdbaaa&amp;parentMessageId=1623872121189&amp;teamName=New AD (Sledgehammer) Engineering&amp;channelName=Salesforce Integration&amp;createdTime=1623966584205>


\[Yesterday 9:50 PM\] Joshua Kendall

Andrius Juozapaitis

Thanks for explaining how it works from the New AD side.  We were not aware that Project and Section would be generated at same time in New AD and assumed Project would be created first/sent with earlier timestamp. 

We are setting up SF to accept the messages in any order from New AD as we want to avoid delays in processing to get data into SF quickly.  We are using the sourcetimestamp as a means to manage the message ordering.  And in case of records needing a parent in order to create, we will create a placeholder until the real record request is sent or processed to SF. 

So in this scenario, the section request was received and processed first (before the project request).  Our code determined that the project did not exist at time of attempting to create the section, so it creates the placeholder project in order to be able to create the section (otherwise we would have to fail or re-cycle the section request for processing until project request is processed).  The placeholder project record will be created with the same timestamp as that on the section (and in this scenario, same as the real project request). However, we did not anticipate that the section would be sent and processed with placeholder project at the same time the real project request was being processed, all with same timestamps.  This is what caused the system to store the external ID on more than one record, but we will fix this by marking the external ID fields as unique.

One thought I had, when a project is being created....would it be possible to add 1 millisecond or 1 second to the section timestamps so they are different than the timestamps on the project?  (ie. OffsetDateTime.now() + 1 or something along those lines).  This would help resolve the issue for ordering because currently the Project and Section have the exact same timestamp down to milisecond when a project is created.  If section is slightly after, when this issue arises again with unique external ID being enforced...the placeholder record would process, the real record fail first attempt but then execute second time because it's timestamp is before that on the Section (and Project placeholder) and the correct data would be on the project. (this proposal would only be for project create)

Edited

​

\[Yesterday 9:52 PM\] Joshua Kendall

I have availability in my morning tomorrow (830 or 9am CST), if you would like to try and connect to discuss this at end of your day.  Otherwise we can bring to our weekly Tues call.

​

\[9:26 AM\] Nizar Ghaoui

Hi all, we have looked into this from different perspective, the best that it is fixed at SF end as it should handle the async requests when created the placeholder section and then updating with the project.

<https://teams.microsoft.com/l/message/19:bf5c904aedff431a89da5c55091d039a@thread.tacv2/1623984640480?tenantId=64ebfaf9-be45-43c2-9e9c-fcb060bf234d&amp;groupId=71e86d43-01fd-490c-8eb4-df52bffdbaaa&amp;parentMessageId=1623872121189&amp;teamName=New AD (Sledgehammer) Engineering&amp;channelName=Salesforce Integration&amp;createdTime=1623984640480>


#Sledgehammer #TaskIntegration 