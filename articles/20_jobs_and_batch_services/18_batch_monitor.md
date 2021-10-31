# **Fabric Batch Monitoring Panel**

## **Overview**  

The batch monitoring dashboard has been added to Fabric version 6.5.2. 

It has been designed to provide a clear picture of all batch processes running across your Fabric servers during operation. It delivers real-time and historical insights such as the performance or the success/failure rates of the processes per node, per entity or per status basis.  

The Batch Monitoring Dashboard can be accessed from Fabric's [Web Admin](/articles/30_web_framework/01_web_framework_overview.md) panel by doing the following steps:
- Navigate to **Admin > Processes > Batch**,
- Select the ```...``` button in the first column of the batch processes table ![image](images/30_jobs_and_batch_services_batchMonitor6.PNG),
- Select the **Monitor** item of the *menu*. as illustrated in the picture below:

![image](images/25_jobs_and_batch_services_batchMonitor1.PNG)


You will need to ensure that a batch processes is being executed or has been previously executed. 
The picture below shows how to create a new batch process or how to access a previously executed batch process.


![image](images/26_jobs_and_batch_services_batchMonitor2.PNG)





## **Batch Monitor Dashboard** 

Click on the **Monitor** item of the *menu*, and the following dashboard will appear:

![image](images/27_jobs_and_batch_services_batchMonitor3.PNG)


### **Batch Monitor**

The batch Monitor Control banner allows you to cancel, pause and resume the batch process currently in focus. 
It provides a progress bar that indicates the number of entities already processed and the number of entities yet to be processed.

![image](images/31_jobs_and_batch_services_batchMonitor7.PNG)


### **General Data**

Execution Status:
Provides the status of the current batch process as defined by the [batch_summary](/articles/20_jobs_and_batch_services/12_batch_sync_commands.md#batch_summary) command

Fabric Command:
The actual batch command being processed. All details can be viewd by clicking on this sign: ![image](images/28_jobs_and_batch_services_batchMonitor4.PNG)

The fully detailed list of tha batch process (resulting from the execution of the [batch_list](/articles/20_jobs_and_batch_services/12_batch_sync_commands.md#batch_list) command) will be displayed as illustrated below:

![image](images/29_jobs_and_batch_services_batchMonitor5.PNG)

Max. No. of Workers:
This parameter can be modified during the execution time and defines the number of threads allocated to this batch process. The value can be adjusted by clicking on the following icon: ![image](images/30_jobs_and_batch_services_batchMonitor6.PNG).


### **Execution Time**

This panel displays the information resulting from the execution of the [batch_summary](/articles/20_jobs_and_batch_services/12_batch_sync_commands.md#batch_summary) command.

![image](images/32_jobs_and_batch_services_batchMonitor8.PNG)



### **Entity Status Summary**

This panel provides a pie-chart view of the status of the entities being processed. See the illustration below: 

![image](images/33_jobs_and_batch_services_batchMonitor9.PNG)


### **Entities Detailed Status**

Entities are categorized into 3 different groups:
- Entities in process
- Failed entities
- Entities with the highest execution time

This allows you to drill down on each instance's execution and identify bottlenecks, issues or unexpected behaviors.


Each table shows the following details for each entity:
- Entity ID
- Node ID
- Process time in ms
- Status

Right click on the ![image](images/28_jobs_and_batch_services_batchMonitor4.PNG) icon, and you will be redirected to the table showing the details for the node that processed this specific entity.


![image](images/34_jobs_and_batch_services_batchMonitor10.PNG)


### **Batch process monitoring per Node or per Data Center**

The summary of the execution of the batch process is shown on a per-node basis or across the Data Centers responsible for the execution of the process.

- DC-centric report:

![image](images/35_jobs_and_batch_services_batchMonitor11.png)

- Node-centric report:

![image](images/36_jobs_and_batch_services_batchMonitor12.png)





[![Previous](/articles/images/Previous.png)](17_batch_process_flow.md)
