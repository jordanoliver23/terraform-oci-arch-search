# OCI Search Service with OpenSearch - Tutorial

## **Pre-requisites**  

### 1. Create the required service policies in the OCI Console, tailoring them to your needs
(e.g. change *any-user* to the desired group, provide the path to the compartment if required)  
  
Working policies:  

```
Allow service opensearch to manage vnics in compartment opensearch
Allow service opensearch to use subnets in compartment opensearch
Allow service opensearch to use network-security-groups in compartment opensearch
Allow group <your_group> to manage opensearch-family in compartment opensearch - OPTIONAL
```

### 2. Create a VCN with a public subnet and a private subnet

Simplified process:  

Console menu &rarr; Networking &rarr; Virtual Cloud Networks &rarr; Start VCN Wizard &rarr; Create VCN with Internet Connectivity  

Custom process:  

“Create VCN” instead of “Start VCN Wizard”, providing your own desired details.  
NOTE: A IPv6 enabled subnet is not supported due to a limitation with Private Endpoints (PE). This should be resolved later in June 2022
<br>

### 3. Create a VM Instance in the public subnet of the VCN
 
Console menu &rarr; Compute &rarr; Instances &rarr; Create instance  
Choose the instance name, compartment and select the public subnet.  
For the other options we will use the default values (Oracle Linux 8, VM.Standard.E4.Flex, assign a public IP).  
Decide whether you wish to use an existing SSH key, or have a new one generated. If choosing to generate, remember to download the keys.

<br>

## **Note** 
For the exercise contained in this tutorial, a Mac and an Oracle Linux instance were used. Windows command line instructions may differ.  

<br>

## 1. Create an OCI Search Service cluster

NOTE: Charges for OCPU, Memory, Block Volume, and Object Storage will incur when the service launches. Also any VMs used to access the cluster endpoints. 

Console menu &rarr; Databases &rarr; OCI Search Service &rarr; Clusters  
Choose the cluster name and compartment where to create the cluster

<img src=".//media/image1.png" style="width:6.26806in;height:3.10347in"
alt="Graphical user interface, text, application, email Description automatically generated" />

Choose the cluster sizing.

<img src=".//media/image2.png" style="width:6.26806in;height:3.06181in"
alt="Graphical user interface, text, application, email Description automatically generated" />

Select the VCN you created. Select the private subnet.

<img src=".//media/image3.png" style="width:6.26806in;height:3.07847in"
alt="Graphical user interface, text, application, email Description automatically generated" />

After the cluster creation, take note of the API endpoints and the IP
addresses which you can alternatively use. These are present in the OCI
Search Service cluster details page.

<img src=".//media/image4.png" style="width:6.26806in;height:1.97708in"
alt="Graphical user interface, text, application Description automatically generated" />

## 2.  Create security rules in the VCN Security List
  
In the VCN, create a Security List with the following security rules. Alternatively, they can be added to the VCN Default Security List.  
Within your VCN details page &rarr; Security Lists &rarr; chosen Security
List &rarr; Add Ingress Rules

Add a rule for port 9200 (OpenSearch), and a rule for 5601 (OpenSearch Dashboards).

<img src=".//media/image5.png" style="width:6.26806in;height:2.37292in" />

<img src=".//media/image6.png" style="width:6.26806in;height:2.39722in" />

<img src=".//media/image0.png" style="width:7.50806in;height:3.10347in" />
  

## 3.  Download the required certificate

Run the following command from inside the created VM instance.  
```
curl -O https://raw.githubusercontent.com/oracle-devrel/terraform-oci-arch-search/main/cert.pem
```  
The certificate will be downloaded and saved as cert.pem, in your current directory.  
Note: this certificate is suitable to region us-ashburn-1.

As an alternative, you can just create your own cert. This is a public cert from oracle but not one that's from a trusted authority (e.g. CA)
Create a cert and store it in the ssh hidden folder under your users directory

