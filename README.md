# vagrant-docker-solr5
(Nice?) try setting up a solr5 cloud w/ Vagrant, shell scripts and Docker only. Uses the Oracle 8 JDK, Docker images based on the Phusion images.

## Preconditions

Vagrant must be installed. 
Haven't finished yet putting all the subsequently explained steps into a provisioning file, but at least this works fine now:

```
host> vagrant up --provider vmware_fusion
host> vagrant ssh core-solr-01 -- -A
// coreos>cd /home/core/share/gwydyon/docker/
// coreos>cd zookeeper && docker build -t gwydyon/zookeeper .
// coreos>cd ../solr5 && docker build -t gwydyon/solr5 .
// coreos>cd .. &&  /home/core/share/gwydyon/dc/docker-compose up
coreos>cd /home/core/share/gwydyon/docker/ && cd zookeeper && docker build -t gwydyon/zookeeper . && cd ../solr5 && docker build -t gwydyon/solr5 . && cd .. &&  /home/core/share/gwydyon/dc/docker-compose up
coreos>docker exec -i -t docker_solr1_1 /opt/solr/bin/solr create_collection -c gwydyon_collection -shards 3 -replicationFactor 2 -p 8983
coreos>docker exec -i -t docker_solr1_1 /opt/solr/server/scripts/cloud-scripts/zkcli.sh -zkhost $(docker inspect --format='{{.NetworkSettings.IPAddress}}' docker_zookeeper_1):2181 -cmd upconfig -confdir /opt/gwydyon/configsets/common/conf -confname common
```

Hint: For removing the Docker containers, use ```docker rm -f $(docker ps -aq)```

### TODO
  * Connecting with the console works, but not with a Java based client.
    Need to fetch the IP address via 
    ```docker inspect --format='{{.NetworkSettings.IPAddress}}' docker_zookeeper_1```

  * Strange problem: Connection to ZK is possible via
    ```.../zookeeper-3.4.6/bin/zkCli.sh -server localhost:2181```
    but not using the Java client. Need to check the code.
