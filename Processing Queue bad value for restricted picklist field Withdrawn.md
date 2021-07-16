In Stage, We got an error in the object ErrorMessage__c that said 

```ad-error
Processing Queue: bad value for restricted picklist field: Withdrawn
```


The Contact Staging record with the error is https://dfsco--stage.lightning.force.com/lightning/r/SH_Contact_Staging__c/a4R63000000PTKaEAO/view

Once the status is "Withdrawn" it should never get updated again so this is weird

I think the issue is here at line 219 in SH_ProcessContactStagingTable - it's copying the current value of "Withdrawn" to Completed In Queue
```Java
else if (withdrawnAssetUserKeys.contains(assetUserKey)) {  
 stagingRecord.Completed_In_Queue__c = stagingRecord.Status__c;  
    stagingRecord.Status__c = SH_Constants.STAGING_STATUS_WITHDRAWN;  
    stagingRecord.Error_Message__c = 'The Source Timestamp is older than the Asset User\'s Last Updated Date';  
}
```

```ad-info
Here is the logs from when the job ran
```

```
51.0 APEX_CODE,DEBUG;APEX_PROFILING,INFO;CALLOUT,INFO;DB,INFO;NBA,INFO;SYSTEM,DEBUG;VALIDATION,INFO;VISUALFORCE,INFO;WAVE,INFO;WORKFLOW,INFO
10:58:01.5 (5565055)|USER_INFO|[EXTERNAL]|005U0000001LhxC|automated.user@rrd.com.stage|(GMT-05:00) Central Daylight Time (America/Chicago)|GMT-05:00
10:58:01.5 (5622169)|EXECUTION_STARTED
10:58:01.5 (5631058)|CODE_UNIT_STARTED|[EXTERNAL]|01p6300000DZyZR|SH_ScheduleStagingDataProcessing -- 58 min
10:58:01.5 (23808209)|SYSTEM_MODE_ENTER|false
10:58:01.5 (24438982)|SYSTEM_MODE_EXIT|false
10:58:01.5 (26851233)|SOQL_EXECUTE_BEGIN|[21]|Aggregations:0|SELECT Id, Action__c, SF_Asset_ID__c, SH_Site_Status__c, Requested_By__c, Status__c, Source_Timestamp__c, SH_Site_Name__c FROM SH_Account_Staging__c WHERE Status__c = :tmpVar1
10:58:01.5 (34533756)|SOQL_EXECUTE_END|[21]|Rows:0
10:58:01.5 (38365124)|SOQL_EXECUTE_BEGIN|[132]|Aggregations:0|SELECT Id, SH_Site_Status_Last_Updated_Date__c, SH_Site_Last_Updated_Date__c FROM Asset WHERE Id IN :tmpVar1
10:58:01.5 (40633474)|SOQL_EXECUTE_END|[132]|Rows:0
10:58:01.5 (42011650)|SOQL_EXECUTE_BEGIN|[70]|Aggregations:0|SELECT Id FROM Asset WHERE Id = :tmpVar1
10:58:01.5 (43608591)|SOQL_EXECUTE_END|[70]|Rows:0
10:58:01.5 (51592303)|SOQL_EXECUTE_BEGIN|[19]|Aggregations:0|SELECT Id, Action__c, SF_Account_ID__c, SF_Contact_ID__c, SF_Asset_ID__c, SH_Role__c, Requested_By__c, Status__c, Source_Timestamp__c, SH_Site_User_Status__c FROM SH_Contact_Staging__c WHERE Status__c = :tmpVar1
10:58:01.5 (58813874)|SOQL_EXECUTE_END|[19]|Rows:1
10:58:01.5 (62070258)|USER_DEBUG|[48]|DEBUG|stagingRecordsList [ {
  "attributes" : {
    "type" : "SH_Contact_Staging__c",
    "url" : "/services/data/v52.0/sobjects/SH_Contact_Staging__c/a4R63000000PTKaEAO"
  },
  "Id" : "a4R63000000PTKaEAO",
  "Action__c" : "Update Status",
  "SF_Contact_ID__c" : "0031O00003S6xroQAB",
  "SF_Asset_ID__c" : "02i63000006XZALAA4",
  "Requested_By__c" : "Rugile Rugile Rugile Rugile Rugile Rugile",
  "Status__c" : "Scheduled Queue",
  "Source_Timestamp__c" : "2021-04-12T20:54:34.000+0000",
  "SH_Site_User_Status__c" : "Active"
} ]
10:58:01.5 (64178311)|SOQL_EXECUTE_BEGIN|[362]|Aggregations:0|SELECT Asset__c, Contact__c, SH_Site_User_Status_Last_Updated_Date__c, SH_Site_User_Role_Last_Update_Date__c FROM Asset_User__c WHERE (Asset__c = :tmpVar1 AND Contact__c = :tmpVar2)
10:58:01.5 (69821885)|SOQL_EXECUTE_END|[362]|Rows:1
10:58:01.5 (75521902)|USER_DEBUG|[53]|DEBUG|lastModifiedByAUKey {
  "02i63000006XZALAA4-0031O00003S6xroQAB" : "2021-04-12T20:54:35.000Z"
}
10:58:01.5 (76250439)|USER_DEBUG|[323]|DEBUG|stagingToAssetUserKey {
  "a4R63000000PTKaEAO" : "02i63000006XZALAA4-0031O00003S6xroQAB"
}
10:58:01.5 (76416946)|USER_DEBUG|[329]|DEBUG|lastModifiedByAssetIds containsKey 02i63000006XZALAA4-0031O00003S6xroQAB
10:58:01.5 (77436210)|USER_DEBUG|[58]|DEBUG|stagingRecordsFiltered [ {
  "attributes" : {
    "type" : "SH_Contact_Staging__c",
    "url" : "/services/data/v52.0/sobjects/SH_Contact_Staging__c/a4R63000000PTKaEAO"
  },
  "Id" : "a4R63000000PTKaEAO",
  "Action__c" : "Update Status",
  "SF_Contact_ID__c" : "0031O00003S6xroQAB",
  "SF_Asset_ID__c" : "02i63000006XZALAA4",
  "Requested_By__c" : "Rugile Rugile Rugile Rugile Rugile Rugile",
  "Status__c" : "Withdrawn",
  "Source_Timestamp__c" : "2021-04-12T20:54:34.000+0000",
  "SH_Site_User_Status__c" : "Active",
  "Completed_In_Queue__c" : "Scheduled Queue",
  "Error_Message__c" : "The Source Timestamp is older than the Asset User's Last Updated Date"
} ]
10:58:01.5 (78462687)|SOQL_EXECUTE_BEGIN|[73]|Aggregations:0|SELECT Contact__c, Asset__c FROM Asset_User__c WHERE (Asset__c = :tmpVar1 AND Contact__c = :tmpVar2)
10:58:01.5 (83573524)|SOQL_EXECUTE_END|[73]|Rows:1
10:58:01.5 (85728258)|USER_DEBUG|[81]|DEBUG|-*-assetUserMap {
  "02i63000006XZALAA4-0031O00003S6xroQAB" : {
    "attributes" : {
      "type" : "Asset_User__c",
      "url" : "/services/data/v52.0/sobjects/Asset_User__c/a4S63000000LVi9EAG"
    },
    "Contact__c" : "0031O00003S6xroQAB",
    "Asset__c" : "02i63000006XZALAA4",
    "Id" : "a4S63000000LVi9EAG"
  }
}
10:58:01.5 (86473958)|USER_DEBUG|[117]|DEBUG|withdrawnAssetUserKeys.contains(assetUserKey)   true
10:58:01.5 (87433002)|SOQL_EXECUTE_BEGIN|[199]|Aggregations:0|SELECT Id, Asset__c, Contact__c FROM Asset_User__c WHERE Id = :tmpVar1
10:58:01.5 (89304775)|SOQL_EXECUTE_END|[199]|Rows:0
10:58:01.5 (90598674)|DML_BEGIN|[236]|Op:Update|Type:SH_Contact_Staging__c|Rows:1
10:58:01.5 (102619499)|DML_END|[236]
10:58:01.5 (119373725)|USER_DEBUG|[165]|DEBUG|INVALID_OR_NULL_FOR_RESTRICTED_PICKLIST: Processing Queue: bad value for restricted picklist field: Withdrawn Fields that affected this error: (Completed_In_Queue__c)
10:58:01.5 (126542934)|USER_DEBUG|[28]|DEBUG| ErrorLogger.logMessages errorMessages: [{"userId":null,"stackTrace":null,"source":"SH_ProcessContactStagingTable.processContactStagingData","payload":"(Completed_In_Queue__c)","errorType":"GENERAL","errorMessage":"Processing Queue: bad value for restricted picklist field: Withdrawn","emailSubject":null,"emailBody":null}]
10:58:01.5 (127260896)|USER_DEBUG|[77]|DEBUG| ErrorLogger.flush logs: [{"userId":null,"stackTrace":null,"source":"SH_ProcessContactStagingTable.processContactStagingData","payload":"(Completed_In_Queue__c)","errorType":"GENERAL","errorMessage":"Processing Queue: bad value for restricted picklist field: Withdrawn","emailSubject":null,"emailBody":null}]
10:58:01.5 (130247572)|DML_BEGIN|[85]|Op:Insert|Type:ErrorMessage__c|Rows:1
10:58:01.5 (197285752)|DML_END|[85]
10:58:01.5 (197935948)|USER_DEBUG|[89]|DEBUG|ErrorLogger.flush publishErrorEvents: []
10:58:01.5 (204811471)|SOQL_EXECUTE_BEGIN|[20]|Aggregations:0|SELECT Id, SH_Project_ID__c, Workbook_Linking__c, Form_Type__c, Project_Created_Date__c, Project_Created_By__c, Self_Filed__c, Project_Filing_Status__c, Project_Name__c, SF_Asset_ID__c, SH_Project_URL__c, Taxonomy_Year__c, Taxonomy__c, Last_Test_Filing__c, Status__c, Contains_iXBRL__c, Filed_On__c, Live_Filed_By__c, Notes__c, Planned_Filing_Date__c, Action__c, Source_Timestamp__c, Requested_By__c FROM SH_Project_Staging__c WHERE Status__c = :tmpVar1
10:58:01.5 (212043308)|SOQL_EXECUTE_END|[20]|Rows:0
10:58:01.5 (215316509)|SOQL_EXECUTE_BEGIN|[118]|Aggregations:0|SELECT SH_Project_ID__c, SH_Project_Last_Updated_Date__c FROM Project_SH__c WHERE SH_Project_ID__c IN :tmpVar1
10:58:01.5 (217498023)|SOQL_EXECUTE_END|[118]|Rows:0
10:58:01.5 (259434786)|SOQL_EXECUTE_BEGIN|[12]|Aggregations:0|SELECT Id, Apex_Job_ID__c, SH_Section_URL__c, Apex_Method__c, Message_ID__c, Section_Name__c, SH_Section_ID_Old__c, Status__c, Action__c, Source_Timestamp__c, Requested_By__c, isDeleted__c, SH_Project_ID__c, SH_Section_ID__c, Type__c FROM SH_Section_Staging__c WHERE Status__c = :tmpVar1
10:58:01.5 (268100822)|SOQL_EXECUTE_END|[12]|Rows:0
10:58:01.5 (274115475)|SYSTEM_MODE_ENTER|false
10:58:01.5 (274149542)|SYSTEM_MODE_EXIT|false
10:58:01.5 (274249854)|SYSTEM_MODE_ENTER|false
10:58:01.5 (274899818)|SYSTEM_MODE_EXIT|false
10:58:01.5 (274996005)|SYSTEM_MODE_ENTER|false
10:58:01.5 (275041588)|SYSTEM_MODE_EXIT|false
10:58:01.5 (275106180)|SYSTEM_MODE_ENTER|false
10:58:01.5 (276437969)|SOQL_EXECUTE_BEGIN|[125]|Aggregations:0|SELECT AD_Section_ID__c, AD_Section_Last_Updated_Date__c FROM AD_Section__c WHERE AD_Section_ID__c IN :tmpVar1
10:58:01.5 (278836715)|SOQL_EXECUTE_END|[125]|Rows:0
10:58:01.5 (279259872)|SYSTEM_MODE_EXIT|false
10:58:01.5 (279353091)|SYSTEM_MODE_ENTER|false
10:58:01.5 (279523154)|SYSTEM_MODE_EXIT|false
10:58:01.5 (279630965)|USER_DEBUG|[45]|DEBUG|#stagingRecordsFiltered ()
10:58:01.5 (279684521)|SYSTEM_MODE_ENTER|false
10:58:01.5 (280264989)|SOQL_EXECUTE_BEGIN|[148]|Aggregations:0|SELECT SH_Project_ID__c FROM Project_SH__c WHERE SH_Project_ID__c = :tmpVar1
10:58:01.5 (282546228)|SOQL_EXECUTE_END|[148]|Rows:0
10:58:01.5 (283007482)|SYSTEM_MODE_EXIT|false
10:58:01.5 (283121450)|SYSTEM_MODE_ENTER|false
10:58:01.5 (283841488)|SOQL_EXECUTE_BEGIN|[62]|Aggregations:0|SELECT AD_Section_ID__c, Name, isDeleted__c FROM AD_Section__c WHERE AD_Section_ID__c = :tmpVar1
10:58:01.5 (285649491)|SOQL_EXECUTE_END|[62]|Rows:0
10:58:01.5 (286056810)|SYSTEM_MODE_EXIT|false
10:58:01.5 (286170493)|SYSTEM_MODE_ENTER|false
10:58:01.5 (289832701)|SOQL_EXECUTE_BEGIN|[29]|Aggregations:0|SELECT AD_Section_SH__c, AD_Section_SH__r.AD_Section_ID__c, AD_Task__r.AD_Task_ID__c FROM SH_AD_Task_Section__c WHERE (AD_Section_SH__c = :tmpVar1 AND AD_Task__r.Task_Status__c NOT IN ('Completed', 'Canceled'))
10:58:01.5 (294045129)|SOQL_EXECUTE_END|[29]|Rows:0
10:58:01.5 (294574105)|SYSTEM_MODE_EXIT|false
10:58:01.5 (294660397)|SYSTEM_MODE_ENTER|false
10:58:01.5 (295594161)|SOQL_EXECUTE_BEGIN|[83]|Aggregations:0|SELECT AD_Section_SH__c, AD_Section_SH__r.AD_Section_ID__c, AD_Task__r.AD_Task_ID__c FROM SH_AD_Task_Section__c WHERE (AD_Section_SH__r.AD_Section_ID__c = :tmpVar1 AND Specialist_Status__c = NULL AND Production_Status__c = NULL AND Final_Review_Status__c = NULL)
10:58:01.5 (298590824)|SOQL_EXECUTE_END|[83]|Rows:0
10:58:01.5 (299128502)|SYSTEM_MODE_EXIT|false
10:58:01.5 (299191772)|SYSTEM_MODE_ENTER|false
10:58:01.5 (299717988)|SOQL_EXECUTE_BEGIN|[473]|Aggregations:0|SELECT Id, AD_Section_ID__c FROM AD_Section__c WHERE AD_Section_ID__c = :tmpVar1
10:58:01.5 (301390418)|SOQL_EXECUTE_END|[473]|Rows:0
10:58:01.5 (301759686)|SYSTEM_MODE_EXIT|false
10:58:01.5 (302359131)|SOQL_EXECUTE_BEGIN|[67]|Aggregations:0|SELECT Id, SH_Project_ID__c FROM Project_SH__c WHERE SH_Project_ID__c = :tmpVar1
10:58:01.5 (304520573)|SOQL_EXECUTE_END|[67]|Rows:0
10:58:01.5 (312424171)|SOQL_EXECUTE_BEGIN|[18]|Aggregations:0|SELECT Id, Apex_Job_ID__c, AD_Task_URL__c, Apex_Method__c, Task_Name__c, Due_Date__c, Weekend_OT_Approved__c, Weekend_OT_Approved_By__c, Task_Status__c, Description__c, Estimated_DFIN_Delivery_Date__c, Document_Format__c, Created_By_Email__c, Submit_EDGAR_Live_Filing__c, Submit_EDGAR_Test_Filing__c, Requested_Live_Filing_Date_Time__c, File_now__c, Total_of_XBRL_Comments__c, Number_of_Attachments__c, Number_of_Notes__c, Message_ID__c, AD_Task_ID__c, SH_Project_ID__c, Assigned_To__c, Status__c, Action__c, Source_Timestamp__c, Requested_By__c FROM SH_Task_Staging__c WHERE Status__c = :tmpVar1
10:58:01.5 (319382870)|SOQL_EXECUTE_END|[18]|Rows:0
10:58:01.5 (321371944)|SOQL_EXECUTE_BEGIN|[146]|Aggregations:0|SELECT AD_Task_ID__c, AD_Task_Last_Updated_Date__c FROM SH_AD_Task__c WHERE AD_Task_ID__c IN :tmpVar1
10:58:01.5 (323113698)|SOQL_EXECUTE_END|[146]|Rows:0
10:58:01.5 (324280220)|SOQL_EXECUTE_BEGIN|[249]|Aggregations:0|SELECT SH_Project_ID__c FROM Project_SH__c WHERE SH_Project_ID__c = :tmpVar1
10:58:01.5 (325891621)|SOQL_EXECUTE_END|[249]|Rows:0
10:58:01.5 (326809756)|SOQL_EXECUTE_BEGIN|[65]|Aggregations:0|SELECT Id, SH_Project_ID__c FROM Project_SH__c WHERE SH_Project_ID__c = :tmpVar1
10:58:01.5 (328398142)|SOQL_EXECUTE_END|[65]|Rows:0
10:58:01.5 (330918087)|SOQL_EXECUTE_BEGIN|[87]|Aggregations:0|SELECT Id, Apex_Job_ID__c, Apex_Method__c, Task_Section_Name__c, SH_Section_Staging__c, Message_ID__c, AD_Task_ID__c, AD_Section_ID__c, Status__c, Action__c, Source_Timestamp__c, Requested_By__c FROM SH_Task_Section_Staging__c WHERE (AD_Task_ID__c = :tmpVar1 AND Status__c = :tmpVar2)
10:58:01.5 (336932438)|SOQL_EXECUTE_END|[87]|Rows:0
10:58:01.5 (342931086)|SOQL_EXECUTE_BEGIN|[25]|Aggregations:0|SELECT Id, Apex_Job_ID__c, Apex_Method__c, Task_Section_Name__c, SH_Section_Staging__c, Message_ID__c, AD_Task_ID__c, AD_Section_ID__c, Status__c, Action__c, Source_Timestamp__c, Requested_By__c FROM SH_Task_Section_Staging__c WHERE Status__c = :tmpVar1
10:58:01.5 (346158812)|SOQL_EXECUTE_END|[25]|Rows:0
10:58:01.5 (348687041)|SOQL_EXECUTE_BEGIN|[347]|Aggregations:0|SELECT AD_Task__r.AD_Task_ID__c, AD_Section_SH__r.AD_Section_ID__c, AD_Task_Section_Last_Updated_Date__c FROM SH_AD_Task_Section__c WHERE (AD_Section_SH__r.AD_Section_ID__c = :tmpVar1 AND AD_Task__r.AD_Task_ID__c = :tmpVar2)
10:58:01.5 (350719731)|SOQL_EXECUTE_END|[347]|Rows:0
10:58:01.5 (351979494)|SOQL_EXECUTE_BEGIN|[589]|Aggregations:0|SELECT AD_Section_ID__c, Full_Section_Name__c, Name FROM AD_Section__c WHERE AD_Section_ID__c = :tmpVar1
10:58:01.5 (353513571)|SOQL_EXECUTE_END|[589]|Rows:0
10:58:01.5 (354794387)|SOQL_EXECUTE_BEGIN|[219]|Aggregations:0|SELECT AD_Task__c, AD_Section_SH__c, AD_Task__r.AD_Task_ID__c, AD_Task__r.Task_Status__c, AD_Section_SH__r.AD_Section_ID__c, Specialist_Status__c, Production_Status__c, Final_Review_Status__c FROM SH_AD_Task_Section__c WHERE (AD_Task__r.AD_Task_ID__c = :tmpVar1 AND AD_Section_SH__r.AD_Section_ID__c = :tmpVar2)
10:58:01.5 (356588430)|SOQL_EXECUTE_END|[219]|Rows:0
10:58:01.5 (357537048)|SOQL_EXECUTE_BEGIN|[204]|Aggregations:0|SELECT Id, AD_Section_ID__c FROM AD_Section__c WHERE AD_Section_ID__c = :tmpVar1
10:58:01.5 (359332123)|SOQL_EXECUTE_END|[204]|Rows:0
10:58:01.5 (360274817)|SOQL_EXECUTE_BEGIN|[191]|Aggregations:0|SELECT Id, AD_Task_ID__c, Assigned_To__c FROM SH_AD_Task__c WHERE AD_Task_ID__c = :tmpVar1
10:58:01.5 (361840075)|SOQL_EXECUTE_END|[191]|Rows:0
10:58:01.364 (364746131)|CUMULATIVE_LIMIT_USAGE
10:58:01.364 (364746131)|LIMIT_USAGE_FOR_NS|(default)
  Number of SOQL queries: 28 out of 100
  Number of query rows: 3 out of 50000
  Number of SOSL queries: 0 out of 20
  Number of DML statements: 2 out of 150
  Number of Publish Immediate DML: 0 out of 150
  Number of DML rows: 2 out of 10000
  Maximum CPU time: 149 out of 10000
  Maximum heap size: 0 out of 6000000
  Number of callouts: 0 out of 100
  Number of Email Invocations: 0 out of 10
  Number of future calls: 0 out of 50
  Number of queueable jobs added to the queue: 0 out of 50
  Number of Mobile Apex push calls: 0 out of 10

10:58:01.364 (364746131)|CUMULATIVE_LIMIT_USAGE_END
10:58:01.5 (364797260)|CODE_UNIT_FINISHED|SH_ScheduleStagingDataProcessing -- 58 min
10:58:01.5 (364815971)|EXECUTION_FINISHED
```
 #Salesforce #TaskIntegration 