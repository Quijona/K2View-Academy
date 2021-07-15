# Fabric API/WSHardening 

## Step 1 - Keys Generation

1. Run the following scrip on one of the fabric's

~~~bash
mkdir -p $K2_HOME/.ssl

keytool -genkey -noprompt \
        -keyalg RSA \
        -alias selfsigned \
        -alias alias1 \
        -dname "CN=k2view.com, OU=SU, O=k2view, L=k2v, S=k2v, C=TLV" \
        -keystore $K2_HOME/.ssl/k2vws.key \
        -storepass Q1w2e3r4t5 \
        -validity 760 \
        -keysize 4096 \
        -keypass Q1w2e3r4t5
~~~

## Step 2 - Copy the Key to all Fabric nodes

1. copy the `$K2_HOME/.ssl/k2vws.key` to all the Fabric nodes to the same location

``` bash
tar -czvf k2vmws.tar.gz -C $K2_HOME/.ssl .
scp k2vmws.tar.gz fabric@10.10.10.10:/opt/apps/fabric/
mkdir -p $K2_HOME/.ssl && tar -zxvf k2vmws.tar.gz -C $K2_HOME/.ssl
```

## Step 3 - Config the Fabric for using TLS for API/WebUI

1. run the bellow on all the fabric nodes, and restart the Fabric service if needed


```bash
sed -i "s@WEB_SERVICE_CERT=@#WEB_SERVICE_CERT=@" $K2_HOME/config/config.ini
sed -i "s@WEB_SERVICE_PORT=@#WEB_SERVICE_PORT=@" $K2_HOME/config/config.ini
sed -i "s@#WEB_SERVICE_SECURE_PORT=443@WEB_SERVICE_SECURE_PORT=9443@" $K2_HOME/config/config.ini 
sed -i "s@WEB_SERVICE_CERT=@#WEB_SERVICE_CERT=@" $K2_HOME/config/config.ini 
sed -i "s@WEB_SERVICE_CERT_PASSPHRASE=@#WEB_SERVICE_CERT_PASSPHRASE=@" $K2_HOME/config/config.ini 
sed -i '91s/.*/WEB_SERVICE_CERT=\/usr\/local\/k2view\/.ssl\/k2vws.key/g' $K2_HOME/config/config.ini
sed -i '93s/.*/WEB_SERVICE_CERT_PASSPHRASE=Q1w2e3r4t5/g' $K2_HOME/config/config.ini
echo "ENABLE_INTER_NODES_SSL=true" >> $K2_HOME/config/config.ini 
chown fabric.fabric $K2_HOME/.ssl/k2vws.key
```




## Step 4 - Check access via HTTPS to API/WebUI

- Restart all Fabric nodes.
- Access points examples: 
  - Admin:
    ``` https://10.10.10.10:9443/ ```
  - Fabric WS will be available at:
    ``` https://10.10.10.10:9443/deploy ```

[![Previous](/articles/images/Previous.png)](/articles/99_fabric_infras/devops/02_fabric_environments.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](
