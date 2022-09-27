# Configuring APM Integration with Fleet
This guide assumes you've already setup Fleet along with an Elastic Agent on the IIS Server. If you haven't, head over to [Installing Elastic Stack](README.md).

## Add the APM Integration
* Log in to Kibana
* Navigate to **Add Integrations** and select **Elastic APM**

<img width="851" alt="image" src="https://user-images.githubusercontent.com/100947826/192190297-fd78f9d5-4aa8-4163-981c-147df3a7820a.png">

* Click the **APM Integration** button

<img width="1489" alt="image" src="https://user-images.githubusercontent.com/100947826/192190355-18f3b193-4f30-4db9-951f-e01528df7e97.png">

* Click **Add Elastic APM**
* Leave the default settings as we'll have the APM integration set up on the IIS server

<img width="767" alt="image" src="https://user-images.githubusercontent.com/100947826/192190545-5c382889-c82b-4beb-a4a4-46b0b6cf7881.png">

* Select the **IIS Server** policy that was created when configuring the IIS integration [Installing Elastic Stack](README.md).

<img width="753" alt="image" src="https://user-images.githubusercontent.com/100947826/192190640-dba28521-f447-4e11-b0de-3c6cf845715d.png">

* Check from the host where the APM Integration is deployed that you can reach http://localhost:8200
* You can also view the Agent logs to see if there are any issues with the APM server

![image](https://user-images.githubusercontent.com/100947826/192405529-9684ffd5-0901-4c91-aa93-dddcd283e08a.png)

Reference: https://www.elastic.co/guide/en/apm/guide/8.4/apm-quick-start.html

## Configure APM Agents
Now that the APM Integration is running, we can start feeding it data by setting up APM agents.  In this example, we want to auto-instrument both a .NET Framework and a .NET Core application.  See [Configure IIS](configure-iis.md) for steps on getting these applications deployed on IIS.

### Download Elastic APM Profiler
* Download the **elastic_apm_profiler_<version>.zip** from the Releases page: https://github.com/elastic/apm-agent-dotnet/releases
* Unzip the file into a folder on the IIS Server (version 1.16.1 was used for this example)

<img width="870" alt="image" src="https://user-images.githubusercontent.com/100947826/192295249-b6302cf4-9c75-427d-b627-bf3fed68afd7.png">

### Configure Environment Variables
There's a number of available settings you can update via environment variable and these can be scoped at your desired level.  For example, you could provide a separate set of environment variables per Application Pool or you could define it a higher level.
  
With IIS, these environment variables can also be set using AppCmd.  Example command below can be run in PowerShell to set the minimum needed settings.
- ELASTIC_APM_ENVIRONMENT - can be used for data separation between teams and projects (see UNICC bitbucket writeup)
  
#### Setting Application Pool Defaults
This will detect either .NET Framework or Core applications when they startup. This can be set from the Configuration Editor in IIS or the AppCmd can be used to programatically set it through PowerShell:

<img width="1327" alt="image" src="https://user-images.githubusercontent.com/100947826/192412844-779c922b-9fdd-4bc8-9e76-9c86bbcbb5cf.png">

```
$appcmd = "$($env:systemroot)\system32\inetsrv\AppCmd.exe"

$profilerHomeDir = "C:\elastic_apm_profiler_1.16.1" 
$environment = @{
  COR_ENABLE_PROFILING = "1"
  COR_PROFILER = "{FA65FE15-F085-4681-9B20-95E04F6C03CC}"
  COR_PROFILER_PATH = "$profilerHomeDir\elastic_apm_profiler.dll"
  ELASTIC_APM_PROFILER_HOME = "$profilerHomeDir"
  ELASTIC_APM_PROFILER_INTEGRATIONS = "$profilerHomeDir\integrations.yml"
  COMPlus_LoaderOptimization = "1" 
  CORECLR_ENABLE_PROFILING = "1"
  CORECLR_PROFILER_PATH = "$profilerHomeDir\elastic_apm_profiler.dll"
}

$environment.Keys | ForEach-Object {
  & $appcmd set config -section:system.applicationHost/applicationPools "/+applicationPoolDefaults.environmentVariables.[name='$_',value='$($environment[$_])']"
}
```

#### Setting Application Pool Specific Variables
See [All Options Summary](https://www.elastic.co/guide/en/apm/agent/dotnet/current/config-all-options-summary.html) for all settings, some specific ones that may be useful at a lower level are:
  
* [Service Environment](https://www.elastic.co/guide/en/apm/agent/dotnet/current/config-core.html#config-environment) - this is used for filtering in the APM application and can be set to any string value such as "team1-dev" or "team2-prod"
* [Service Name](https://www.elastic.co/guide/en/apm/agent/dotnet/current/config-core.html#config-service-name) - this will default to the assembly name but you can give it a more user-friendly name if needed
  
<img width="786" alt="image" src="https://user-images.githubusercontent.com/100947826/192414245-82316c5a-eb8a-4e92-9a1e-d3d74d6b9a47.png">
  
# End Result
🍰 We should now see APM data for multiple .NET applications in the APM UI.
  
![image](https://user-images.githubusercontent.com/100947826/192415221-83854e2f-b39b-4a02-a537-1a588f59465a.png)


# Troubleshooting
## No APM Data
Per [.NET agent setup](https://www.elastic.co/guide/en/apm/agent/dotnet/current/setup-auto-instrumentation.html), the profiler requires Microsoft C and C++ (MSVC) runtime libraries to be installed.  Download here: https://docs.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170
  
## Profiler Logs
By default, the Profiler logs will be written to: C:\ProgramData\elastic\apm-agent-dotnet\logs

![image](https://user-images.githubusercontent.com/100947826/192345980-f3b439fd-c267-4769-9cce-ed01c70fb2e9.png)
  
This can be changed with the environment variable **ELASTIC_APM_PROFILER_LOG_DIR**.  Reference: https://www.elastic.co/guide/en/apm/agent/dotnet/current/setup-auto-instrumentation.html
