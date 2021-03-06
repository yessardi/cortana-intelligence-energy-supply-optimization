# Energy Supply Optimization Solution

This document is focusing on the post deployment instructions for the automated deployment through [Cortana Intelligence Solutions](https://gallery.cortanaintelligence.com/solutions). The source code of the solution as well as manual deployment instructions can be found [here](https://github.com/Azure/cortana-intelligence-energy-supply-optimization/tree/master/Manual%20Deployment%20Guide).

# Architecture
The architecture diagram shows various Azure services that are deployed by [Resource Optimization Solution]() using [Cortana Intelligence Solutions](https://gallery.cortanaintelligence.com/solutions), and how they are connected to each other in the end to end solution.

![Solution Diagram](https://github.com/Azure/cortana-intelligence-energy-supply-optimization/blob/master/Manual%20Deployment%20Guide/Figures/resourceOptArchitecture.png)

</Guide>

## Technical details and workflow

1.  The sample data is streamed by newly deployed **Azure Web Jobs**. The web job uses resource related data from Azure SQL to generate the simulated data.

2.  This simulated data feeds into the **Azure Storage** and writes message in storage queue, that will be used in the rest of the solution flow.

3.  Another **Web Job** monitors the storage queue and initiate an Azure Batch job once message in queue is available.

4.  The **Azure Batch** service together with **Data Science Virtual Machines** is used to optimize the energy supply from a particular resource type given the inputs received. (Data Science Virtual Machines are selected in this example due to the convenience they bring to the developer, and alternatives exist for acheiving other goals such as system efficiency.)

4.  **Azure SQL Database** is used to store the optimization results received from the **Azure Batch** service. These results are then consumed in the **Power BI** dashboard.

6.  Finally, **Power BI** is used for results visualization.

All the resources listed above are already deployed in your subscription. The following instructions will guide you on how to monitor things that you have deployed and create visualizations in Power BI.

# Post Deployment Instructions
Once the solution is deployed to the subscription, you can see the services deployed by clicking the resource group name on the final deployment screen in the CIS.

This will show all the resources under this resource groups on [Azure management portal](https://portal.azure.com/).

After successful deployment, the entire solution is automatically started on cloud. You can monitor the progress from the following resources.

## **Monitor progress** 

#### Web Jobs/Functions
3 Azure Web jobs are created during the deployment. You can monitor the web jobs by clicking the link on your deployment page.
* One-time running web jobs are used to start certain Azure services.
  * PopulateStorageContainer: This function is used to upload all the Spark Job files to Azure Blob Storage.
  * ExecuteSqlQuery: This function helps create required tables in Azure SQL Database.
  * installpkgs: This webjob installs necessary python packages on the web app server to run the AzureBatchWebJob.
  * CiqsHelpers, pbjs, pbiweb, uploadPbix: These are the helper functions which are mostly used to deploy the Power BI, upload the dashboard and establish the Power BI connection with SQL Server.

* Continuous running web jobs are used as data generator.
 * EnergyResourceDataSimulator : This is a webjob which runs on Function App Server and writes the rawdata into incommingdatafiles blob container and a message in the incommingmessage queue. 
  * derciqs02: This webjob monitors the incommingmessage queue and once it finds messages in the queue, it moves the raw data from incommingdatafiles to batchdatafiles, creates Azure Batch and submits job to the Azure batch cluster. The job runs optimization code on batch nodes and it writes the result back to the SQL Database energyoptciqs.

#### Azure Batch
Azure Batch is the compute engine in this solution. Each time the data simulator writes data/message in the storage, a webjob reads the message, creates batch pool in Azure Batch and submits the optimization job for the simulated data. 

#### Azure SQL Database
Azure SQL database is used to save the data and optimized results. You can use the SQL server and database name showing on the last page of deployment with the username and password that you set up in the beginning of your deployment to log in your database and check the results.

## **Visualization**

The essential goal of this part is to get the optimization results and visualize it. Power BI can directly connect to an Azure SQL database as its data source, where the prediction results are stored.

The PowerBI dashboard is deployed along with the solution and you can get the link to the dashboard in the after deployment instruction page. 

If you want a desktop version of Power BI, follow the instruction provided [here](https://github.com/Azure/cortana-intelligence-energy-supply-optimization/tree/master/Manual%20Deployment%20Guide#7-setup-power-bi) to import the template and create/publish your own PowerBI Dashboard. 


## **Customization**
You can reuse the source code in the [Manual Deployment Guide](https://github.com/Azure/cortana-intelligence-energy-supply-optimization/tree/master/Manual%20Deployment%20Guide) to customize the solution for your data and business needs.
