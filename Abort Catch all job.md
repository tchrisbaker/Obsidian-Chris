for (CronTrigger c : [SELECT id FROM Crontrigger WHERE CronJobDetail.JobType = '7' AND CronJobDetail.Name LIKE 'SH_ScheduleStagingDataProcessing%']) {
    System.abortJob(c.Id);
}
