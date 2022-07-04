
# oauth2:
read the 1st pattern: Authorization Flow (https://dzone.com/articles/oauth-20-in-a-nutshell)  
also you can view the `desktop c++` section below.  
(
step-by-step guide:
https://developers.google.com/identity/protocols/oauth2/native-app
)

# identity platform:
- is a hub for authentication. 
- You manually get oauth2 tokens from oauth2 providers then give it to Identity Platform. It signs you in, returns you a IdToken and a RefreshToken.
- It has various types of account management methods such as resetPassword, sendVerifyCode, ...
- An interesting feature is that you can store additional information in the idToken then retrieve it in later for authorize purpose (such as classify normal users and admins). 

- identity platform doesnot have client/admin libary, it only has the Rest API. Client side needs the `API key` and `idToken` to get user info. Server side needs `service account` to set some important user info (such as custom claims). 
- You can use the Firebase Authentication's client library. view more below.

(
quickstart:  
https://cloud.google.com/identity-platform/docs/sign-in-user-email

Rest Api of Identity Platform:  
https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/lookup

Rest Api for Identity platform for refreshing token:
https://cloud.google.com/identity-platform/docs/use-rest-api#section-refresh-token
)

# Firebase Authentication:
- is a subset of Identity Platform.
- it provides Google/Github/Facebook/Password Signin in the sense that it manages the oauth2 flows for you. you don't need to do google oauth2 flow like in Identity platform.
- Once you signed in to Firebase Authentication, it allows you to access Firebase Storage, Firebase Database,... It's more powerful if you store authorize information in custom claims and then check those claims in GG cloud storage.

- internally, it interacts with oauth2 services (google, facebook,..) and then sign you in.
- it acts as Oauth Client (web server) in the 1st pattern (link above).

- it has client libraries for web and mobile. it has admin libraries for cross-platform servers such as nodejs, java, go.
- it doesnot have client/admin library for c++ desktop/server app.

(  
Firebase Auth client libraries: (firebase auth is subset of Identity platform. firebase only supports web and mobile, not c++/desktop)  
https://firebase.google.com/docs/reference/js/auth.auth

Firebase Auth admin libraries:
https://firebase.google.com/docs/reference/admin/node/firebase-admin.auth.baseauth.md#baseauth_class

Firebase Auth Rest API (same as Identity Platform rest api):
https://firebase.google.com/docs/reference/rest/auth
)

-> recommended: use Firebase Authentication client libraries for mobile and web app to simplify the signin process. use Firebase Authentication admin library for nodejs server to verify users and add custom claims.  
-> recommended: only use Identiy Platform rest API for c++ client desktop app. donot use in c++ server app. 
(because to use in c++ server, you have to maunally exchange the service account to get oauth2 access token which is very complex process and you will not want to mess with it. if needed, use c++ app as a cooperative tool that supports Nodejs server with CPU intensive tasks.) 

-> note about firebase storage: Firebase Authentication offers a convenient way to control user access to Firebase storage without additional server. Once you singed in into Firebase Authentication (or Identity Platform), you get an IdToken to access Firebase storage. Firebase storage will view your idtoken to know who you are and what privilages you have (such as admin or normal) through your idtoken's custom claims. But this is not recommended. because you have to use Firebase products not google cloud products. for example, you have to create new separate Firebase Storage bucket regardless of wheater you already have buckets in GGcloud storage or not. Please view Google `Cloud Storage article` for more generic solution for storaging files.   

# google sigin:
## web & mobile
Firebase Authentication has client libraries for web (javascript), mobile (android, ios, flutter).  
(https://firebase.google.com/docs/reference/js/auth.auth)

## server
Firebase Authentication has Nodejs admin library.  
(https://firebase.google.com/docs/reference/admin/node/firebase-admin.auth.baseauth)

## desktop c++
use Rest Api of Firebase Authentication or Identity Platform (both are same). you will need `API key`.  
(https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/lookup
or: https://firebase.google.com/docs/reference/rest/auth#section-get-account-info )

the flow is as followings:  
- start local server to receive response (authorization code) from Google Oauth2 server.
- prepare the request to Google Oauth2 server. (needs some keys which can be get from GGcloud console)
- open browser and go to that prepared request.
- user selects an account.
- Google Oauth2 responds an Authorization Code to RedirectUri (the local server).
- local server receives Authorization Code and use that code to exchange for Access Token or Id Token.
(by requesting to Google Oauth2 server).  
(reference: https://developers.google.com/identity/protocols/oauth2/native-app)  

-> that's all for Oauth2 flow.  
-> further, you want to signin to Identity Platform. because it manages your authentication better with IdToken,RefreshToken and custom claims.   

- after local server successfully exchanged for AccessToken/IdToken, it put those tokens to the request to Identity Platform to sign in.  
- Identity Platform will responds a pair of IdToken and RefreshToken.  
(idtoken and refreshtoken here is different from Google Oauth2 Idtoken/refreshtoken)
- local server receives those tokens and store it in local file.  
(reference: https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/signInWithIdp)

-> That's all about the Sigin to Identity Platform.  
-> the idToken is your identity, keep it secret. api server can extract info from idToken to know who you are.

-> In some scenarios, apiserver want to store some additional info to your idtoken for convenient purposes such as quickly get your's role (admin or user) to perform some certain actions. apiserver can store addtional info to Custom claims.
(the usage of custom claims to identify you in Firebase Storage is not recommended. see the article `Storage` for more details.)

- After local server has the IdToken of Identity Platform, it sends that IdToken to your ApiServer.
- ApiServer (eg: nodejs server with Firebase Authentication Admin Sdk) can verify your identity and set up resources (database) for you and also set some CustomClaims.
- After apiserver set CustomClaims, the client needs to refresh IdToken to get the IdToken that holds the new custom Claims.  

(reference:
verify id token:
https://firebase.google.com/docs/reference/admin/node/firebase-admin.auth.baseauth#baseauthverifyidtoken

add custom claims:
https://firebase.google.com/docs/reference/admin/node/firebase-admin.auth.baseauth.md#baseauthsetcustomuserclaims

refresh token:
https://firebase.google.com/docs/auth/admin/custom-claims#propagate_custom_claims_to_the_client
or:
https://cloud.google.com/identity-platform/docs/use-rest-api#section-refresh-token
)

## server c++
Firebase Authentication has NO admin libraries for c++ desktop.  
there is only Identiy Platform's Rest Api for c++ server. But it requires you to manually exchange `service account` for oauth2 access token. so, not recommneded. 
(https://cloud.google.com/identity-platform/docs/reference/rest/v1/accounts/lookup)

For simplier, you can use Nodejs server instead.   
then if needed, use c++ app/tool to cooperate with Nodejs server to do CPU intensive tasks.  
using nodejs can give you the ability to deploy your server on linux-supported cloud environemnt (most cloud services support Linux).  
if you write c++ server on Windows, you may want to deploy in Azure Container Apps (https://azure.microsoft.com/en-us/services/container-apps/)


# Overview about Google Cloud account, key, permission, credentials
(https://cloud.google.com/docs/authentication)

## API key
used for public anonymous access to google cloud apis.
You should restrict it in production. you can restrict which domain, ip can use and which api services can use.  
(https://cloud.google.com/docs/authentication/api-keys)

## OAuth 2.0 Client IDs
is an ID to identify your app (web app or desktop or mobile app).
used in conjunction with Oauth2 flow (such as Google oauth2 flow) to get user consent.

for example, Google will show your app info (through your oauth2 client ID) to end users and ask if them accept giving your app permissions (such as reading profile, email or more powerful permissions).
The same for Github/Facebook, you have to create a oauth2 client id in Github/Facebook console and then use that id to ask if end-users of facebook/github agree to give you permissions.

Usually, the common use case it to login user with google, facebook, github, ... You only ask users for permissions to access their email, profile. After getting the oauth2 token which has enough scope to get email/profile, you send that token to Identity Platform to signup/signin your users. (or you can query user info using those tokens with your own authentication system. Not recommended)

(https://cloud.google.com/docs/authentication/end-user  
https://developers.google.com/identity/protocols/oauth2/native-app)

## service account
contains an ID and a secret key. (internally, it's used to exchange the oauth2 access token to access powerful Admin API)
this is the most powerful credentials. should only used in trusted environments (such as server).
you can also extend or restrict the scope of service account. 

(using client library to handle service account: (nodejs server is recommended)
https://cloud.google.com/docs/authentication/production

manually get oauth2 tokens from service account: (not recommended)  
https://developers.google.com/identity/protocols/oauth2/service-account
)
