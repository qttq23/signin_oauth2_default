
# oauth2:
read the 1st pattern: Authorization Flow (https://dzone.com/articles/oauth-20-in-a-nutshell)  
also you can view the `desktop c++` section.

# identity platform:
- is a hub for authentication. it signs you in, give you IdToken and RefreshToken. allows you to access Google Cloud Storage, Firebase Database,...
- it provides Google/Github/Facebook/Password Signin.

- internally, it interacts with oauth2 services (google, facebook,..) and then sign you in.
- it acts as Oauth Client (web server) in the 1st pattern (link above).

(  
quickstart:  
https://cloud.google.com/identity-platform/docs/sign-in-user-email

Rest Api of Identity Platform:  
https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/lookup

Firebase Auth client libraries: (firebase auth is subset of Identity platform. firebase only supports web and mobile, not c++/desktop)  
https://firebase.google.com/docs/reference/js/auth.auth

)

# google sigin:
## web & mobile
Identity platform has client libraries (Firebase Authentication)   
for web (javascript), mobile (android, ios, flutter).  
admin libraries for nodejs server.  
(https://firebase.google.com/docs/reference/js/auth.auth)

It doesnt have client or server(admin) libraries for desktop c++ app and server c++.  
c++ client/server has to use the RestApi of Identity Platform.  

## desktop c++
use Rest Api of Firebase Authentication or Identity Platform (both are same).  
(https://firebase.google.com/docs/reference/rest/auth#section-get-account-info  
or: https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/lookup)

the flow is as followings:  
- start local server to receive response (authorization code) from Google Oauth2 server.
- prepare the request to Google Oauth2 server. (needs some keys which can be get from GGcloud console)
- open browser and go to that prepared request.
- user selects an account.
- Google Oauth2 responds an Authorization Code to RedirectUri (the local server).
- local server receives Authorization Code and use that code to exchange for Access Token or Id Token.
(by requesting to Google Oauth2 server).

-> that's all for Oauth2 flow.  
-> further, you want to signin to Identity Platform. because it manages your authentication better with IdToken,RefreshToken and custom claims.   
Also, if you signin to Identity Platform, you can access other services such as Google Cloud Storage, Firebase Database,..   

- after local server successfully exchanged for AccessToken/IdToken, it put those tokens to the request to Identity Platform to sign in.  
- Identity Platform will responds a pair of IdToken and RefreshToken.  
(idtoken and refreshtoken here is different from Google Oauth2 Idtoken/refreshtoken)
- local server receives those tokens and store it in local file.

-> That's all about the Sigin to Identity Platform.  
-> c++ desktop client are now able to access Google cloud storage.  
-> but you may want to control which users (such as normal users and admin) are allowed to read/write to storage.  
-> the strategy is to put some Custom Claims to the IdToken of Identity Platform.   
-> later if your c++ client makes request to Google Cloud storage, GGcloud Storage will investigate the CustomClaims in IdToken and behaves properly.  

- After local server has the IdToken of Identity Platform, it sends that IdToken to your ApiServer.
- ApiServer (eg: nodejs server with Firebase Admin Sdk) can verify your identity and set up resources (database) for you and also set some CustomClaims.
- After apiserver set CustomClaims, the client needs to refresh IdToken to get the IdToken that holds the new custom Claims.

## server c++
NO admin libraries.  
only Rest Api for c++ server.  
(https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/lookup)

For simplier, you can use Nodejs server instead.   
then if needed, use c++ app/tool to cooperate with Nodejs server to do CPU intensive tasks.  
using nodejs can give you the ability to deploy your server on linux-supported cloud environemnt (most cloud services support Linux).  
if you write c++ server on Windows, you may want to deploy in Azure Container Apps (https://azure.microsoft.com/en-us/services/container-apps/)

