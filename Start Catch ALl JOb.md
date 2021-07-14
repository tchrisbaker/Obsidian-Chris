for (integer i = 0; i <= 59; i += 1) {
    String sch1 = '0 ' + i + ' * * * ?';
    SH_ScheduleStagingDataProcessing schdp = new SH_ScheduleStagingDataProcessing();
    system.schedule('SH_ScheduleStagingDataProcessing -- ' + i + ' min', sch1, schdp);
}

[[Salesforce]]