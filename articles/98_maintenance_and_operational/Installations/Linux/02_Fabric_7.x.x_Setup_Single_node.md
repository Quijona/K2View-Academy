# Fabric 7.x.x Setup Single Node

**Note** Fabrix 7.x.x required JDK 17 (included in the package), However, the certifed version of Cassandra and Kafka required JDK 8.* Package ( latest tested version included with each supplied Package)

## Setup Cassandra

### Load The Package 

1.  Retrieve the latest Cassandra package (located at: [latest version](https://download.k2view.com/index.php/s/JCaRQITovgh6mcz)).

2.  Connect to the Linux execution server as "cassandra" user and copy the package to the home directory.

3. Untar the package (the package name varies based on the version) as follows:

   ~~~bash
   tar -zxvf k2v_cassandra-3.11.xxx.tar.gz
   ~~~

4. Updated the .bash_propile to use python 2.7:

   ~~~bash
   sed -i '11i\alias python='/usr/bin/python2.7'\' ~/.bash_profile
   source ./.bash_profile
   # verified the Python version 
   python --version
   ~~~


### Setup Single Node Cassandra

Run the commands as shown below.

1.  Run the pre setup commands.

2.  Start Cassandra.

3.  Run the post setup commands.

**Pre Setup Configuration**:

Update the following parameters if needed:

- `dc=`
- `cluster_name:` 

~~~bash
sed -i "s@INSTALL_DIR=.*@INSTALL_DIR=$(pwd)@" .bash_profile
bash -l
rm -rf  cassandra/data && ln -s /opt/apps/cassandra/storage/  cassandra/data
sed -i 's@dc=.*@dc=DC1@' $INSTALL_DIR/cassandra/conf/cassandra-rackdc.properties
sed -i 's@cluster_name: .*@cluster_name: 'integration'@' $INSTALL_DIR/cassandra/conf/cassandra.yaml
sed -i s/seeds:.*/"seeds: $(hostname -I |awk {'print $1'})"/g $INSTALL_DIR/cassandra/conf/cassandra.yaml
sed -i s/listen_address:.*/"listen_address: $(hostname -I |awk {'print $1'})"/g $INSTALL_DIR/cassandra/conf/cassandra.yaml
sed -i s/broadcast_rpc_address:.*/"broadcast_rpc_address: $(hostname -I |awk {'print $1'})"/g $INSTALL_DIR/cassandra/conf/cassandra.yaml
sed -i 's@endpoint_snitch:.*@endpoint_snitch: GossipingPropertyFileSnitch@' $INSTALL_DIR/cassandra/conf/cassandra.yaml
sed -i 's@LOCAL_JMX=.*@LOCAL_JMX='no'@' $INSTALL_DIR/cassandra/conf/cassandra-env.sh
sed -i "s@-Djava.rmi.server.hostname=.*@-Djava.rmi.server.hostname=$(hostname -I |awk {'print $1'})\"@" $INSTALL_DIR/cassandra/conf/cassandra-env.sh
sed -i "s@-Dcom.sun.management.jmxremote.password.file=.*@-Dcom.sun.management.jmxremote.password.file=/opt/apps/cassandra/cassandra/conf/.jmxremote.password\"@" $INSTALL_DIR/cassandra/conf/cassandra-env.sh
echo "monitorRole QED" >> ~/cassandra/conf/.jmxremote.password
echo "controlRole R&D" >> ~/cassandra/conf/.jmxremote.password
echo "cassandra cassandra" >> ~/cassandra/conf/.jmxremote.password
echo "k2view Q1w2e3r4t5" >> ~/cassandra/conf/.jmxremote.password
chmod 400 ~/cassandra/conf/.jmxremote.password

~~~

### Start Cassandra

~~~bash
cassandra
~~~

### Post Setup on One Node

Create new superuser for Cassandra, and change the **cassandra** default user's password:

~~~bash
echo "create user k2admin with password 'Q1w2e3r4t5' superuser;" |cqlsh -u cassandra -p cassandra
echo "ALTER user cassandra with PASSWORD 'ZBU3Ld35NvXU3qud' superuser;" |cqlsh -u k2admin -p Q1w2e3r4t5
~~~

**Note**: if you select to change the password from the example above, note that you will need to update it later in point that you preconfigure the Fabric. We refer the the following SED lines:


~~~bash
sed -i "s@#PASSWORD=.*@PASSWORD=Q1w2e3r4t5@" $K2_HOME/config/config.ini
sed -i "s@#PASSWORD=.*@PASSWORD=Q1w2e3r4t5@" $K2_HOME/config/iifConfig.ini
~~~

## Setup Kafka

### Load the Package 

1.  Download the package from: [latest version](https://download.k2view.com/index.php/s/PTswLot7a21eFET).

2.  Connect to Linux as "kafka" user and copy the package to the home directory.

3. Untar the package (the package name varies based on the version) as follows:

   ~~~bash
   tar -zxvf k2view_Confluent_7.xxx.tar.gz && bash -l
   ~~~

4. Pre setup:

   ~~~bash
   export kserver1=$(hostname -I |awk {'print $1'})

   if [ "$(hostname -I |awk {'print $1'})" == "$kserver1" ]; then echo 1 > $K2_HOME/zk_data/myid; fi
   if [ "$(hostname -I |awk {'print $1'})" == "$kserver1" ]; then sed -i "s@broker.id=.@broker.id=1@" $CONFLUENT_HOME/server.properties ; fi

   sed -i "s@log.retention.minutes=.*@log.retention.hours=48@" $CONFLUENT_HOME/server.properties
   sed -i "s@advertised.listeners=.*@advertised.listeners=PLAINTEXT:\/\/$(hostname -I |awk {'print $1'}):9093@" $CONFLUENT_HOME/server.properties
   sed -i "s@advertised.host.name=.*@advertised.host.name=PLAINTEXT:\/\/$(hostname -I |awk {'print $1'}):9093@" $CONFLUENT_HOME/server.properties
   sed -i "s@listeners=PLAINTEXT:\/\/.*@listeners=PLAINTEXT:\/\/$(hostname -I |awk {'print $1'}):9093@" $CONFLUENT_HOME/server.properties
   sed -i "s@zookeeper.connect=.*.@zookeeper.connect=$kserver1:2181@" $CONFLUENT_HOME/server.properties
   echo "default.replication.factor=3" >> $CONFLUENT_HOME/server.properties
   sed -i "s@log.dirs=.*@log.dirs=$K2_HOME/zk_data@" $CONFLUENT_HOME/server.properties
   sed -i "s@dataDir=.*@dataDir=$K2_HOME/zk_data@" $CONFLUENT_HOME/zookeeper.properties
   sed -i "s@server.1=.*@#server.1=$(hostname -I |awk {'print $1'}):2888:3888@" $CONFLUENT_HOME/zookeeper.properties
   echo "server.1=$kserver1:2888:3888" >> $CONFLUENT_HOME/zookeeper.properties

   ~~~

5. Start Kafka and Zookeeper:

   ~~~bash
   $K2_HOME/kafka/bin/zookeeper-server-start -daemon $K2_HOME/kafka/zookeeper.properties
   sleep 10
   $K2_HOME/kafka/bin/kafka-server-start -daemon $K2_HOME/kafka/server.properties
   ~~~

6. Verify the Kafka and Zookeeper are running:

   ~~~bash
   $CONFLUENT_HOME/bin/zookeeper-shell localhost:2181 <<< "ls /brokers/ids"
   ~~~


## Setup for Fabric & TDM 7

### Load the Package 

1.  Download the package from the links that were provided to you.

2.  Untar the package:
   ~~~bash
   tar -zxvf [package name].tar.gz 
   ~~~

3. Pre setup:

   ~~~bash
   sed -i "s@K2_HOME=.*@K2_HOME=$(pwd)@" .bash_profile
   bash -l 
   ~~~

   - Presumption: Cassandra, Kafka and Fabric are running on the same node. If not, you have to update the IP's for: **cserver1** and  **kserver1**. 


   ~~~bash
   # Cassandra IP
   export cserver1=$(hostname -I |awk {'print $1'})
   # Kafka IP
   export kserver1=$(hostname -I |awk {'print $1'})
   cp -r $K2_HOME/fabric/config.template $K2_HOME/config

   # update heap memory if you have minimume of 32G
   ## sed -i 's@-Xmx2G@-Xmx8G@' $INSTALL_DIR/config/jvm.options
   ## sed -i 's@-Xms2G@-Xms8G@' $INSTALL_DIR/config/jvm.options

   cp config/adminInitialCredentials.template config/adminInitialCredentials
   sed -i 's@user.*@k2consoleadmin/KW4RVG98RR9xcrTv@' config/adminInitialCredentials

   sed -i 's@#REPLICATION_OPTIONS=.*@REPLICATION_OPTIONS={ '"'"'class'"'"' : '"'"'NetworkTopologyStrategy'"'"', '"'"DC1"'"' : 1}@' $K2_HOME/config/config.ini
   sed -i "s@#HOSTS=.*@HOSTS=$cserver1@" $K2_HOME/config/config.ini
   sed -i "s@#USER=.*@USER=k2admin@" $K2_HOME/config/config.ini
   ~~~

   - In case of using Kafka + iidFinder, run also the following commands:

   ~~~bash
   sed -i "s@#PASSWORD=.*@PASSWORD=Q1w2e3r4t5@" $K2_HOME/config/config.ini
   sed -i "s@#MESSAGES_BROKER_TYPE=.*@MESSAGES_BROKER_TYPE=KAFKA@" $K2_HOME/config/config.ini
   sed -i "s@#BOOTSTRAP_SERVERS=.*@BOOTSTRAP_SERVERS=$kserver1:9093@" $K2_HOME/config/config.ini
   sed -i "s@HOSTS=.*@HOSTS=$cserver1@" $K2_HOME/config/iifConfig.ini
   sed -i "s@#USER=.*@USER=k2admin@" $K2_HOME/config/iifConfig.ini
   sed -i "s@#PASSWORD=.*@PASSWORD=Q1w2e3r4t5@" $K2_HOME/config/iifConfig.ini
   sed -i "s@#KAFKA_BOOTSTRAP_SERVERS=.*@KAFKA_BOOTSTRAP_SERVERS=$kserver1:9093@" $K2_HOME/config/iifConfig.ini
   sed -i "s@#ZOOKEEPER_BOOTSTRAP_SERVERS=.*@ZOOKEEPER_BOOTSTRAP_SERVERS=$kserver1:2181@" $K2_HOME/config/iifConfig.ini
   sed -i 's@#IIF_REPLICATION_OPTIONS=.*@IIF_REPLICATION_OPTIONS={ '"'"'class'"'"' : '"'"'NetworkTopologyStrategy'"'"', '"'"DC1"'"' : 1}@' $K2_HOME/config/iifConfig.ini
   sed -i "s@#BOOTSTRAP_SERVERS=.*@BOOTSTRAP_SERVERS=$kserver1:9093@" $K2_HOME/config/iifConfig.ini
   ~~~

:notebook_with_decorative_cover: **Note** that the Fabric Admin user is defined now as **k2consoleadmin**.

4. Start Fabric:

   ~~~bash
   k2fabric start && k2fabric status
   ~~~

5. Connect to the Fabric console with:

   ~~~bash
   fabric -u k2consoleadmin -p KW4RVG98RR9xcrTv
   ~~~

   - Same user and password should be used for login to the Web Framework.

## PGSQL 

TDM 7.xx is certified with PGSQL 9.6 & 13. You can supply access to Existing PG if you have one.
TDM requires user & password with full **create**, **delete** and **update** privileges. 

The customer can provide the **PGSQL**, or find below the installation instructions for **K2view** **PGSQL**:

<ul>      
<li>
<a href="/articles/98_maintenance_and_operational/Installations/Linux/PGSQL_setup.md">Setup PGSQL 13.3</a></li>



[![Previous](/articles/images/Previous.png)](01_Fabric_7.xx_Installation_intro.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](003_Fabric_7.x.x_Setup_Single_DC_multi_nodes.md)  
