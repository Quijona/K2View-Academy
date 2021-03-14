# **Fabric Web Services Security** 

## Tokens

Fabric uses an Authentication mechanism that secures Web Service access to exposed data like in LUI or reference tables. The following credentials are required for authenticating Web Service calls:
- Unsecured API key.  
- Secured API key based on the JSON Web Tokens (JWT) solution. 

JWT is an open industry standard method (RFC 7519) that securely represents claims between two parties. 

API keys can be used in two modes:

- Unsecured mode, signed by Fabric.
- Secured mode, using a digital signature on the client side. The secret key is shared as an output only once to be used by the client to generate the digital signature.


Fabric supports backward capability via token authentication and an enhanced Create Token command for secured tokens. 


### JWT Tokens Generation 

To generate a JWT token using the Fabric Authenticate API, do the following:

1. Open Postman.
2. Select the POST method and enter the Fabric server's address: http://localhost:3213/api/authenticate in the URL window.
3. In the **Body** tab, enter the **username** and **password** as keys and then enter the corresponding values, for example; admin/admin. 
4. Follow the above steps to define an API key.
5. Click **SEND**.
6. Open the **Cookies** tab next to the response body, the API key is displayed in the Value field. 

For example: 

             eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0b2tlbiIsImlzcyI6ImZiciIsImlhdCI6MTYwNjY2MDg4MiwiZXhwIjoxNjA2NjYxNzgyLCJ1bm0iOiJhZG1pbiJ9.sQpH343SbfLPHrR7lp5eG4qZKGXXhMrkggX9wqVzLBQ

<img src="/articles/26_fabric_security/images/05_devop-prodEnv_PostMAN.png">
    



## Tokens and the Admin Panel

To generate a new token for accessing Fabric API, do the following: 

1. Open the **Admin Panel** web page and select **Admin**, **Security** and then click the **API keys** tab.

2. Click the **Add API Key +** button on the upper right of the window.

3. Fill in the following details:
  - Name (Mandatory).
  - Secured (Optional).
4. Click  **Save**.
 
When the secured option has been selected, the secret key is displayed in a pop-up window and can be copied.
For example:
```f151c40f-fede-4fb3-8010-398ffbc02329```


<img src="/articles/26_fabric_security/images/07_fabric_webToken.PNG">

Note that if the secured option has not been selected,  the Token Name is used as the token value.

To create a the JWT key based on a secured API key, continue with the following steps:

5. Open Postman to generate the JWT key from a non-secured token:
    - Open a new request
    - Select the body field:
        - Enter a unsecured API key token: ```apikey``` with value set to ```1234``` (create token 1234 as non secure token in the admin API panel)
    - Select the POST method and set the URL to: ```http://localhost:3213/api/authenticate```
    - Send the request, and check the request has been validated in the response body.
    - Click on the *cookies* under the *send* button
    - Copy/Paste the JWT token ```eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0b2tlbiIsImlzcyI6ImZiciIsImlhdCI6MTYxNTczMTU5NSwiZXhwIjoxNjE1NzMyNDk1LCJhcGsiOiJncmVnIn0.5-L8D_jKe_cUzl98mtf1XYxJcq28IMcqrzw6SAN1i34```
 
 6. Go to JWT.io to replace the JWT key with the secured API key created in step 3.
    - In the "decoded" side of the window (right panel), replace the "apk" value with the name of the secured token you created in the Admin Panel in step 3 of the process described above.
    
    - in the *verify signature* window below, replace the ```your-256-bit-secret``` with the value of the secret key created in step 3: ```f151c40f-fede-4fb3-8010-398ffbc02329```
        - check the ```secret base64 encoded``` box
    
    - Copy the key as shown in the *Encoded* box, on the left: ```eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0b2tlbiIsImlzcyI6ImZiciIsImlhdCI6MTYxNTczMjg5NCwiZXhwIjoxNjE1NzMzNzk0LCJhcGsiOiJyb25yb24ifQ.GBl8Lu7U7RED7lPdDDhr8mzV6h4m9Q3ForBrQoZxkcM```
    
    - Go back to postman and replace the jwt key in the cookies management window with the key copied from the step above
 




## Tokens from the Command Line

For information about creating a token from the command line, click [here](/articles/17_fabric_credentials/02_fabric_credentials_commands.md#create-token).

## Granting Web Service Permissions to Roles 

When assigning a role to a user, different types of methods can be attributed. 

Read this [article](/articles/17_fabric_credentials/01_fabric_credentials_overview.md#rbac-in-fabric) for the list of supported roles, and then click [here](/articles/17_fabric_credentials/02_fabric_credentials_commands.md#grant-ws_name-to-role-) to learn how to grant permissions to specific roles.


### Web Service Authorization Using the API Key 

#### Project Web Services

Permissions can be granted to a role in a Web Service or in all Web Services. An API key is assigned to the role using the ```GRANT <ws_name> TO <ROLE>``` command line.


#### Product Web Services

The API key is assigned to a user. Permissions for product Web Services are defined by combining the API key assigned to the user and the permissions of the roles assigned to the user.

Example:

``` 
    create user 'greg';
    create role 'writeRole';
    grant WRITE on * to 'writeRole';
    assign role 'writeRole' to user 'greg';
    create token 'test_token' user 'greg';
```

This snipet shows how the WRITE permission granted to the writeRole has been assigned to the user; and how the test_token token reflecting this role/permission has been generated for the user.

When trying to invoke the Web Service with the DELETE verb using the 'test_token' token, Fabric throws the following error since the delete permission has not been granted to the specific token: 

``` "Com.k2view.cdbms.exceptions.UnauthorizedException: test_read is not allowed to perform [DELETE_INSTANCE]" ```






[![Previous](/articles/images/Previous.png)](/articles/26_fabric_security/04_fabric_interfaces_security.md)[<img align="right" width="60" height="54" src="/articles/images/Next.png">](/articles/26_fabric_security/06_data_masking.md)

