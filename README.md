# Overview
There are several ways to deploy the Elastic stack, in order of increasing difficulty:
- Fully-managed cloud offering (start a [free trial](https://www.elastic.co/industries/public-sector/fedramp) with no credit card)
- Self managed with Elastic Cloud for Kubernetes (ECK) or Elastic Cloud Enterprise (ECE)...these are orchestrated options making it simple to install and upgrade
- Self-managed using binaries

In this walkthrough, we'll deploy Elastic using the last option.  This will include setting up self-managed Elasticsearch, Kibana, and Fleet Server on a Linux virtual machine. Then we'll set up some Observability for an IIS Server running both .NET Core and .NET Framework applications, as well as a SQL Server to see both infrastructure metrics and APM data. 

:raised_hand: **That being said, if using the Elastic Cloud...skip to [Configuring IIS](configure-iis.md).**

![](https://www.elastic.co/guide/en/fleet/master/images/fleet-server-communication.png)
Reference: [https://www.elastic.co/guide/en/fleet/master/fleet-server.html](https://www.elastic.co/guide/en/fleet/master/fleet-server.html)

# Setup
* Elastic v8.3.3
* AWS EC2 t2.large (2vCPU, 8GiB RAM)
* Debian 11 Bullseye

# Install Elasticsearch
* [Download and unzip archive](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/targz.html)
* Run Elasticsearch
```
./bin/elasticsearch
```
* Copy the password, HTTP CA certificate SHA-256 fingerprint, and Kibana enrollment token
* Add inbound rule on port 9220 for EC2 Security Group
* Open up **elasticsearch.yml** located in $ES_HOME/config and change network.host to **0.0.0.0**
<img width="598" alt="image" src="https://user-images.githubusercontent.com/100947826/192082715-2d81185a-687d-48f4-aaca-5bf92b6cb1af.png">

* Check Elasticsearch is running
```
export set ES_HOME=/home/admin/elasticsearch-8.4.2
curl --cacert $ES_HOME/config/certs/http_ca.crt -u elastic https://localhost:9200
```
# Install Kibana
* [Download and unzip archive](https://www.elastic.co/guide/en/kibana/8.3/targz.html)
* Update **kibana.yml** in $KIBANA_HOME/config with **server.host=0.0.0.0** to make it externally accessible
* From EC2 Security Group, add inbound rule for port 5601.
* Run Kibana
```
./bin/kibana
```
<img width="800" alt="image" src="https://user-images.githubusercontent.com/100947826/192077161-0f0f7e7d-0931-48da-89c6-ebe16832242d.png">

* Open up browser to specified URL and enter enrollment token noted previously

:heart: Elasticsearch and Kibana are now setup and you can login with username **elastic** and the password noted in previous step

# Setup Fleet Server
Now we'll setup Fleet Server which provides centralized management of Elastic Agents.

* Navigate to Fleet
* Follow Quickstart
  * Set Fleet Server URL to **https://<dns_of_ec2_instance>:8220**
  * add **--insecure** flag to end of command to bypass self-signed cert warning. For production environments, see [Configure TLS/SSL](https://www.elastic.co/guide/en/fleet/8.4/secure-connections.html)

```
sudo ./elastic-agent install \   
--fleet-server-es=https://172.31.44.170:9200 \  
--fleet-server-service-token=AAEAAWVsYXN0aWMvZmxlZXQtc2VydmVyL3Rva2VuLTE2NjM5OTU1NTU2ODA6d0hKdmdMNTFTbEMwWUhheHFZVmRJdw \  
--fleet-server-policy=fleet-server-policy \   
--fleet-server-es-ca-trusted-fingerprint=3acd20b394adcd3fcd83ada57cec534d2c653237ddda65eacff91d66789ad7be \
--insecure
```
* Add inbound port rule for port 8220 for EC2 Security Group
<img width="1388" alt="image" src="https://user-images.githubusercontent.com/100947826/192081144-cf388ecd-b0c3-474b-a159-f35495a5b5f1.png">

* Update **kibana.yml** to point the Fleet output to the publicly accessible Elasticsearch address
<img width="841" alt="image" src="https://user-images.githubusercontent.com/100947826/192083211-a23fa6af-8950-4431-bfef-fdd0c5bd08fe.png">

* Navigate to Fleet UI and check that Fleet Server agent is healthy and logs are displayed

<img width="1455" alt="image" src="https://user-images.githubusercontent.com/100947826/192081030-3844a3b5-8aef-4954-b84a-a9303b4e1963.png">

* Restart Kibana

:tada: Take a minute to enjoy successfully setting up Elasticsearch, Kibana, and Fleet Server. Next, we'll get into adding the single, unified Elastic Agent to hosts we want to monitor.

# IIS Server
We'll set up an IIS Server from scratch and deploy both .NET Core and .NET Framework sample applications to see the Elastic APM profiler in action.

## Create Windows Server VM
In this example, a Windows Server VM is created using Azure.  We'll configure it later with IIS and some .NET applications later in [Configure IIS](configure-iis.md).

<img width="795" alt="image" src="https://user-images.githubusercontent.com/100947826/192081239-08d060fa-a9f0-4cde-b023-640b28ba07dc.png">

## Adding an Elastic Agent
* From the Elastic Fleet UI, click **Add Agent**
* Create a new policy called **IIS Server Policy**

<img width="836" alt="image" src="https://user-images.githubusercontent.com/100947826/192081626-06d5e9de-e7fd-44a5-a921-2802deca520a.png">

* Copy the Powershell command for Windows installation of the Elastic Agent
* Add the **--insecure** flag since we are using self-signed certificates and run the Powershell script on the Windows Server VM
```
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.3.3-windows-x86_64.zip -OutFile elastic-agent-8.3.3-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.3.3-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.3.3-windows-x86_64
.\elastic-agent.exe install --url=https://ec2-3-14-3-84.us-east-2.compute.amazonaws.com:8220 --enrollment-token=TWozNWJZTUJWQlQ3RU9yaFBkdTM6cHdGTkYtRXVUR3F0QjdVOFBCNTZXdw== --fleet-server-es-ca-trusted-fingerprint=3acd20b394adcd3fcd83ada57cec534d2c653237ddda65eacff91d66789ad7be --insecure
```
* Confirm data in the Agent logs

![image](https://user-images.githubusercontent.com/100947826/192442954-42fb7bb3-ebf7-4347-a303-c6eaa68fbd80.png)

# Wrapping Up
:tada: We've configured Elastic and Kibana with Fleet Server. Next we move on to [Configuring IIS](configure-iis.md)

![image](https://user-images.githubusercontent.com/100947826/192443230-5f9c97ef-c44a-45d0-af9e-9ddb7d34da77.png)


# Troubleshooting
## Reset Elastic password
If you forget the elastic built-in user password, run the following:
```
bin/elasticsearch-reset-password -u elastic
```
Reference: https://www.elastic.co/guide/en/elasticsearch/reference/8.4/targz.html