```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA2y6afR3mPA/I8e2kRLUvsbjtmc8DBFX8anM6bd/4nv4AvNcI
LW93UEBDNTk2aLw2VBRLeYaht/QfZyP2NTWq7tCleFFyzZoWmPpjmnI7IaMCqUkK
VpjRDHae3+i5w3VdUWvGeo6AxXg64Jqoen5YevX19MeCPFcnYMLW2iCo2yEXX+8e
JAx1Wspxil1Bxnx6s7DGulldpnHQ7IiQrQYCqGd56SjY6xSrlMjNby4YLLmOftSU
9H9RMvmL39i9/CROEcVRb3E6vovfhz246nMrtOkLxOjieu2ztQ2pOswCMxfJcHFP
NrQk5RvIUw+LykuQOHVyvnUzy7KFsDAmhEryZwIDAQABAoIBAAXRTVlvzzod4yQE
vjzyoDLz6R6Rf4+AZsQ+jbj33mX98PASNw3ZrQ2MvxvtClQqVrjRlxVBLQ6wZJr8
ud68r25KTHIOm2D4q4vg7X7edFJWvM3YefVFdhsCFQJ1b2TQOytblHeRS7qyD8IB
aOJjcx7EY4RdPUgzugBX+5Lrlf/G6vwZK1GrsdJA5pIHgSrYHYpOYlf3CnI6r7On
Wl7hX0ye8YY/1iQWCdFxM3JE4BkmkZftVcdrrNtdTBRYdV7ZAhFa9eFFz/pmPNb6
PknADclVfuSNfUTKOIb+6etm9q6IekKU6whZzS5U+1Bb+hnp8FGfzP4mWMerqDmP
hHhuwv0CgYEA+UhRmjvQ/zKBXSB7w/aCNfz8ilqF452q6K4cIyvncIvUkKrzPbf/
oD4ERgP2xYXQng/G+wFHitPYF+xob7gEeNbjMFcrKnfGMYWZi3P88/y2iBKvaeZq
phDsbMYnEt0T5LEzxpn4336cEPRFqhWhwyST9ybIC7URI/In3yOkNxUCgYEA4Rai
01QIBvqhaCm0SB460aOoJV1XWlpvpXPpS/04ci1QvMUJUxkndFjAbLZMYzRjWP9v
qLPxKqlH8iCKUqLYZRuIP4ZBBd6xMCU+ceWryBvJ/cP5xfMkmQ03i3BR61zon8ot
n2JBoafRX5uGx04Ur1ceimPliFWN5bh6VHufYosCgYBTvAUdJ8aWUmK943Fva9hl
RiuWVb3vrUCBlCqDbfX6Ch5G0gWOz8WgD/Tjh+VWiBKBZY9TNSTQ70QBFTonfMqT
xKrfzAgF5eG/NL9U5osrcdHmd1BQ5EMisUCZcR4i6fwKr7NSnNnKSP8nesYD0exa
XmkNdgtwU0wEpQzbmV9J2QKBgQCgpX00gsbv5DUKmKk4x4qHUNyTPlk3/U+tsFqT
h3if1MPI1n/fNRa5rRY5AKroKt21CSnyJ+s53XOh1aOjcuIq10mYvQLvY47mo847
kAXYXiz91r8PjodSTOKVvGZbKwZD9RI2rPPWomWGbQP2fz24Ht+HOeD6OsV5bP6y
CUEqHQKBgGZZ0wKovepNPid7hN0gBQ2pkg0KaWwagSm87bzsmMhIeK2k+N3ufna1
wB3RDqrNsZGW/emUdI1ekcu/A2EcSSrDlyjXgC0rgv/7P7WHvfu74AiHT301xgMw
P6ca3blziswmBFftSmkYMPW07YkTrERN3kK11pVYYcDjD8qRt43B
-----END RSA PRIVATE KEY-----
```

## 4.  Test the connection to OCI Search Service – OpenSearch endpoint

### 4.1. From inside the created VM instance  
NOTE: This is done just to validate access from your VM to the OS cluster
  
a.  Connect to the instance via SSH:  
```
ssh -i ~/.ssh/id_rsa_opensearch.key opc@<your_VM_instance_public_IP>
```
JORDAN:
```
ssh -i ~/.ssh/id_rsa_opensearch.key opc@10.0.154.88
```

b.  Run one of the following commands:
```
curl https://amaaaaaanlc5nbya44qen6foty3gyu7ihpo22mzmtjw5ixtcjgetjcqwipuq.opensearch.us-ashburn-1.oci.oracleiaas.com:9200 --cacert cert.pem
# OpenSearch API endpoint example, with certificate

curl https://10.1.1.190:9200 --insecure 
# OpenSearch private IP example
```
JORDAN:
```
curl https://amaaaaaakztjrmiaca3oyxmzegtn25lfh77dmc6heo65r647cc4u3qvz4ewa.opensearch.us-ashburn-1.oci.oracleiaas.com:9200 --cacert cert.pem
# OpenSearch API endpoint example, with certificate

curl https://10.0.216.169:9200 --insecure 
# OpenSearch private IP example
```


