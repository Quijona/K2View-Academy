# Cassandra Hardening

The following steps ensure that the keys that secure Fabric and Cassandra are properly generated and configured.

- The example password ```Q1w2e3r4t5``` is used for TLS keys and can be replaced in all the following sections by a new password.
- Do not forget to replace all `$K2_HOME/` & `$INSTALL_DIR`  values with the full and correct path location for both Fabric and Cassandra.


## Step 1 - Keys Generation

1. Connect as **cassandra** user
2. Run the `secure_cassandra.sh` file that generate the keys. It included with our supplied psackage or can be downloaded from [here](https://owncloud-bkp2.s3.amazonaws.com/adminoc/Utils/Hardening/secure_cassandra.sh). 
3. Stop Cassandra services before running the script.
    ```bash
    ## run: 
    stop-server
    ```

    ```bash
    echo "export K2_HOME=$INSTALL_DIR" >> .bash_profile
    source ~/.bash_profile
    cd ~/
    rm -rf .cassandra .cassandra_ssl .oracle_jre_usage .ssl
    ````

    **Note:** run on single Cassandra node only. To change the password or the cluster name, edit the secure_cassandra.sh or execute using the password and cluster name parameters


    ```bash
    ./secure_cassandra.sh {Password} {Cluster_Name}
    ```

    for example: 

    ```bash
    ./secure_cassandra.sh Q1w2e3r4t5 k2tls
    ```
    output example:
    
    ```bash
    Warning:
    The JKS Keystore uses a proprietary format. It is recommended to migrate    to PKCS12 which is an industry standard format using "keytool  -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.  keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra. keystore -deststoretype pkcs12".
    Certificate stored in file </opt/apps/k2view/.cassandra_ssl/    CLUSTER_k2tls_PUBLIC.cer>
    
    Warning:
    The JKS Keystore uses a proprietary format. It is recommended to migrate    to PKCS12 which is an industry standard format using "keytool  -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.  keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra. keystore -deststoretype pkcs12".
    Certificate was added to keystore
    [Storing /opt/apps/k2view/.cassandra_ssl/cassandra.truststore]
    
    Warning:
    The JKS Keystore uses a proprietary format. It is recommended to migrate    to PKCS12 which is an industry standard format using "keytool  -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.  keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra. keystore -deststoretype pkcs12".
    Certificate stored in file </opt/apps/k2view/.cassandra_ssl/    CLIENT_k2tls_PUBLIC.cer>
    
    Warning:
    The JKS Keystore uses a proprietary format. It is recommended to migrate    to PKCS12 which is an industry standard format using "keytool  -importkeystore -srckeystore /opt/apps/k2view/.cassandra_ssl/cassandra.  keystore -destkeystore /opt/apps/k2view/.cassandra_ssl/cassandra. keystore -deststoretype pkcs12".
    Certificate was added to keystore
    [Storing /opt/apps/k2view/.cassandra_ssl/cassandra.truststore]
    Importing keystore /opt/apps/k2view/.cassandra_ssl/cassandra.keystore to    /opt/apps/k2view/.cassandra_ssl/cassandra.pks12.keystore...
    Entry for alias k2tls_client successfully imported.
    Entry for alias k2tls_cluster successfully imported.
    Import command completed:  2 entries successfully imported, 0 entries   failed or cancelled
    MAC verified OK
    MAC verified OK 
    ```
    
    The following 7 files will be created under the `$INSTALL_DIR/.cassandra_ssl` directory:
    - k2tls_CLIENT.key.pem
    - k2tls_CLIENT.cer.pem
    - cassandra.keystore
    - cassandra.pks12.keystore
    - cassandra.truststore
    - CLIENT_k2tls_PUBLIC.cer
    - CLUSTER_k2tls_PUBLIC.cer

    And a cassandra_keys.tar.gz file containing all files,will be created, to be transfered to the other nodes

    > Note the certificate and key file name contain the cluster name you have set.

## Step 2 - Transfer Keys and Certificates to All Cassandra and Fabric Nodes

Copy the priveously created file  *cassandra_keys.tar.gz* to all cassandra noe in the cluster

See the example below: 

``` bash

# copy to other nodes in case more than one node is used
# 10.10.10.10 represents IP address of another node
scp  cassandra_keys.tar.gz cassandra@10.10.10.10:/opt/apps/cassandra/

# login to every node searately and run the following command
mkdir -p $INSTALL_DIR/.cassandra_ssl && tar -zxvf cassandra_keys.tar.gz -C $INSTALL_DIR/.cassandra_ssl
```

## Step 3 - Cassandra YAML

1. Edit the `cassandra.yaml` file with the appropriate passwords and certification files.
2. Execute this as a Cassandra user on all the Cassandra nodes.
   > Replace the password in the following command with the one you set before. 

```bash
sed -i "s@internode_encryption: none@internode_encryption: all@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@key_password: cassandra@key_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@keystore: conf/.keystore*@keystore: $INSTALL_DIR/.cassandra_ssl/cassandra.keystore@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# truststore:@truststore:@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@truststore: conf/.truststore*@truststore: $INSTALL_DIR/.cassandra_ssl/cassandra.truststore@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# protocol: TLS*@protocol: TLS@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@keystore_password: cassandra@keystore_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# truststore: conf/.truststore*@truststore: $INSTALL_DIR/.cassandra_ssl/cassandra.truststore@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# truststore_password: cassandra*@truststore_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@truststore_password: cassandra*@truststore_password: Q1w2e3r4t5@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# protocol: TLS*@protocol: TLS@" $CASSANDRA_HOME/conf/cassandra.yaml
sed -i "s@# require_client_auth: .*@require_client_auth: false@g" $CASSANDRA_HOME/conf/cassandra.yaml
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
1. Edit the `.cassandra/cqlshrc` file using the appropriate passwords and certification files created earlier.
2. Execute this as a Cassandra user on all Cassandra nodes. 
> replace the key and certificate file name with the files created before.

    ```bash
    mkdir -p ~/.cassandra
    cp $INSTALL_DIR/cassandra/conf/cqlshrc.sample $INSTALL_DIR/.cassandra/cqlshrc

    sed -i "s@\;\[ssl\]@\[ssl\]@" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i '/^\[csv]/i factory = cqlshlib.ssl.ssl_transport_factory' $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "s@; certfile = .*@certfile = $INSTALL_DIR/.cassandra_ssl/k2tls_CLIENT.cer.pem@" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "s@;validate = true@validate = true@" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "105iversion = SSLv23" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "s@;userkey = .*@userkey = $INSTALL_DIR/.cassandra_ssl/k2tls_CLIENT.key.pem@" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "s@;usercert = .*@usercert = $INSTALL_DIR/.cassandra_ssl/k2tls_CLIENT.cer.pem@" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "s@port = .*@port = 9142@" $INSTALL_DIR/.cassandra/cqlshrc
    sed -i "s@hostname = .*@hostname = $(hostname -I |awk {'print $1'})@" $INSTALL_DIR/.cassandra/cqlshrc
    ```
3. You can check your configuratin by connecting to cassandra by the followiong command: (use the user & password yo udefined earlier)
   ```bash
   cqlsh -u k2admin -p Q1w2e3r4t5 --ssl
   ```

## Step 5 - Disable the default cassandra superuser

Cassandra default **superuser** is `cassandra` and it must be disabled before going to production. Before doing so, you need to create new **superusers**, one for SYSDBA, and one for Fabric connection use

1. connect to one of the Cassandra nodes console, and create 2 new **superuser's**

   ~~~bash
   echo "create user k2admin with password 'Q1w2e3r4t5' superuser;" | cqlsh -u cassandra -p cassandra $(hostname -i) 9142 --ssl
   echo "create user k2sysdba with password '3ptBF9eMSsyLrXr3' superuser;" | cqlsh -u cassandra -p cassandra $(hostname -i) 9142 --ssl
   ~~~

2. drop the `cassandra` user

   ~~~bash
   echo "drop role cassandra;" | cqlsh -u k2sysdba -p 3ptBF9eMSsyLrXr3 $(hostname -i) 9142 --ssl
   ~~~

   

[![Previous](/articles/images/Previous.png)](/articles/99_fabric_infras/devops/03_fabric_api_and_ui_hardening.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](/articles/99_fabric_infras/devops/05_connect_fabric_to_cassandra_with_tls.md)
