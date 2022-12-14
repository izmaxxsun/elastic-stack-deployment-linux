# Configure IIS on Windows Server
This guide walks through configuring IIS on Windows Server to serve up a .NET Framework and .NET Core web application. This example uses IIS Version 10.

🐪 If you already have IIS set up, skip to [Install ASP.NET](#install-aspnet)

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
* From **IIS Manager**, browse to the Default site to check the default website is running and IIS is operational

<img width="1258" alt="image" src="https://user-images.githubusercontent.com/100947826/192178292-21cdbdfe-18f2-4787-b402-9adc5c9633aa.png">

Reference: [Set up IIS on Windows Virtual Machine | Microsoft Developer Support](https://devblogs.microsoft.com/premier-developer/set-up-iis-on-windows-virtual-machine/)

## Install ASP.NET
* From **Windows Powershell**, run **Install-WindowsFeature Web-Asp-Net45**

<img width="733" alt="image" src="https://user-images.githubusercontent.com/100947826/192179690-55750f85-50ec-45a7-b2a0-047bc6f35359.png">

* Close IIS Manager and re-open it, an **ASP.NET** section is now visible

<img width="579" alt="image" src="https://user-images.githubusercontent.com/100947826/192403305-5a567db1-5ec0-480e-9a1b-0f4b50bb7b08.png">

## Install .NET Core Hosting Bundle
Download and install the hosting bundle (https://docs.microsoft.com/en-us/aspnet/core/tutorials/publish-to-iis?view=aspnetcore-6.0&tabs=visual-studio)

<img width="341" alt="image" src="https://user-images.githubusercontent.com/100947826/192403976-cffaf6ca-4cc1-44ee-9e11-518e7fc74397.png">

## Install C and C++ Runtime Libraries
This prerequisite is prominently listed at the top of the Profiler Auto Instrumentation documentation but can be overlooked and burn unnecessary time.  Download here: https://docs.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170

## Restart IIS
We've made a lot of changes, time to restart IIS. Couple ways to do this, but this one does the trick.
```
iisreset /noforce
```

## Deploy .NET Applications
## Serving Up a .NET Framework Application
* Copy the **framework.app.publish** folder from this repository or use Git to clone it onto the Windows server. This was built using .NET Framework 4.8. Place it in a folder, e.g. C:\lab_files.
* From IIS Manager, right-click on **Sites** and select **Add Website...**

<img width="295" alt="image" src="https://user-images.githubusercontent.com/100947826/192178956-cb8bbbc9-696c-47da-84b5-64de97b66980.png">

* Provide a **Site Name**, e.g. Framework WebApp
* Set **Physical Path** to the publish folder of the application, i.e. **C:\lab_files\framework.app.publish**
* Set the **Binding** to an unassigned port, e.g. 8080
* Click **Browse** to view the site

<img width="1200" alt="image" src="https://user-images.githubusercontent.com/100947826/192179411-c30432d6-c1c6-42b0-9d52-467c5cec76e5.png">

## Serving up a .NET Core Application
* Copy the **core.app.publish** folder from this repository or use Git to clone it onto the Windows server. This was built using .NET 6.  Place it in a folder, e.g. C:\lab_files.
* From IIS Manager, right-click on **Sites** and select **Add Website...**
* Provide a **Site Name**, e.g. Core WebApp
* Set **Physical Path** to the publish folder of the application, i.e. **C:\DemoFiles\WebApplication1\WebApplication1\bin\app.publish**
* Set the **Binding** to an unassigned port, e.g. 8081
* (Optional) From Windows Explorer, update the Security settings for **Web.config** (located in **bin/app.publish**) so that the **IIS_IUSRS** has read-execute access
* Click **Browse** to view the site

:tada: We've deployed two .NET applications from the legacy .NET Framework and the newer .NET Core. Next, we'll look at [Adding APM](configure-apm.md).
