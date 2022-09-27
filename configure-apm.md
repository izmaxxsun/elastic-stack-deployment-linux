# Configuring APM Integration with Fleet
This guide assumes you've already setup Fleet. If you haven't, head over to [Installing Elastic Stack](README.md).

## Add the APM Integration
* Log in to Kibana
* Navigate to **Add Integrations** and select **Elastic APM**

<img width="851" alt="image" src="https://user-images.githubusercontent.com/100947826/192190297-fd78f9d5-4aa8-4163-981c-147df3a7820a.png">

* Click the **APM Integration** button

<img width="1489" alt="image" src="https://user-images.githubusercontent.com/100947826/192190355-18f3b193-4f30-4db9-951f-e01528df7e97.png">

* Click **Add Elastic APM**
* Leave the default settings as we'll have the APM integration set up on the IIS server

<img width="767" alt="image" src="https://user-images.githubusercontent.com/100947826/192190545-5c382889-c82b-4beb-a4a4-46b0b6cf7881.png">

* Select the **IIS Server** policy that was created when configuring the IIS integration (see writeup -- TBD)

<img width="753" alt="image" src="https://user-images.githubusercontent.com/100947826/192190640-dba28521-f447-4e11-b0de-3c6cf845715d.png">

* Check from the host where the APM Integration is deployed that you can reach http://localhost:8200

Reference: https://www.elastic.co/guide/en/apm/guide/8.4/apm-quick-start.html

## Configure APM Agents
Now that the APM Integration is running, we can start feeding it data by setting up APM agents.  In this example, we want to auto-instrument both a .NET Framework and a .NET Core application.  See [Configure IIS](configure-iis.md) for steps on getting these applications deployed on IIS.

### Download Elastic APM Profiler
* Download the **elastic_apm_profiler_<version>.zip** from the Releases page: https://github.com/elastic/apm-agent-dotnet/releases
* Unzip the file into a folder on the IIS Server

<img width="870" alt="image" src="https://user-images.githubusercontent.com/100947826/192295249-b6302cf4-9c75-427d-b627-bf3fed68afd7.png">

### Make sure C and C++ (MSVC) Runtime Libraries Installed (for .NET APM Agent)
This prerequisite is prominently listed at the top of the Profiler Auto Instrumentation documentation but can be overlooked and burn unnecessary time.  Download here: https://docs.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170

### Configure Environment Variables
There's a number of available settings you can update via environment variable and these can be scoped at your desired level.  For example, you could provide a separate set of environment variables per Application Pool or you could define it a higher level.
  
With IIS, these environment variables can also be set using AppCmd.  Example command below can be run in PowerShell to set the minimum needed settings.
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
}

$environment.Keys | ForEach-Object {
  & $appcmd set config -section:system.applicationHost/applicationPools "/+applicationPoolDefaults.environmentVariables.[name='$_',value='$($environment[$_])']"
}
```
![image](https://user-images.githubusercontent.com/100947826/192351064-653c80f5-c593-4286-9459-c86610f20316.png)


https://www.elastic.co/guide/en/apm/agent/dotnet/current/setup-auto-instrumentation.html
## .NET Core
  
```


appcmd.exe set config  -section:system.applicationHost/applicationPools /+"[name='webcore'].environmentVariables.[name='CORECLR_ENABLE_PROFILING',value='1']" /commit:apphost

appcmd.exe set config  -section:system.applicationHost/applicationPools /+"[name='webcore'].environmentVariables.[name='CORECLR_PROFILER',value='{FA65FE15-F085-4681-9B20-95E04F6C03CC}']" /commit:apphost

appcmd.exe set config  -section:system.applicationHost/applicationPools /+"[name='webcore'].environmentVariables.[name='CORECLR_PROFILER_PATH ',value='C:\Users\izmaxx\Downloads\elastic_apm_profiler_1.16.1\elastic_apm_profiler.dll']" /commit:apphost

appcmd.exe set config  -section:system.applicationHost/applicationPools /+"[name='webcore'].environmentVariables.[name='ELASTIC_APM_PROFILER_HOME',value='C:\Users\izmaxx\Downloads\elastic_apm_profiler_1.16.1']" /commit:apphost

appcmd.exe set config  -section:system.applicationHost/applicationPools /+"[name='webcore'].environmentVariables.[name='ELASTIC_APM_PROFILER_INTEGRATIONS',value='C:\Users\izmaxx\Downloads\elastic_apm_profiler_1.16.1\integrations.yml']" /commit:apphost


```
# Troubleshooting
## No APM Data
Per [.NET agent setup](https://www.elastic.co/guide/en/apm/agent/dotnet/current/setup-auto-instrumentation.html), the profiler requires Microsoft C and C++ (MSVC) runtime libraries to be installed.  Download here: https://docs.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170
  
## Profiler Logs
By default, the Profiler logs will be written to: C:\ProgramData\elastic\apm-agent-dotnet\logs

![image](https://user-images.githubusercontent.com/100947826/192345980-f3b439fd-c267-4769-9cce-ed01c70fb2e9.png)
  
This can be changed with the environment variable **ELASTIC_APM_PROFILER_LOG_DIR**.  Reference: https://www.elastic.co/guide/en/apm/agent/dotnet/current/setup-auto-instrumentation.html
