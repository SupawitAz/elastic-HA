# elastic-HA
install elastic HA 

# Download els
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.1-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-8.4.1-linux-x86_64.tar.gz.sha512 
tar -xzf elasticsearch-8.4.1-linux-x86_64.tar.gz
cd elasticsearch-8.4.1/ 
```

# Firewalld
firewall-cmd --permanent --add-port=9200/tcp 
firewall-cmd --permanent --add-port=9300/tcp 
