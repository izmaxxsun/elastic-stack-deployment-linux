# Configure IIS on Windows Server
This guide walks through configuring IIS on Windows Server to serve up a .NET Framework and .NET Core web application.

## Install IIS
* From Server Manager, click **Add roles and features**
* On **Select installation type**, select **Role-based or feature-based installation**
* On **Select destination server**, select server from server pool and click **next**
* On **Select server roles**, check the **Web Server (IIS)** checkbox and click **next**
* From the **Add Roles and Features Wizard**, check **Include management tools** and click **Add Features**
<img width="415" alt="image" src="https://user-images.githubusercontent.com/100947826/192176892-1bfe8fd4-ca26-48de-a57d-5fd0ee9d3e4a.png">
* Click **Next**
* Check **ASP.NET 4.8** and click **Next**
<img width="783" alt="image" src="https://user-images.githubusercontent.com/100947826/192177026-f4c324a7-8325-4ee9-a973-bf7dcfe2f174.png">
* On **Web Server Role (IIS)**, click **Next**
* On **Select role services**, click **Next**
* Confirm selections and click **Install**

Reference: [Set up IIS on Windows Virtual Machine | Microsoft Developer Support](https://devblogs.microsoft.com/premier-developer/set-up-iis-on-windows-virtual-machine/)

## Install 
Install-WindowsFeature Web-Asp-Net45

## Install .NET Core Hosting Bundle
