# vagrant-docker-solr5
(Nice?) try setting up a solr5 cloud w/ Vagrant, shell scripts and Docker only. Uses the Oracle 8 JDK, Docker images based on the Phusion images.

## Preconditions

Vagrant must be installed. 
I haven't managed it yet putting all the subsequently explained steps into a provisioning file, but at least this works fine now:

```
host> vagrant up --provider vmware_fusion
host> vagrant ssh core-solr-01 -- -A
coreos>cd /home/core/share/gwydyon/docker/
coreos>cd zookeeper && docker build -t gwydyon/zookeeper .
coreos>cd ../solr5 && docker build -t gwydyon/solr5 .
coreos>cd .. && docker-compose up
```

### TODO
Connecting with the console works, but not with a Java based client.