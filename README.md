# micropass-oauth2-keycloak-connector

## Securing REST API using Keycloak and Spring Oauth2

Keycloak is Open Source Identity and Access Management Server, which is a OAuth2 and OpenID Connect(OIDC) protocol complaint. This article is to explain how Spring Boot REST APIs can be secured with Keycloak using Spring OAuth2 library.

Keycloak documentation suggest 3 ways to secure Spring based REST APIS.

* Using Keycloak Spring Boot Adapter
* Using keycloak Spring Security Adapter
* Using OpenID Connect (OIDC)+ OAuth2

Let us see how we can use Keycloak OIDC support and Spring OAuth2 library to secure REST APIs. Benefits Of Using Spring OAuth2 Over Keycloak Adapter is explained at the end of this article.

Let us explore how to setup Keycloak and interact with it using Spring OAuth2 library.
This is a lengthy article with step by step instructions, screenshots, code snippets. Complete code is available on github. I recommend reading this article before looking into the code.

## Step 1: Getting Started With Keycloak
Refer Keycloak getting started documentation to run and setup keycloak admin user.

After running Keycloak, access keycloak admin console using 
http://localhost:8080/auth

Setup keycloak username=admin, password=admin.

**Note**: Standalone Keycloak runs on Wildfly server. Don’t worry about configuring a user to manage Wildfly server. We need a Keycloak admin user to create realm, client, user, role etc in Keycloak.

## Step 2: Create dev Realm
Name: dev

![](static/images/01.add_dev_realm.png)
Figure 1: add dev realm
## Step 3: Create a Client (Micro-Service)
Client ID       : employee-service
Client Protocol : openid-connect

![](static/images/02.add_client.png)
Figure 2: Add client

## Step 4: Configure Client
If Keycloak runs on Port 8080, make sure your microservice runs on another port. In the example, micro-service is configured to run on 8085.

Access Type         : confidential
Valid Redirect URIs : http://localhost:8085
Service Accounts Enabled : On
Authorization Enabled : On
**Note**: Access Type confidential supports getting access token using client credentials grant as well as authorization code grant. If a micro-service need to call another micro-service, caller will be ‘confidential’ and callee will be ‘bearer-only’.

![](static/images/03.configure_client.png)
Figure 2: Configure client

## Step 5: Create Client Role
Create a role under the client. In this case, role USER is created under employee-service.

![](static/images/04.create_role.png)
Figure 3: Create role

## Step 6: Create a Mapper (To get user_name in access token)
Keycloak access token is a JWT. It is a JSON and each field in that JSON is called a claim. By default, logged in username is returned in a claim named “preferred_username” in access token. Spring Security OAuth2 Resource Server expects username in a claim named “user_name”. Hence, we had to create below mapper to map logged in username to a new claim named user_name.

![](static/images/05.create_mapper.png)
Figure 4: Create Mapper

## Step 7: Create User

![](static/images/06.create_user.png)
Figure 5: Create User

## Step 8: Map Client Role To User
In order to provide access to client (micro-service), respective role needs to be assigned/mapped to user.

![](static/images/07.assign_role_to_user.png)
Figure 6: Assign role to user

## Step 9: Get Configuration From OpenID Configuration Endpoint
Because Keycloak is OpenID Connect and OAuth2 complaint, below is OpenID Connection configuration URL to get details about all security endpoints,

GET http://localhost:8080/auth/realms/dev/.well-known/openid-configuration

Important URLS to be copied from response:

issuer : http://localhost:8080/auth/realms/dev

authorization_endpoint: ${issuer}/protocol/openid-connect/auth

token_endpoint: ${issuer}/protocol/openid-connect/token
 
token_introspection_endpoint: ${issuer}/protocol/openid-connect/token/introspect

userinfo_endpoint: ${issuer}/protocol/openid-connect/userinfo

Response also contains grant types and scopes supported

grant_types_supported: ["client_credentials", …]

scopes_supported: ["openid", …]

## To Get Access Token Using Postman (For Testing)
Select Authorization Type as OAuth 2.0, click on ‘Get New Access Token’ and enter following details.

![](static/images/08.get_access_token_from_keycloak.png)


Postman tool screenshot: To get access token from keycloak for a client

Make sure you select client authentication as “Send client credentials in body” while requesting token.
Callback URL is redirect URL configured in Keycloak.
Client secret may be different for you, copy the one from client configuration in keycloak.
You may also use https://jwt.io to inspect the contents of token received.