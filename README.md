# Data Streaming of Servitel Message 
* Data-Streaming for Servitel Message PoC (Ver. Alfa).
* Parse the Mes File as per schema
* AvroProducer for Servitel message to the confluent kafka broker
    
#### Prerequisites:
* Create topic with name as `sv_ingress_topic`
	#
		sudo docker run --rm confluentinc/cp-kafka:3.1.1 kafka-topics --create --topic sv_ingress_topic --partitions 1 --replication-factor 1 --if-not-exists --zookeeper <ip:port>
* Map the data-sources (Mes file over the azure or local folder or some services) to `'/app/mes-files'` folder (inside working directory) while running the script.
   
* URL format for schema registry while POST   		
	>  http://<ip-address schema worker/docker container>:<port>/subjects/rmp-key/versions

* Use below format to pass the value of 'SCHEMA_CONTAINER_NAME' in command while running script.
   	> 'http://<ip-address:port>'

# Getting Started:
## Local deploy ##

1.	Installation process
   	> git clone [http://devsdb.visualstudio.com/_git/SDB-IOEE_StreamProcessingPoC](http://devsdb.visualstudio.com/_git/SDB-IOEE_StreamProcessingPoC)
   	
2.	Software dependencies and building docker image 
    `docker build -t <image-name> .`
3.  POST the schema over schema registry :
	#
		curl -X POST -i -H "Content-Type: application/json" \--data '{"schema":"{\"name\":\"rmpSchema\",\"type\":\"record\",\"doc\":\"Basic schema for controller data\",\"namespace\":\"schindler.rmp.controller\",\"fields\":[{\"name\":\"attended\",\"doc\":\"Control Center is attended manned\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"unattended\",\"doc\":\"Control Center is unattended unmanned\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Module\",\"doc\":\"Lift Module Identification unit Id\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"TSEQP\",\"doc\":\"Datetime for a message originated at the equipment side\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"TSUTC\",\"doc\":\"Datetime for a message where the origin is not relevant and time is provided in UTCGMT\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"TSKG\",\"doc\":\"Datetime for a message originated at the KG Control Center side\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Subcode\",\"doc\":\"Subcode type\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Parameter\",\"doc\":\"Meta data from controller\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Code\",\"doc\":\"code value\",\"type\":[\"string\",\"float\",\"null\"]}]}"}' \http://<ip-address schema worker/docker container>:<port>/subjects/rmpkey/versions
    **Notes**: follow the format while post over schema server: `http://<ip_address>:<port>` 
4. Run the script to see controller data locally
   #
    	docker run -ti --rm -v <absolute path to working dir:/sv-ingress> -w <working-dir name on docker container> -e KAFKA_CONTAINER_NAME=name:port -e SCHEMA_CONTAINER_NAME='http://<ip-address:port>' --network vagrant_default --name rmp_container  <image name> python data-stream.py

	**Example:**
	#
		docker run -ti --rm -v /vagrant/sv_ingress:/sv_ingress -w /sv_ingress -e KAFKA_CONTAINER_NAME=kafka_avro:29092 -e SCHEMA_CONTAINER_NAME='http://schemareg:8081' --network vagrant_default --name rmpcontainer rmp_poccontainer python data-stream.py

## Deploy to Azure cluster
1. ssh with private key on virtual machine `refer repo for ip address`
2. git clone [http://devsdb.visualstudio.com/_git/SDB-IOEE_StreamProcessingPoC](http://devsdb.visualstudio.com/_git/SDB-IOEE_StreamProcessingPoC)

3.	Software dependencies and building docker image 
    `docker build -t <image-name> .`
    
4. POST the schema over schema registry
   #
    
		curl -X POST -i -H "Content-Type: application/json" \--data '{"schema":"{\"name\":\"rmpSchema\",\"type\":\"record\",\"doc\":\"Basic schema for controller data\",\"namespace\":\"schindler.rmp.controller\",\"fields\":[{\"name\":\"attended\",\"doc\":\"Control Center is attended manned\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"unattended\",\"doc\":\"Control Center is unattended unmanned\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Module\",\"doc\":\"Lift Module Identification unit Id\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"TSEQP\",\"doc\":\"Datetime for a message originated at the equipment side\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"TSUTC\",\"doc\":\"Datetime for a message where the origin is not relevant and time is provided in UTCGMT\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"TSKG\",\"doc\":\"Datetime for a message originated at the KG Control Center side\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Subcode\",\"doc\":\"Subcode type\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Parameter\",\"doc\":\"Meta data from controller\",\"type\":[\"string\",\"float\",\"null\"]},{\"name\":\"Code\",\"doc\":\"code value\",\"type\":[\"string\",\"float\",\"null\"]}]}"}' \http://<ip-address schema worker/docker container>:<port>/subjects/rmpkey/versions

5. Run the script to see controller data running over azure cluster,
   #
   	sudo docker run -ti --rm -v <path of data-source files/folder to be mapped>:/<work dir>/mes-files -w /app -e KAFKA_CONTAINER_NAME= <ip-adress:port> -e SCHEMA_CONTAINER_NAME='http://<ip-address:port>' --name <container name> <image name> python data-stream.py

	**Example:**
	#
		sudo docker run -ti --rm -v ~/mesFiles:/sv-ingress/mes-files -v /home/ioeeadmin/SDB-IOEE_StreamProcessingPoC/sv_ingress:/sv_ingress -w /sv_ingress -e KAFKA_CONTAINER_NAME=10.0.1.7:9092 -e SCHEMA_CONTAINER_NAME='http://10.0.1.6:8081' --name rmpcontainer sv_image python data-stream.py
	
## Miscellaneous 
   
###### To test the Broker
1. Create topic 
   sudo docker run --rm confluentinc/cp-kafka:3.1.1 kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper 10.0.1.7:2181
2. Docker  Producer Image  
   sudo docker run --rm confluentinc/cp-kafka:3.1.1 bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list 10.0.1.7:9092 --topic foo && echo 'Produced 42 messages.'"
3. Docker Consumer Image 
   docker run --rm confluentinc/cp-kafka:3.1.1 kafka-console-consumer --bootstrap-server 10.0.1.7:9092 --topic foo --new-consumer --from-beginning --max-messages 42
4. Get a list of topics
   $ curl "http://<broker ip:port>/topics"
5. Get info about one <sv_ingress_topic> topic
   $ curl "http://<brokerip:port>/topics/sv_ingress_topic"
6. Verify the topic 
    sudo docker run --rm confluentinc/cp-kafka:3.3.0-SNAPSHOT kafka-topics --describe --topic sv_ingress_topic --zookeeper <ip:port>
7. Delete topic
   bin/kafka-topics.sh --zookeeper zk_host:port/chroot --delete --topic my_topic_name
8. For more info about confluent
   http://docs.confluent.io/1.0.1/quickstart.html

9. Login to Container
#  
	1. docker run -ti --rm -v <absolute path to working dir:/sv-ingress> -w <working-dir name on docker container> -e KAFKA_CONTAINER_NAME=name:port -e SCHEMA_CONTAINER_NAME='http://<ip-address:port>' --network vagrant_default --name rmp_container  <image name> bash                     or
    2. or sudo docker exec -i -t <container name> /bin/bash

## Contribute
Source Code: [http://devsdb.visualstudio.com/_git/SDB-IOEE_StreamProcessingPoC](http://devsdb.visualstudio.com/_git/SDB-IOEE_StreamProcessingPoC)
## Author
Naweli Bharati: naweli.bharti@gmail.com



