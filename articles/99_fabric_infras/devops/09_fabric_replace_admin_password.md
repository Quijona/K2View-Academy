# Fabric - Replace the Fabric Defulte Admin Password



## On a fresh new cluster  

on the first Fabric node use the `adminInitialCredentials` file for seting the user and password for the Fabric "admin" user. The file will be read when Fabric start for the first time, and then deleted

bellow is example:

~~~bash
cp config/adminInitialCredentials.template config/adminInitialCredentials
sed -i 's@user.*@k2consoleadmin/KW4RVG98RR9xcrTv@' config/adminInitialCredentials
~~~

Now you can start Fabric, and use the following user and password example:

`user: k2consoleadmin` 

`password: KW4RVG98RR9xcrTv` 

## On Running Cluster 

### Via the Web Admin UI



### Via console

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



[![Previous](/articles/images/Previous.png)](/articles/99_fabric_infras/devops/08_oracleGG_hardening.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](/articles/99_fabric_infras/devops/10_fabric_definde_master_key.md)
