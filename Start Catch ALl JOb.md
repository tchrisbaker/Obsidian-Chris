```ad-note
Start Catch-All Job


The Sledgehammer project has a catch-all job which will process any Staging Records that failed
```

```Java

for (integer i = 0; i <= 59; i += 1) {
    String sch1 = '0 ' + i + ' * * * ?';
    SH_ScheduleStagingDataProcessing schdp = new SH_ScheduleStagingDataProcessing();
    system.schedule('SH_ScheduleStagingDataProcessing -- ' + i + ' min', sch1, schdp);
}
```


#Salesforce #Apex #Jobs #Sledgehammer 


[[Abort Catch all job]]