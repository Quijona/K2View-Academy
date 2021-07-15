# Fabric Hardening

The following steps ensure that the keys that secure Fabric and Cassandra are generated and configured.

- The example password ```1234Q!``` is used for TLS keys and can be replaced in all the following sections by a new password.
- Do not forget to replace all $K2_HOME/ values with the full and correct path location in both Fabric and Cassandra.


## Step 1 - Keys Generation

1. Run the keys script that can be downloaded from [location](https://owncloud-bkp2.s3.amazonaws.com/adminoc/Utils/Hardening/secure_cassandra.sh). 
2. Stop Fabric and Cassandra services before running the script.


```bash
cd $K2_HOME/
rm -rf .cassandra .cassandra_ssl export .oracle_jre_usage .ssl

chmod +x secure_cassandra.sh

!! run on single Fabric node only !!
* To change the password or the cluster name, edit the secure_cassandra.sh or execute using the password and cluster name parameters
e.g.: ./secure_cassandra.sh {Password} {Cluster_Name}

./secure_cassandra.sh Q1w2e3r4t5 k2tls

Warning:
The JKS Keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -deststoretype pkcs12".
Certificate stored in file </opt/apps/k2view/.cassandra_ssl/CLUSTER_k2tls_PUBLIC.cer>

Warning:
The JKS Keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -deststoretype pkcs12".
Certificate was added to keystore
[Storing /opt/apps/k2view/.cassandra_ssl/cassandra.truststore]

Warning:
The JKS Keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -deststoretype pkcs12".
Certificate stored in file </opt/apps/k2view/.cassandra_ssl/CLIENT_k2tls_PUBLIC.cer>

Warning:
The JKS Keystore uses a proprietary format. It is recommended to migrate to PKCS12 which is an industry standard format using "keytool -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore -deststoretype pkcs12".
Certificate was added to keystore
[Storing /opt/apps/k2view/.cassandra_ssl/cassandra.truststore]
Importing keystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore to /opt/apps/k2view/.cassandra_ssl/cassandra.pks12.keystore...
Entry for alias k2tls_client successfully imported.
Entry for alias k2tls_cluster successfully imported.
Import command completed:  2 entries successfully imported, 0 entries failed or cancelled
MAC verified OK
MAC verified OK 
```

The following 7 files will appear under the ```$K2_HOME/.cassandra_ssl``` directory:
- k2tls_CLIENT.key.pem
- k2tls_CLIENT.cer.pem
- cassandra.keystore
- cassandra.pks12.keystore
- cassandra.truststore
- CLIENT_k2tls_PUBLIC.cer
- CLUSTER_k2tls_PUBLIC.cer

## Step 2 - Transfer Keys and Certificates to All Cassandra and Fabric Nodes

Tar and copy them to all Cassandra and Fabric nodes in the cluster.  

See the example below: 
``` 
tar -czvf keys.tar.gz -C $K2_HOME/.cassandra_ssl .
scp keys.tar.gz fabric@10.10.10.10:/home/fabric/
mkdir -p $K2_HOME/.cassandra_ssl && tar -zxvf keys.tar.gz -C $K2_HOME/.cassandra_ssl

## for Cassandra (after file copied):
mkdir -p $INSLATT_DIR/.cassandra_ssl && tar -zxvf keys.tar.gz -C $INSLATT_DIR/.cassandra_ssl
```

## Step 3 - Cassandra YAML

1. Edit the cassandra.yaml file with the appropriate passwords and certification files.
2. Execute this as a Cassandra user on all Cassandra nodes. 

```bash
sed -i "s@internode_encryption: none@internode_encryption: all@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@key_password: cassandra@key_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@keystore: conf/.keystore*@keystore: $INSLATT_DIR/.cassandra_ssl/cassandra.keystore@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# truststore:@truststore:@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@truststore: conf/.truststore*@truststore: $INSLATT_DIR/.cassandra_ssl/cassandra.truststore@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# protocol: TLS*@protocol: TLS@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@keystore_password: cassandra@keystore_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# truststore: conf/.truststore*@truststore: $INSLATT_DIR/.cassandra_ssl/cassandra.truststore@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# truststore_password: cassandra*@truststore_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@truststore_password: cassandra*@truststore_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# protocol: TLS*@protocol: TLS@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# require_client_auth: .*@require_client_auth: true@g" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# store_type: JKS@store_type: JKS@g" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i '0,/enabled: false/! {0,/enabled: false/ s/enabled: false/enabled: true/}' $CASSANDRA_HOME/conf/cassandra.yaml
sed -i -e 's/# \(.*cipher_suites.*\)/\1/g' $CASSANDRA_HOME/conf/cassandra.yaml
sed -i -e 's/# \(.*native_transport_port_ssl:.*\)/\1/g' $CASSANDRA_HOME/conf/cassandra.yaml
sed -i '1,/cdc_enabled: false/ {0,/cdc_enabled: false/ s/cdc_enabled: false/cdc_enabled: true/}' $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@back_pressure_enabled: .*@back_pressure_enabled: true@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@native_transport_port: .*@native_transport_port: 9142@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i -e 's/# \(.*native_transport_port_ssl:.*\)/\1/g' $CASSANDRA_HOME/conf/cassandra.yaml
```

3. Restart the Cassandra service on each node: ```cassandra```


## Step 4 - Cassandra CQLSHRC
1. Edit the .cassandra/cqlshrc file using the appropriate passwords and certification files.
2. Execute this as a Cassandra user on all Cassandra nodes. 
```bash
cp $INSLATT_DIR/cassandra/conf/cqlshrc.sample $INSLATT_DIR/.cassandra/cqlshrc

sed -i "s@\;\[ssl\]@\[ssl\]@" $INSLATT_DIR/.cassandra/cqlshrc
sed -i '/^\[csv]/i factory = cqlshlib.ssl.ssl_transport_factory' $INSLATT_DIR/.cassandra/cqlshrc
sed -i "s@; certfile = .*@certfile = $INSLATT_DIR/.cassandra_ssl/k2tls_CLIENT.cer.pem@" $INSLATT_DIR/.cassandra/cqlshrc
sed -i "s@;validate = true@validate = true@" $INSLATT_DIR/.cassandra/cqlshrc
sed -i "s@;userkey = .*@userkey = $INSLATT_DIR/.cassandra_ssl/k2tls_CLIENT.key.pem@" $INSLATT_DIR/.cassandra/cqlshrc
sed -i "s@;usercert = .*@usercert = $INSLATT_DIR/.cassandra_ssl/k2tls_CLIENT.cer.pem@" $INSLATT_DIR/.cassandra/cqlshrc
sed -i "s@port = .*@port = 9142@" $INSLATT_DIR/.cassandra/cqlshrc
sed -i "s@hostname = .*@hostname = $(hostname -I |awk {'print $1'})@" $INSLATT_DIR/.cassandra/cqlshrc
```


## Step 5 - CQLSHRC Configuration
1. Edit the $K2_HOME/config/cqlshrc file with the appropriate passwords and certification files.
2. Execute this on all Fabric nodes.

```bash
sed -i '/^\[csv]/i factory = cqlshlib.ssl.ssl_transport_factory' $K2_HOME/config/cqlshrc
sed -i "s@\;\[ssl\]@\[ssl\]@" $K2_HOME/config/cqlshrc
sed -i "s@; certfile = .*@certfile = $K2_HOME/.cassandra_ssl/k2tls_CLIENT.cer.pem@" $K2_HOME/config/cqlshrc
sed -i "s@;validate = true@validate = true@" $K2_HOME/config/cqlshrc
sed -i "s@;userkey = .*@userkey = $K2_HOME/.cassandra_ssl/k2tls_CLIENT.key.pem@" $K2_HOME/config/cqlshrc
sed -i "s@;usercert = .*@usercert = $K2_HOME/.cassandra_ssl/k2tls_CLIENT.cer.pem@" $K2_HOME/config/cqlshrc
sed -i "s@port = .*@port = 9142@" $K2_HOME/config/cqlshrc
```

NOTE:
If there is no cqlshrc file under the $K2_HOME/config path, do the following and repeat the previous setup phase of the file:

```cp $K2_HOME/apps/apache-cassandra-3.11.4/conf/cqlshrc.sample $K2_HOME/config/cqlshrc```

The location of the version path may change based on the Fabric version.



## Step 6 - Config.ini Setup

1. Edit the config.ini file to support SSL and set up the WS SSL Key (Fabric only).
2. Run the following on all Fabric Nodes:

```bash
sed -i "s@WEB_SERVICE_PORT=.*@#WEB_SERVICE_PORT=3213@" $K2_HOME/config/config.ini
sed -i "s@#WEB_SERVICE_SECURE_PORT=443@WEB_SERVICE_SECURE_PORT=9443@" $K2_HOME/config/config.ini
sed -i "s@#WEB_SERVICE_CERT=.*@WEB_SERVICE_CERT=$K2_HOME\/.cassandra_ssl\/cassandra.keystore@" $K2_HOME/config/config.ini
sed -i "s@#WEB_SERVICE_CERT_PASSPHRASE=.*@WEB_SERVICE_CERT_PASSPHRASE=Q1w2e3r4t5@" $K2_HOME/config/config.ini
sed -i 's/#ENABLE_INTER_NODES_SSL=false/ENABLE_INTER_NODES_SSL=true/' $K2_HOME/config/config.ini
sed -i "s/#SECURE=false/SECURE=true/" $K2_HOME/config/config.ini
sed -i "s@#CASSANDRA_PORT=.*@CASSANDRA_PORT=9142@g" $K2_HOME/config/config.ini
sed -i "s@#PORT=9042@PORT=9142@g" $K2_HOME/config/config.ini
sed -i 's@#PORT=${cassandra.default.port}@PORT=9142@g' $K2_HOME/config/config.ini
sed -i 's@#SSL=false@SSL=true@g' $K2_HOME/config/config.ini
```

## Step 7 - JVM Options

Edit the $K2_HOME/config/jvm.options file using the appropriate passwords and certification files:

```bash
sed -i 's@#-Djavax.net.ssl.keyStore=.*@-Djavax.net.ssl.keyStore=$K2_HOME/.cassandra_ssl/cassandra.keystore@g' $K2_HOME/config/jvm.options
sed -i 's@#-Djavax.net.ssl.keyStorePassword=.*@-Djavax.net.ssl.keyStorePassword=Q1w2e3r4t5@g' $K2_HOME/config/jvm.options
sed -i 's@#-Djavax.net.ssl.trustStore=.*@-Djavax.net.ssl.trustStore=$K2_HOME/.cassandra_ssl/cassandra.truststore@g' $K2_HOME/config/jvm.options
sed -i 's@#-Djavax.net.ssl.trustStorePassword=.*@-Djavax.net.ssl.trustStorePassword=Q1w2e3r4t5@g' $K2_HOME/config/jvm.options
```

- Start the Fabric service on each node: ```K2fabric start```


## Step 8 - CQLSH & SSL

Configure the CQLSH to use the SSL flag: 

1. Copy the cqlshrc file from the $K2_HOME/config/cqlshrc path to the local $K2_HOME/.cassandra/ path on each Fabric node.

```cp $K2_HOME/config/cqlshrc $K2_HOME/.cassandra/```

   Note: Create the directory if it does not exist:
```mkdir -p $K2_HOME/.cassandra/```

2. Run the following command:
   ```cqlsh -u k2admin -p Q1w2e3r4t5 --ssl```

   Note: If the k2admin user credentials do not exist, use a valid user.


## Step 9 - Configure HTTPS for Web Services

Run the ```curl https://10.10.10.10:9443/ws -k ``` command from a Fabric node. The response should be similar to the following:
```bash
"data" : null,
"error" : "General Fabric error - <com.k2view.cdbms.exceptions.WebServiceException: method name is missing>"
```

Note: From version 6.2 and above you can access the Web Framework website using an https at this URL:
```https://10.10.10.10:9443/```

## Step 10 - Fabric Encryption Support

1. Edit fabric-server-start.sh on each node in the $K2_HOME/fabric/scripts/ folder.
2. Add the encryption flag to the ```exec $JAVA``` section.

```bash

# Set environemnt settings
. $BASEDIR/fabric-env.sh

CLASSPATH=$CLASSPATH:$FABRIC_CONF
CLASSPATH=$CLASSPATH:${ROOTDIR}/lib/fabric/*
CLASSPATH=$CLASSPATH:${ROOTDIR}/lib/provided/*
CLASSPATH=$CLASSPATH:${ROOTDIR}/ExternalJars/*
CLASSPATH=$CLASSPATH:${ROOTDIR}/lib/admin
CLASSPATH=$CLASSPATH:${ROOTDIR}/lib/olingo
CLASSPATH=$CLASSPATH:${ROOTDIR}/resources

rm -f $FABRIC_HOME/.boot_status

exec $JAVA |
        -classpath $CLASSPATH |
        -DFABRIC_HOME=${FABRIC_HOME} |
        $JVM_OPTS |
        com.k2view.fabric.boot.Boot |
        log:type=LOGBACK |
        config:jmx=true |
        fabricdb |
        cassandradriver fabricclusterid fabricsystemschema:forceAlter=true interfacemanager |
        fabriccredentials fabriccluster lutypes fabricprovider jobs webserver |
        commonarea |
        dbserve |
        clustertimecheck |
       *encryption* |

```

NOTE: 
Do not forget to add | after the **clustertimecheck** value for the new line to be taken into consideration.


3. Execute the following commands:

```bash
sed -i 's/clustertimecheck/clustertimecheck \\/' $K2_HOME/fabric/scripts/fabric-server-start.sh
sed -i -e '/clustertimecheck \\/a\        encryption\' $K2_HOME/fabric/scripts/fabric-server-start.sh
```


## Step 10 - Define Fabric Working Mode for the Master Key


### Generate Master Key Without Keystore

Set the **MASTERKEY_KEY_STORE_ENABLED** parameter of the config.ini file to **false** (default) to generate a master key without a Keystore. 


### Generate Master Key Using the Keystore
Set the **MASTERKEY_KEY_STORE_ENABLED** parameter of the config.ini file to **true**.

#### Create Keystore Directory 
After adding the encryption module to the fabric-server-start.sh module, create the Keystore folder under the k2view home directory for all nodes:
```bash
cd $K2_HOME
mkdir keystore
```

#### Run the Keytool
Run the Keytool command on the coordinator node:
``` keytool -genseckey -alias masterkey_key_name -keyalg aes -keysize 256 -storepass <password> -keystore  $K2_HOME/keystore/fabric.keystore -storetype JCEKS ```

- Copy the key to all other nodes:
``` scp $K2_HOME/keystore/fabric.keystore fabric@10.10.10.10:/$K2_HOME/keystore/ ```

#### Edit Config.ini Script
- Edit the **KEY_STORE_PASSWORD** parameter in the config.ini to the password used in the Keytool command, and enable the **KEY_STORE_LOCATION** parameter and set it to point to the correct path for all Fabric nodes:

```bash
sed -i "s@#KEY_STORE_LOCATION=.*@KEY_STORE_LOCATION=$K2_HOME/keystore/fabric.keystore@" $K2_HOME/config/config.ini
sed -i 's@#KEY_STORE_PASSWORD=.*@KEY_STORE_PASSWORD= <password>@' $K2_HOME/config/config.ini
```

### Restart Fabric Nodes
- Restart all Fabric nodes.


## Step 11 - Replace the Fabric Admin Password

1. Run the following from the console:

```bash
# create second admin user
cqlsh -uadmin -padmin --ssl

fabric on;
create user 'admin2' with password 'Q7xp8GPNmjZp' SUPERUSER;
assign role admin to user admin2;

# Alter the admin with new password
cqlsh -uadmin2 -pQ7xp8GPNmjZp --ssl 
fabric on;
ALTER USER admin WITH PASSWORD 'Q7xp8GPNmjZp' SUPERUSER;

```

2. Check the connection: 

```bash
# connect with the following
cqlsh -uadmin2 -pQ7xp8GPNmjZp --ssl
fabric on;
```

Access points examples:

-- Fabric WS will be available at:
``` https://10.10.10.10:9443/deploy ```

-- Admin:
``` https://10.10.10.10:9443/admin/gui/index.html ```



[![Previous](/articles/images/Previous.png)](/articles/99_fabric_infras/devops/02_fabric_environments.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](/articles/99_fabric_infras/devops/04_kafka_hardening.md)