## 4.2.  From your local machine, through port forwarding
## (alternative to 4.1.)

a. Run the following port forwarding SSH command in the Terminal. Do not
close the Terminal afterwards, for the connection to remain in place.

```
ssh -C -v -t -L 127.0.0.1:5601:<your_opensearch_dashboards_private_IP>:5601 -L 127.0.0.1:9200:<your_opensearch_private_IP>:9200 opc@<your_VM_instance_public_IP> -i <path_to_your_private_key>
```
NOTE: If you lose access to OSD, check your tunneling. It seems to drop fairly quickly with no activity

JORDAN
```
ssh -C -v -t -L 127.0.0.1:5601:10.0.211.1:5601 -L 127.0.0.1:9200:10.0.216.169:9200 opc@129.158.214.157 -i ~/.ssh/ssh-key-2022-03-14.key
```

b. Open a new Terminal window and run the following command:
```
curl https://localhost:9200 --insecure
```

If all the steps were performed correctly you should see a response as
follows, regardless of what option was chosen:  

```
{
  "name" : "opensearch-master-0",
  "cluster_name" : "opensearch",
  "cluster_uuid" : "M6gclrE3QLGEBlkdme8JkQ",
  "version" : {
    "distribution" : "opensearch",
    "number" : "1.2.4-SNAPSHOT",
    "build_type" : "tar",
    "build_hash" : "e505b10357c03ae8d26d675172402f2f2144ef0f",
    "build_date" : "2022-02-08T16:44:39.596468Z",
    "build_snapshot" : true,
    "lucene_version" : "8.10.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
}
```

## 5. Ingest data

Run the following commands from within your VM instance.  
We are providing examples both with --cacert and --insecure.  
When using --cacert, always use the cluster API endpoint and not the cluster private IP.

### 5.1. Download the sample data set

```
curl -O https://raw.githubusercontent.com/oracle-devrel/terraform-oci-arch-search/main/OCI_services.json
```

### 5.2. Create mapping (optional)
  
If you don't run this command, nor any variation of it, a default mapping will be auto-created.

```
curl -XPUT "https://<your_opensearch_private_IP>:9200/oci" -H 'Content-Type: application/json' --insecure -d' 
{
  "mappings": {
    "properties": {
    "id": {"type": "integer"},
    "category": {"type": "keyword"},
    "text": {"type": "text"},
    "title": {"type": "text"},
    "url": {"type": "text"}
    }
  }
}
'
```
JORDAN
```
curl -XPUT "https://10.0.216.169:9200/oci" -H 'Content-Type: application/json' --insecure -d' 
{
  "mappings": {
    "properties": {
    "id": {"type": "integer"},
    "category": {"type": "keyword"},
    "text": {"type": "text"},
    "title": {"type": "text"},
    "url": {"type": "text"}
    }
  }
}
'
```



### 5.3. Push the data set

```
curl -H 'Content-Type: application/x-ndjson' -XPOST "https://<your_opensearch_private_IP>:9200/oci/_bulk?pretty" --data-binary @OCI_services.json --insecure
```
JORDAN
```
curl -H 'Content-Type: application/x-ndjson' -XPOST "https://10.0.216.169:9200/oci/_bulk?pretty" --data-binary @OCI_services.json --insecure
```

### 5.4. check your indices

```
curl "https://amaaaaaanlc5nbya44qen6foty3gyu7ihpo22mzmtjw5ixtcjgetjcqwipuq.opensearch.us-ashburn-1.oci.oracleiaas.com:9200/_cat/indices" --cacert cert.pem
```
### 5.4.a other operations to delete existing indicies (if needed)
```
curl -XDELETE "https://amaaaaaanlc5nbya44qen6foty3gyu7ihpo22mzmtjw5ixtcjgetjcqwipuq.opensearch.us-ashburn-1.oci.oracleiaas.com:9200/shakespeare" –insecure
```
```
curl -XDELETE "https://amaaaaaanlc5nbya44qen6foty3gyu7ihpo22mzmtjw5ixtcjgetjcqwipuq.opensearch.us-ashburn-1.oci.oracleiaas.com:9200/oci" –insecure
```


