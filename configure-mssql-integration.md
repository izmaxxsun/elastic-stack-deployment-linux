# Monitoring SQL Server using Elastic Agent
This assumes you already have the Elastic platform setup (you can use Elastic Cloud or follow the [self-managed install](README.md))

## Setup SQL Server
Created Azure VM with SQL Server 2022

* Change **Server authentication** to **SQL Server and Windows Authentication Mode**

<img width="698" alt="image" src="https://user-images.githubusercontent.com/100947826/192433530-3966ef63-f695-454c-9ef7-e93755a4529a.png">

* Create a user that can view metrics (e.g. elastic:elastic)
* Restart SQL Server (using SQL Server Configuration Manager)

## Create Elastic Agent Policy
This is a re-usable configuration that can be applied to one or more SQL server instances.  We will use Fleet, the centralized manager for Elastic Agents, to deploy an agent to the SQL Server host machine.

* Navigate to Fleet
* Create a new Agent Policy, e.g. SQL Server Policy. This will include the System integration.

![image](https://user-images.githubusercontent.com/100947826/192433762-ae9caadf-01bf-45f7-bee5-dda7ae66db9e.png)

* After the new policy is created, click to open it again and add the **SQL Server** integration.  Pick the one that says **Collect events from Microsoft SQL Server with Elastic Agent**

![image](https://user-images.githubusercontent.com/100947826/192433906-3cc55a26-baa9-418d-af63-eed40afb791b.png)

* Click **Add Microsoft SQL Server Integration**
* For SQL Server integration, update the password to access metrics as well as the log file location (e.g. C:\Program Files\Microsoft SQL Server\MSSQL...)
* Click **Add Elastic Agent to your hosts**
* Follow the guide to enroll the Elastic Agent using Fleet

![image](https://user-images.githubusercontent.com/100947826/192434327-f8798cf9-8d31-4784-8317-8ec00ca0a6af.png)

* Check the Agent logs to make sure there are no errors

![image](https://user-images.githubusercontent.com/100947826/192436359-638280ff-306a-4b9b-83c2-1f9e8747f0b1.png)


## View Dashboards 
* Navigate to Dashboards
* Search for **SQL Server** to see dashboards such as **Metrics Microsfot SQL Server Performance**


## View Performance Metrics
These can be viewed from Discover UI and selecting the **metrics-*** data view

![image](https://user-images.githubusercontent.com/100947826/192436554-de36d0c0-c442-4eb6-943e-0536f5f2e4dc.png)

These can be viewed from Discover UI and selecting the **metrics-*** data view
