# Elastic-HA
install & set up elastic HA in debianOS (disable all firewall if it dosen't work)

## Download ELS
```
# Not root user
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-8.4.1-linux-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-8.4.1-linux-x86_64.tar.gz
cd elasticsearch-8.4.1/ 
```

## Firewalld for ELS
```
# for els
firewall-cmd --permanent --add-port=9200/tcp 
firewall-cmd --permanent --add-port=9300/tcp 
firewall-cmd --reload
# for kibana
firewall-cmd --permanent --add-port=5601/tcp 
firewall-cmd --reload
```

## Set vm.max_map_count 65530 to 262144
```
sysctl -w vm.max_map_count=262144
```
## Edit(if set) or add /etc/sysctl.conf
```
vm.max_map_count=262144
```

## Edit config/elasticsearch.yml (cd elasticsearch-8.4.1/) for first node
```
cluster.name: [CLUSTER_NAME] # ex. els-cluster
node.name: [NODE_NAME]       # ex. els-1 
network.host: [IP_ADDRESS]   # ex. 192.168.24.41
# optional 
node.roles: [ data, master ] # set role for node
```

## Run first elasticsearch node
```
bin/elasticsearch
```

## Useful command 
```
The generated password for the elastic built-in superuser is:
<password>

The enrollment token for Kibana instances, valid for the next 30 minutes:
<enrollment-token>

The hex-encoded SHA-256 fingerprint of the generated HTTPS CA DER-encoded certificate:
<fingerprint>

You can complete the following actions at any time:
Reset the password of the elastic built-in superuser with
'bin/elasticsearch-reset-password -u elastic'.

Generate an enrollment token for Kibana instances with
'bin/elasticsearch-create-enrollment-token -s kibana'.

Generate an enrollment token for Elasticsearch nodes with
'bin/elasticsearch-create-enrollment-token -s node'.
```

## Other elasticsearch node config before join
```
cluster.name: [CLUSTER_NAME] # same as first node ex. els-cluster
node.name: [NODE_NAME]       # ex. els-2
network.host: [IP_ADDRESS]   # ex. 192.168.24.42
# optional 
node.roles: [ data, master ] # set role for node
```

## Join other node to cluster
```
# token get from command 'bin/elasticsearch-create-enrollment-token -s node' at first node 
# this command use when join node
bin/elasticsearch --enrollment-token <token> 
```

## curl ELS node
```
# password can reset by use command 'bin/elasticsearch-reset-password -u elastic'
# --cacert are certificate path
curl --cacert [path]/elasticsearch-8.4.1/config/certs/http_ca.crt -u [username]:[password] https://[host-ipaddress]:9200
# example curl 1 node
curl --cacert /home/supawit/elasticsearch-8.4.1/config/certs/http_ca.crt -u elastic:Ung*_sm0s=5P5J7y+G1t https://192.168.24.41:9200
```

## Download Kibana
```
curl -O https://artifacts.elastic.co/downloads/kibana/kibana-8.4.1-linux-x86_64.tar.gz
curl https://artifacts.elastic.co/downloads/kibana/kibana-8.4.1-linux-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
tar -xzf kibana-8.4.1-linux-x86_64.tar.gz
cd kibana-8.4.1/ 
```

## set kibana config at config/kibana.yml
```
server.host: [IP_ADDRESS_KIBANA]  # ex. 192.168.24.44
```

## Run kibana and access kibana URL 
```
bin/kibana
```
## After run use URL to access kibana
![image](https://user-images.githubusercontent.com/112536860/190978740-5564cc93-726a-4e57-b49b-abc1c29d204c.png)



## Get token for kibana from elasticsearch cluster 
```
# use command at elasticsearch cluster 
bin/elasticsearch-create-enrollment-token -s kibana
```

## Get verification code from kibana command
user and password for login 
user = elastic
password = from command 'bin/elasticsearch-reset-password -u elastic'


## At dev tools , try to get all node info and create some index
```
GET /_nodes/_all/process
# For shot
GET /_cat/nodes?v
# OR you can use curl via terminal by example command 
curl --cacert /home/supawit/elasticsearch-8.4.1/config/certs/http_ca.crt -u elastic:Ung*_sm0s=5P5J7y+G1t https://192.168.24.43:9200/_nodes/_all/process | json_pp
# Create index
PUT /test-index
# Get data from index
GET /test-index/_search
```

## Install Logstash 
```
# apt-get
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install logstash
# yum
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
Add the following in your /etc/yum.repos.d/ directory in a file with a .repo suffix, for example logstash.repo
```
[logstash-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
Now it ready to install
```
sudo yum install logstash
```

## certificate
copy certificate from elasticsearch instance (default path : elasticsearch-8.4.1/config/certs)
```
# example copy elasticsearch http_ca.crt to logstash at path /home/supawit
scp http_ca.crt 192.168.24.45:/home/supawit
```

## example Logstash pipeline that get input from keyboard
```
input {
  stdin {
    id => "my_plugin_id"
  }
}
filter {
    # Can be empty
}
output {
elasticsearch {
        hosts => ["https://192.168.24.41:9200","https://192.168.24.42:9200","https://192.168.24.43:9200"]

        # SSL enabled
        ssl => true
        ssl_certificate_verification => true

        # Path to your Cluster Certificate
        cacert => "/home/supawit/http_ca.crt"

        # The Logstash Username and Password created Earlier
        user => "elastic"
        password => "Supawit0"
        # The name of the Index
        index => "test-index"
        #ilm_enabled => false
        }
}
```
## Get input from filebeat
```
input {
  beats {
    port => 5044
  }
}
filter {
}
output {
elasticsearch {
        hosts => ["https://192.168.24.41:9200","https://192.168.24.42:9200","https://192.168.24.43:9200"]
        ssl => true
        ssl_certificate_verification => true
        cacert => "/home/supawit/http_ca.crt"
        user => "elastic"
        password => "Supawit0"
        index => "test-index"

```

## Run logstash
```
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/filebeat.conf
```

## Install beat
```
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.4.1-linux-x86_64.tar.gz
tar xzvf filebeat-8.4.1-linux-x86_64.tar.gz
```

## config firebeat.yaml
```
filebeat.inputs: 
- type: log
  paths:
    - /home/supawit/logstash-tutorial.log

output.logstash:
  # The Logstash hosts
  hosts: ["192.168.24.45:5044"]
```

## Run filebeat
```
./filebeat -e -c filebeat.yml -d "publish"
```

## Test update file-log
```
echo hello-world >> /home/supawit/logstash-tutorial.log
```

## Use filebeat in k8s
```
# Download filebeat yaml
curl -L -O https://raw.githubusercontent.com/elastic/beats/8.4/deploy/kubernetes/filebeat-kubernetes.yaml 
```
## Config filebeat-kubernetes.yaml
```
# In this case out put will sent to logstach
output.logstash:
  # The Logstash hosts
  hosts: ["192.168.24.45:5044"]
```

## After config Let apply and see the result