# OpenSearch API endpoint example, with certificate

JORDAN
```
curl "https://amaaaaaanlc5nbya44qen6foty3gyu7ihpo22mzmtjw5ixtcjgetjcqwipuq.opensearch.us-ashburn-1.oci.oracleiaas.com:9200/_cat/indices" --cacert cert.pem

OR 

```
curl -X GET "https://10.0.1.190:9200/_cat/indices" --insecure
# OpenSearch private IP example
```
  

## 6. Query the OCI Search Service – Sample search query

### 6.1. From the VM instance shell:
```
curl -X GET "https://amaaaaaanlc5nbya44qen6foty3gyu7ihpo22mzmtjw5ixtcjgetjcqwipuq.opensearch.us-ashburn-1.oci.oracleiaas.com:9200/oci/_search?q=title:Kubernetes&pretty" --cacert cert.pem
# OpenSearch API endpoint example, with certificate
```
JORDAN:
```
curl -X GET "https://amaaaaaakztjrmiaca3oyxmzegtn25lfh77dmc6heo65r647cc4u3qvz4ewa.opensearch.us-ashburn-1.oci.oracleiaas.com:9200/oci/_search?q=title:Kubernetes&pretty" --cacert cert.pem
# OpenSearch API endpoint example, with certificate
```

OR

```
curl -X GET "https://10.0.1.190:9200/oci/_search?q=title:Kubernetes&pretty" --insecure
# OpenSearch private IP example
```
### 6.2. From your local terminal, after port forwarding:  
```
curl -X GET "https://localhost:9200/oci/_search?q=title:Kubernetes&pretty" --insecure
```
### 6.3. From your local browser, after port forwarding:  
```
https://localhost:9200/oci/_search?q=title:Kubernetes&pretty
```
### 6.4. From your browser w/o port forwarding needed 
```
https://129.153.146.243/api/search?q=text:oci&pretty
```

Refer to OpenSearch or ElasticSearch tutorials for more on query syntax.  
  

## 7.  Connect to OCI Search Service – OpenSearch Dashboards

From your local machine, through port forwarding
(Ignore this step if you’ve executed it above and the connection is still open):
```
ssh -C -v -t -L 127.0.0.1:5601:<your_opensearch_dashboards_private_IP>:5601 -L 127.0.0.1:9200:<your_opensearch_private_IP>:9200 opc@<your_instance_public_ip> -i <path_to_your_private_key>
```
JORDAN:
```
ssh -C -v -t -L 127.0.0.1:5601:10.0.211.1:5601 -L 127.0.0.1:9200:10.0.211.1:9200 opc@129.158.214.157 -i ~/.ssh/ssh-key-2022-03-14.key
```
Access <https://localhost:5601> in a browser of your choice.  
Currently, there will be a warning of the kind "your connection is not private", depending on the browser. Choose the option which allows you to proceed anyway. After that, you should see the screen below.  

<img src=".//media/image7.png" style="width:6.26806in;height:2.85278in" />
  

## 8.  Search and visualize data in OCI Search Service - OpenSearch Dashboards

With the port forwarding connection in place, access https://localhost:5601 in your browser.  
Go to Menu &rarr; Management &rarr; Stack Management &rarr; Index Patterns and create an index pattern, with name = `oci`

<img src=".//media/image9.png" style="width:6.26806in;height:3.17014in" />

Go to Menu &rarr; Discover to use the Dashboards UI to search your data.  
  
<img src=".//media/image10.png" style="width:6.26806in;height:3.12639in" />

Go to Menu &rarr; Dashboards and follow the steps below to create a sample
pie chart.

a.  Create new &rarr; New Visualization &rarr; Pie

<img src=".//media/image11.png" style="width:6.26806in;height:3.19444in" />

b.  Choose `oci` as source

c.  In Buckets, click ‘Add’ &rarr; Split slices, provide the parameters
    as below and click ‘Update’

<img src=".//media/image12.png" style="width:6.26806in;height:3.52778in"/>


HELPFUL LINK
```
https://localhost:9200/opensearch_dashboards_sample_data_ecommerce/_search?q=*:*&pretty
```
