
# AKS-Start-Stop-Automation

## Disclaimer
As per the Microsoft documentation, when using the cluster start/stop feature, the following restrictions apply:

- This feature is only supported for Virtual Machine Scale Sets backed clusters.
- The cluster state of a stopped AKS cluster is preserved for up to 12 months. If your cluster is stopped for more than 12 months, the cluster state cannot be recovered. For more information, see the AKS Support Policies.
- You can only start or delete a stopped AKS cluster. To perform any operation like scale or upgrade, start your cluster first.
- The customer provisioned PrivateEndpoints linked to private cluster need to be deleted and recreated again when you start a stopped AKS cluster.
- Because the stop process drains all nodes, any standalone pods (i.e. pods not managed by a Deployment, StatefulSet, DaemonSet, Job, etc.) will be deleted.

- When you start your cluster back up, the following is expected behavior: 
  - The IP address of your API server may change.
  - If you are using cluster autoscaler, when you start your cluster back up your current node count may not be between the min and max range values you set. The cluster starts with the number of nodes it needs to run its workloads, which isn't impacted by your autoscaler settings. When your cluster performs scaling operations, the min and max values will impact your current node count and your cluster will eventually enter and remain in that desired range until you stop your cluster

It is important that you don't repeatedly start/stop your cluster. Repeatedly starting/stopping your cluster may result in errors. Once your cluster is stopped, you should wait 15-30 minutes before starting it up again.

Source: https://docs.microsoft.com/en-us/azure/aks/start-stop-cluster?tabs=azure-cli 


### Author Disclaimer


**I do not recommend using this for production clusters as there is a chance that the cluster could fail during the start or stop process.**

If the cluster does fail, the following command may bring it back up:

```az resource update --name <AKS_CLUSTER_NAME> --resource-group <RESOURCE_GROUP> --namespace Microsoft.ContainerService --resource-type ManagedClusters```

## Requirements
- 1 Automation Account on each subscription that has at least 1 AKS cluster
- 2 runbooks in each Automation Account
  - 1 Runbook to start the cluster
  - 1 Runbook to stop the cluster
- 2 schedules in each Automation Account
  - Schedule to stop the cluster
  - Schedule to start the cluster

## Creating the resources
### Automation Account

1.  Go to the Automation Account resource and click “Create”
 
2. In the creation menu:
 - Basics
    - Choose the subscription that the cluster is in. 
    - Select or create a resource group *Can differ from the targeted cluster(s)*
    - Enter a name for the Automation Account
    - Select a region *Can differ from the targeted cluster(s)*
 - Advanced:
    - Select “System Assigned”
    - Unselect “User Assigned”
 - Networking
   - Connectivity Configuration
	   - Public Access
	   -  Private Access *(optional)* 
	   - *If private access is selected, you would have to create a private endpoint*
	   - Tags
	   - Review + Create
		   - Review the selected options and click “create”
		  
3. After creation, create a managed identity and assign it the “contributor” role and set the scope to “subscription”

### Runbook

1. Click “runbook under the “Process Automation” setting in your    automation account
   
2. Now click “Create a runbook”
   
3. In the “Create a runbook” page: 
	- Name: Stop-AKS-Clusters-By-Tag 
	- Runbook type: Powershell
	- Runtime version: 5.1
	- Description:    Selects all AKS clusters in the current subscription that have a tag    named "Hours" that has a value of    "Business"
   
4. Click “Create”
5.  Repeat the process for the second runbook that starts the cluster
	 - Name the fields accordingly


### Script

#### Logic

 - The automation account attempts to connect to azure using its’ managed Identity
 - The script will:
	 1. Get all the clusters
	 2. Filter the previous list of clusters to only list the ones that are successfully stopped (or started for the stop script) and have a tag named Hours that is equal to Business
		 - For each of these clusters, it will perform the stop or stop (depending on the script) process as a job.





#### Stop Cluster Script

 1. Select the stop-cluster runbook
 2. Click “Edit”
 3. Paste this script into the editor
 4. Click “Save” in the header
 5. Click “Publish”



#### Start Cluster Script

 1. Select the start-cluster runbook
 2. Click “Edit”
 3. Paste this script into the editor
 4. Click “Save” in the header
 5. Click “Publish”
 

## Scheduling

A schedule can be created and linked to a runbook to execute a script during certain days and specific times.

**Create a Schedule:**

 1. n the automation account settings, select “Schedules”
2. Select “Add a schedule”
3. Fill in the following fields:
	 - Name
	 - Description (optional)
	 - Starts
		 - Choose what day schedule should begin and at what time
 - Time zone
 - Reoccurrence
	 - Select “Recurring”
 - Recur Every:
 - 1 week
 - On these days:
	 - Monday
	 - Tuesday
	 - Wednesday
	 - Thursday
	 - Friday
 - Set Expiration
	 - Up to you
4. Create
5. Repeat this step for the second schedule and fill in its respective scheduling times


### Link Schedules to runbook

 1. Go to the Stop-Cluster runbook
 2. Click on “Schedules”
 3. Add a schedule
 4. Select “Link a schedule to your runbook”
 5. Then select the schedule that stops the cluster
6. Repeat the process for the start cluster runbook
