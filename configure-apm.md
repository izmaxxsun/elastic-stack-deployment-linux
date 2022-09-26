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

Reference: https://www.elastic.co/guide/en/apm/guide/8.4/apm-quick-start.html

## Configure APM Agents
* Download the **elastic_apm_profiler_<version>.zip** from the Releases page: https://github.com/elastic/apm-agent-dotnet/releases
* Unzip the file into a folder on the IIS Server

<img width="870" alt="image" src="https://user-images.githubusercontent.com/100947826/192295249-b6302cf4-9c75-427d-b627-bf3fed68afd7.png">

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

# Troubleshooting
## Profiler Logs
By default, the Profiler logs will be written to: C:\ProgramData\elastic\apm-agent-dotnet\logs

![image](https://user-images.githubusercontent.com/100947826/192345980-f3b439fd-c267-4769-9cce-ed01c70fb2e9.png)
  
This can be changed with the environment variable **ELASTIC_APM_PROFILER_LOG_DIR**.  Reference: https://www.elastic.co/guide/en/apm/agent/dotnet/current/setup-auto-instrumentation.html
