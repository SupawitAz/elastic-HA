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

## 

