# Monitoring SQL Server using Elastic Agent
This assumes you already have the Elastic platform setup (you can use Elastic Cloud or follow the [self-managed install](README.md))

## Setup SQL Server
Created Azure VM with SQL Server 2022

## Create Elastic Agent Policy
This is a re-usable configuration that can be applied to one or more SQL server instances.  We will use Fleet, the centralized manager for Elastic Agents, to deploy an agent to the SQL Server host machine.

* Navigate to Fleet
* Create a new Agent Policy, e.g. SQL Server Policy
* Include the System and SQL Server integrations
* For SQL Server integration, update the password to access metrics as well as the log file location (e.g. C:\Program Files\Microsoft SQL Server\MSSQL...)
* Install on SQL Server host machine

## View Dashboards 

## View Performance Metrics
These can be viewed from Discover UI
