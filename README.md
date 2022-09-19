# Elastic-HA
install & set up elastic HA 

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
![image](https://user-images.githubusercontent.com/112536860/190978978-7c5f747b-134b-4665-8f32-70192cd5fd83.png)

## At dev tools , try to get all node info
```
GET /_nodes/_all/process
# for shot
GET /_cat/nodes?v
# OR you can use curl via terminal by example command 
curl --cacert /home/supawit/elasticsearch-8.4.1/config/certs/http_ca.crt -u elastic:Ung*_sm0s=5P5J7y+G1t https://192.168.24.43:9200/_nodes/_all/process | json_pp
```






