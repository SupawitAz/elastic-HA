# Elastic-HA
install & set up elastic HA 

## Download els
```
# Not root user
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-8.4.1-linux-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-8.4.1-linux-x86_64.tar.gz
cd elasticsearch-8.4.1/ 
```

## Firewalld for els
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
```

## Run first elasticsearch node
```
./bin/elasticsearch
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
```

## Join other node to cluster
```
# token get from command 'bin/elasticsearch-create-enrollment-token -s node' at first node 
# this command use when join node
bin/elasticsearch --enrollment-token <token> 
```

## Join other node to cluster
