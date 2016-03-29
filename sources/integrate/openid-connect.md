# Table of Contents 
[TOC]

# Overview
[OpenID Connect](http://openid.net/connect) ("Connect") is a standard
profile of OAuth2 which defines a protocol to enable a website or mobile
application to send a person to a domain for authentication and required
attributes (e.g. email address, first name, last name, etc.). OpenID Connect
also provides some of the plumbing around authentication to automate how
this happens. If a person is visiting a website for the first time, the
process that OpenID Connect defines is 100% bootstrapable by the
website. This is really critical for Internet scalability. To visit
someone's website, or to send someone an email, you do not need to get
the system administrators involved. Connect provides the same type of
scalable infrastructure for authentication and authorization, and promises to define a base level domain
identification.

## Jargon (taxonomy)

If you are familiar with SAML, there are many parallels in OpenID
Connect, but the jargon (or "taxonomy") is different. For example,
instead of attributes, we have "user claims". Instead of Service
Provider (SP), we have "client". Instead of Identity Provider (IdP), it
is an OpenID Provider (OP).

## Discovery 

The first thing you want to know about any OAuth2 API is where are the
endpoints (i.e. what are the uris where you call the APIs). OpenID
Connect provides a very simple mechanism to accomplish this: make a GET
request to `https://<domain>/.well-known/openid-configuration`.

[OpenID Connect
Discovery](http://openid.net/specs/openid-connect-discovery-1_0.html) is
based on a previous standard called
[WebFinger](http://en.wikipedia.org/wiki/WebFinger).

If you want to try a sample discovery request, you can make a GET
request to [Gluu's OpenID Connect Discovery
Page](https://idp.gluu.org/.well-known/openid-configuration).

## Scopes

In SAML, the IdP releases attributes to the SP. OpenID Connect provides
similar functionality, with more flexibility in case the person needs to
self-approve the release of information from the IdP to the website (or
mobile application). In OAuth2, scopes can be used for various purposes.
OpenID Connect uses OAuth2 scopes to "group" attributes. For example, we
could have a scope called "address" that includes the street, city,
state, and country user claims. By default the Gluu Server defines six
scopes.

![Scopes Screenshot](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_scope.png)

The Gluu Server administrator can easily add more scopes in the GUI.
Click *Add Scope* and you will be presented with the following screen:

![Add Scopes](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/openid_connect/oxtrust_scope_screenshot.png)

You will have the ability to provide a Display Name, Description,
whether or not the scope is provided by default, and the claims that are
included in the scope.

Default Scope needs some further explanation. When a client uses dynamic
client registration, the OpenID Connect specification says that the
`openid` scope should always be released, which contains an identifier
for that person, normally the username. If you want to release another
scope automatically, set the Default Scope to `true` for that scope. You
can always explicitly release a scope to a certain client later on, but
this will require some manual intervention by the domain administrator.

To add more claims, simply click "Add Claim" and you will be presented
with the following screen:

![Add Claims](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_scopeadd.png)

## Client Registration

A client in OAuth2 could be either a website or mobile application.
OpenID Connect has an API for [Dynamic Client
Registration](http://openid.net/specs/openid-connect-registration-1_0.html)
which efficiently pushes the task to the application developer. If you
do not want to write an application to register your client, there are a
few web pages around that can do the job for you. Gluu publishes the
[oxAuth-RP](https://seed.gluu.org/oxauth-rp) and there is also another in [PHP
RP](http://www.gluu.co/php-sample-rp).

If you cannot get the developer to help themselves, or if your domain
doesn't want to allow dynamic client registration, you can use the
oxTrust admin GUI to manually add trusted clients.

Available **Clients** can be seen by hitting the **Search** button
leaving the search box empty.

![Client List](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_clientlist.png)

A new client can be added by clicking the **Add Client** link.

![Add Client](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_addclient.png)

Clicking on the _Add Client_ link allows the Gluu Server administrator
to add a new client. The search box can be used to look up previously
added clients as well. The screenshot below shows the interface to add a
new client.

![Add new client](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_newclient.png)

* _Client Name:_ This contains the recognizable and unique display name
  of the client.

* _Client Secret:_ This is the Data Encryption Standard scheme used by
  Confidential Clients to authenticate the token endpoints. The value for
  the secret can be inserted manually, but it is highly recommended to use
  the Dynamic Client Registration Endpoint. The Gluu oxAuth provides a
  random, generated Client Secret in the Dynamic Client Registration
  procedure.

* _Application Type:_ There are two types of applications, Web and
  Native. The different configuration for the different application types
  are given below.

	* _Web:_ The Dynamic Client Registration is the default for web. In
    this type the redirect_uri for implicit grant type must be a real
    hostname with HTTPS. This type is not approved any localhost or HTTP.
    The web application uses the authorization code flow for clients which
    can maintain a client secret between the uris and the authorization
    server.

	* _Native:_ Custom uri for Native type application have to follow HTTP
    with localhost. This is suitable for a mobile app which cannot maintain
    the client secret between itself and the authorization server.

* _Pre Authorization:_ The Gluu Server disables this option by default,
but it is possible to allow the users to access any URL according to the
Organization Policy by the Gluu Server administrator.

* _Logo URI:_ The logo for the client application

* _Client URI:_ The home page for the client

* _Policy URI:_ The URI for the client policy 

* _Terms of Service URI:_ The conditions for the client

* _JWKS URI:_ The URL for the Client's JSON Web Key Set

* _Sector Identifier URI:_ The Sector Identifier URL

* _JWS alg Algorithm for signing the ID Token:_ The algorithm to sign the ID token. See available algorithms

* _JWE alg Algorithm for encrypting the ID Token:_ See [Algorithms section](#algorithm) for options

* _JWE enc Algorithm for encrypting the ID Token:_ See [Algorithms section](#algorithm) for options

* _JWS alg Algorithm for signing the UserInfo Responses:_ See [Algorithms section](#algorithm) for options

* _JWS alg Algorithm for encrypting the UserInfo Responses:_ See [Algorithms section](#algorithm) for options

* _JWE enc Algorithm for encrypting the UserInfo Responses:_ See [Algorithms section](#algorithm) for options

* _JWS alg Algorithm for signing Request Objects:_ See [Algorithms section](#algorithm) for options

* _JWE alg Algorithm for encrypting Request Objects:_ See [Algorithms section](#algorithm) for options

* _JWE enc Algorithm for encrypting Request Objects:_ See [Algorithms section](#algorithm) for options

* _Authntication method for the Token Endpoint:_ The authentication method can be `client_secret_basic, client_secret_post, client_secret_jwt, private_key_jwt, none`

* _JWS alg Algorithm for Authentication method to Token Endpoint:_ See [Algorithms section](#algorithm) for options

* _Default Maximum Authentication Age:_ Enter validity of authntication in seconds

* _Require Auth Time:_ The requirement for authentication time

* _Persist Client Authorizations*:_ The options are `true, false`

* _Initiate Login URI:_ The https scheme that can be used by third party to initiate login by RP

* _Logout URI:_ The logout URL

* _Logout Session Required*:_ The options are `true, false`

**Buttons at the bottom**

* _Add Login URI:_ This option can be used to add the login URL.
![Add Login URI](![Add Scopes](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth_loginuri.png)

* _Add Scopes:_ This option can be used to add the required scopes in the Gluu Server.
![Add Scopes](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_addscope.png)

  The available scopes can be listed by hitting the *Search* button, and
  keeping the search phrase blank. Furthermore, from this the Gluu Server
  administrator can select the required scopes.

* _Add Response Type:_ There are three types of responses in the Gluu
  Server and they are Code, Token and ID Token. The Gluu Server
  Administrator can select all of them for testing purposes.
![Response Type](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_response.png)

* _Add Grant Type:_ There are 3 grant type available in this option `authorization_code, implicit, refresh_token`
![Grant Type](https://raw.githubusercontent.com/GluuFederation/docs/master/sources/img/oxTrust/admin_oauth2_grants.png)

* _Add Contact:_ Use this option to add the email address for the Client contact

* _Add Default ACR value:_ Use this option to define the default ACR Value

* _Add Request URI:_ Use this option to add the Request URI

### Algorithm
oxAuth supports various types of signature and encryption
algorithms for authorizing request parameter passing, ID token signature
and encryption, signing return responses, Encrypt User Info Endpoints
etc.

**Note:** It is a good practice to implement ID Token Signatures with the RSA
SHA-256 algorithm (algorithm value RS256). Additionally, oxAuth also
supports other algorithms that are listed below.

_Available Signature Algorithms:_ HS256, HS384, HS512, RS256, RS384, RS512, ES256, ES384, ES512.

_Encryption, Key Encryption Algorithms:_ RSA1_5, RSA-OAEP, A128KW, A256KW.

_Block Encryption Algorithms:_ A128CBC+HS256, A256CBC+HS612, A128GCM, A256GCM,

### Custom Client Registration

Using interception scripts you can customize client registration
behavior. For example, by default oxAuth allows new clients to access to
default scopes only. With a custom client registration interception
script it is possible to allow access to more scopes. For instance, we
can use `redirect_uri` to determine if we need to allow access to
additional scopes or not.

To access the interface for custom scripts in oxTrust, navigate to
Configuration --> Custom Scripts --> Custom Client Registration.

Take a look at our [example client registration
script](../customize/sample-client-registration-script.py)
for further reference.

### Search clients
![](http://www.gluu.org/docs/img/openid_connect/oxtrust_search_clients.png "Screenshot of oxTrust browse / search clients")

### View client

![](http://www.gluu.org/docs/img/openid_connect/oxtrust_view_client.png "Screenshot of oxTrust view client")

## Session management

Logout is a catch-22. There is no perfect answer to logout that
satisfies all the requirements of all the domains on the Internet. For
example, large OpenID Providers, like Google, need a totally stateless
implementation--Google cannot track sessions on the server side for
every browser on the Internet. But in smaller domains, server side
logout functionality can be a convenient solution to cleaning up
resources.

The OpenID Connect [Session
Management](http://openid.net/specs/openid-connect-session-1_0.html) is
still marked as draft, and new mechanisms for logout are in the works.
The current specification requires JavaScript to detect that the session
has been ended in the browser. It works... unless the tab with the
JavaScript happens to be closed when the logout event happens on another
tab. Also, inserting JavaScript into every page is not feasible for some
applications. A new proposal is under discussion where the OpenID
Connect logout API would return `IMG` HTML tags to the browser with the
logout callbacks of the clients. This way, the browser could call the
logout uris (not the server).

The Gluu Server is very flexible, and supports both server side session
management, and stateless session management. For server side business
logout, the domain admin can use Custom Logout scripts. This can be
useful to clean up sessions in a legacy SSO system (i.e. SiteMinder), or
perhaps in a portal.

The key for logout is to understand the limitations of logout, and to
test the use cases that are important to you, so you will not be
surprised by the behavior when you put your application into production.

## Testing with oxAuth RP

  - Go to https://seed.gluu.org/oxauth-rp
  - Or deploy `oxAuth-rp.war`

### OpenID Connect Discovery

  - Enter an identifier, for example: https://seed.gluu.org or acct:mike@seed.gluu.org
  - Click submit.

![](http://www.gluu.org/docs/img/oxAuth-RP/openidconnectdiscovery.png "Screenshot of oxAuth-RP OpenID Connect Discovery")

### Dynamic Client Registration

![](http://www.gluu.org/docs/img/oxAuth-RP/dynamicclientregistration.png "Screenshot of oxAuth-RP Dynamic Client Registration")

#### Client Read

![](http://www.gluu.org/docs/img/oxAuth-RP/clientread.png "Screenshot of oxAuth-RP Client Read")


### Authorization Endpoint

#### Request Authorization and receive the Authorization Code and ID Token

  - Go to https://seed.gluu.org/oxauth-rp
  - Enter the Authorization Endpoint (eg: https://seed.gluu.org/oxauth/seam/resource/restv1/oxauth/authorize)
  - Select the Response Types: CODE and ID_TOKEN
  - Enter the Client ID (eg: @!EDFB.879F.2DAE.D95A!0001!0442.B31E!0008!A2DA.C10F)
  - Select the desired scopes: OpenID is mandatory, profile, address,
    email and phone are optional.
  - Enter a Redirect uri, e.g. https://seed.gluu.org/oxauth-rp/home.seam
  - Optionally enter a state value.
  - Click submit.

![](http://www.gluu.org/docs/img/oxAuth-RP/requestauthorizationcodegrant.png "Screenshot of oxAuth-RP Authorization Endpoint")

#### Request Access Token using the Authorization Code

  - Once redirected back to https://seed.gluu.org/oxauth-rp
  - Enter the Token Endpoint (eg: https://seed.gluu.org/oxauth/seam/resource/restv1/oxauth/token)
  - Select the Grant Type: AUTHORIZATION_CODE
  - Enter the Client ID.
  - Enter the Client Secret.
  - Enter the Code received from the previous request
  - Enter the Redirect uri, e.g. https://seed.gluu.org/oxauth-rp/home.seam
  - Enter the scopes: OpenID profile address email phone.
  - Click submit.

![](http://www.gluu.org/docs/img/oxAuth-RP/requestaccesstokenwithauthorizationcode.png "Screenshot of oxAuth-RP Token Endpoint")

#### Request new Access Token using the Refresh Token

  - Go to https://seed.gluu.org/oxauth-rp
  - Enter the Token Endpoint (https://seed.gluu.org/oxauth/seam/resource/restv1/oxauth/token)
  - Select the Grant Type: REFRESH_TOKEN
  - Enter the Client ID.
  - Enter the Client Secret.
  - Enter the Refresh Token received in a previous request.
  - Click submit.

![](http://www.gluu.org/docs/img/oxAuth-RP/refreshtoken.png "Screenshot of oxAuth-RP Refresh Token")


### UserInfo Endpoint

![](http://www.gluu.org/docs/img/oxAuth-RP/userinfoendpoint.png "Screenshot of oxAuth-RP User Info Endpoint")

### OpenID Connect Session Management

#### End Session Endpoint

![](http://www.gluu.org/docs/img/oxAuth-RP/endsession.png "Screenshot of oxAuth-RP End Session Endpoint")

#### Check Session iFrame

![](http://www.gluu.org/docs/img/oxAuth-RP/checksession.png "Screenshot of oxAuth-RP Check Session iFrame")

# oAuth 2 Grants

There are two additional flows that the Gluu Server supports for user
and client authentication, which are not part of the OpenID Connect
specification. The flows are explained in the following page.

* [oAuth 2 Grants](./oauth2grants.md)
