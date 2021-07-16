```ad-note
Abort Catch-All Job


The Sledgehammer project has a catch-all job which will process any Staging Records that failed
```

```Java
for (CronTrigger c : [SELECT id FROM Crontrigger WHERE CronJobDetail.JobType = '7' AND CronJobDetail.Name LIKE 'SH_ScheduleStagingDataProcessing%']) {
    System.abortJob(c.Id);
}
```

#Salesforce #Jobs #Sledgehammer #Apex

[[Start Catch All Job]]