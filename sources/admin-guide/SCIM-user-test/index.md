**Table of Contents** 

- [Adding New User](#adding-new-user)
- [SCIM UMA Authentication](#scim-uma-authentication )
	- [Base configuration: Create oxAuth Clients, Policies](#base-configuration-create-oxauth-clients-policies)
	- [oxTrust configuration (Resource Server)](#oxtrust-configuration-resource-server)
	- [SCIM Client (Requesting Party) sample code](#scim-client-requesting-party-sample-code)


# Adding New User 

To keep the Gluu Server up-to-date with the latest user claims, your organization can either "push" or "pull" identity data. In the "pull" mode, otherwise known as LDAP Syncronization or Cache Refresh, the Gluu Server can use an existing LDAP identity source like Microsoft Active Directory as the authoritative source of identity information. If you "push" identities to the Gluu Server, you can use the JSON/REST SCIM API. 


At the moment, SCIM endpoints allow two types of Authentication modes:

1. SCIM UMA Authentication
2. SCIM oxAuth Authentication

To use any of the given authentication mode, we need to create user instance with specified authentication mode. We'll discuss each of the methods here:

# SCIM UMA User Authentication

This is step by step guide to configure UMA for oxTrust and SCIM client. 

## Base Configuration: Create oxAuth Clients, Policies

1. Register oxAuth client with scope “uma_protection”. Property “oxAuthTokenEndpointAuthMethod” of this client should has value “client_secret_basic”. It's possible to do that using few methods: [Client Registration](http://ox.gluu.org/doku.php?id=oxauth:clientregistration), using [oxTrust](http://ox.gluu.org/doku.php?id=oxtrust:home) GUI, manually add entry to LDAP. oxTrust will use this oxAuth client to obtain PAT. Sample result entry:

        dn: inum=@!1111!0008!F781.80AF,ou=clients,o=@!1111,o=gluu
        objectClass: oxAuthClient
        objectClass: top
        displayName: Resource Server Client
        inum: @!1111!0008!F781.80AF
        oxAuthAppType: web
        oxAuthClientSecret: eUXIbkBHgIM=
        oxAuthIdTokenSignedResponseAlg: HS256
        oxAuthScope: inum=@!1111!0009!6D96,ou=scopes,o=@!1111,o=gluu
        oxAuthTokenEndpointAuthMethod: client_secret_basic

2. Register oxAuth client with scope “uma_authorization”. Property “oxAuthTokenEndpointAuthMethod” of this client should has value “client_secret_basic”. It's possible to do that using few methods: [Client Registration](http://ox.gluu.org/doku.php?id=oxauth:clientregistration), using [oxTrust](http://ox.gluu.org/doku.php?id=oxtrust:home) GUI, manually add entry to LDAP. SCIM Client will use this oxAuth client to obtain AAT. Sample result entry:

        dn: inum=@!1111!0008!FDC0.0FF5,ou=clients,o=@!1111,o=gluu
        objectClass: oxAuthClient
        objectClass: top
        displayName: Requesting Party Client
        inum: @!1111!0008!FDC0.0FF5
        oxAuthAppType: web
        oxAuthClientSecret: eUXIbkBHgIM=
        oxAuthIdTokenSignedResponseAlg: HS256
        oxAuthScope: inum=@!1111!0009!6D97,ou=scopes,o=@!1111,o=gluu
        oxAuthTokenEndpointAuthMethod: client_secret_basic

3. Create UMA policy. These are list of steps which allows to add new policy: 

 	1. Log with administrative privileges into oxTrust.
 	2. Open menu “Configuration→Manage Custom Scripts”.
 	4. Select “UMA Authorization Policies” tab and click “Add custom script configuration”.
 	5. Select language “Python”.
 	6. Paste this base policy script:


            from org.xdi.model.custom.script.type.uma import AuthorizationPolicyType
            from org.xdi.util import StringHelper, ArrayHelper
            from java.util import Arrays, ArrayList
            from org.xdi.oxauth.service.uma.authorization import AuthorizationContext

            import java

            class AuthorizationPolicy(AuthorizationPolicyType):
                def __init__(self, currentTimeMillis):
                    self.currentTimeMillis = currentTimeMillis

            def init(self, configurationAttributes):
                print "UMA authorization policy. Initialization"
                print "UMA authorization policy. Initialized successfully"

                return True   

            def destroy(self, configurationAttributes):
                print "UMA authorization policy. Destroy"
                print "UMA authorization policy. Destroyed successfully"
                return True   

            def getApiVersion(self):
                return 1

            # Authorizae access to resource
            #   authorizationContext is org.xdi.oxauth.service.uma.authorization.AuthorizationContext
            #   configurationAttributes is java.util.Map<String, SimpleCustomProperty>
            def authorize(self, authorizationContext, configurationAttributes):
                print "UMA Authorization policy. Attempting to authorize client"
                client_id = authorizationContext.getGrant().getClientId()
                user_id = authorizationContext.getGrant().getUserId()

                print "UMA Authorization policy. Client: ", client_id
                print "UMA Authorization policy. User: ", user_id
                if (StringHelper.equalsIgnoreCase("@!1111!0008!FDC0.0FF5", client_id)):
                    print "UMA Authorization policy. Authorizing client"
                    return True
                else:
                    print "UMA Authorization policy. Client isn't authorized"
                    return False

                print "UMA Authorization policy. Authorizing client"
                return True
 - Replace in script above client inum "@!1111!0008!FDC0.0FF5" with client inum which were added in step 2.
 - Click "Enabled" check box.
 - Click "Update" button.


4. Add UMA scope. These are list of steps which allows to add new scope.

	 - Log with administrative privileges into oxTrust.
	 - Open menu “OAuth2→UMA”.
	 - Select “Scopes” tab and click “Add Scope Description”.
	 - Select “Internal” type.
	 - Fill the form.
	 - Select policy which we aded in previous step.
	 - Click “Add” button. Sample result entry:

            dn: inum=@!1111!D386.9FB1,ou=scopes,ou=uma,o=@!1111,o=gluu
            objectClass: oxAuthUmaScopeDescription
            objectClass: top
            displayName: Access SCIM
            inum: @!1111!D386.9FB1
            owner: inum=@!1111!0000!D9D9,ou=people,o=@!1111,o=gluu
            oxPolicyScriptDn: inum=@!1111!CA0D.1918!2DAF.F995,ou=scripts,o=@!1111,o=gluu
            oxId: access_scim
            oxRevision: 1
            oxType: internal

5. Register UMA resource set. It's possible to do that via Rest API or via oxTrust GUI. Sample code: [https://github.com/GluuFederation/oxAuth/blob/master/Client/src/test/java/org/xdi/oxauth/ws/rs/uma/RegisterResourceSetFlowHttpTest.java) These are list of steps which allows to add new resource set:

	 - Log with administrative privileges into oxTrust.
	 - Open menu “OAuth2→UMA”.
	 - Select “Resources” tab and click “Add Resource Set”.
	 - Fill the form.
	 - Add UMA Scope which we created in previous steps.
	 - Add Client which we created in second step.
	 - Click “Add” button. Sample result entry:

                dn: inum=@!1111!C264.D316,ou=resource_sets,ou=uma,o=@!1111,o=gluu
                objectClass: oxAuthUmaResourceSet
                objectClass: top
                displayName: SCIM Resource Set
                inum: @!1111!C264.D316
                owner: inum=@!1111!0000!D9D9,ou=people,o=@!1111,o=gluu
                oxAuthUmaScope: inum=@!1111!D386.9FB1,ou=scopes,ou=uma,o=@!1111,o=gluu
                oxFaviconImage: http://example.org/scim_resource_set.jpg
                oxId: 1403179695657
                oxRevision: 1

## oxTrust configuration (Resource Server)

Add next oxTrust UMA related configuration properties to oxTrust.properties:

    # UMA SCIM protection
    uma.issuer=https://centos65.gluu.info
    uma.client_id=@!1111!0008!F781.80AF
    uma.client_password=<encrypted_password>
    uma.resource_id=1403179695657
    uma.scope=https://centos65.gluu.info/oxauth/seam/resource/restv1/uma/scopes/access_scim

Values of these properties correspond to entries from first section.

Note: In order to recreate oxTrust configuration in LDAP you should remove oxTrust configuration entry from LDAP and restart tomcat.
Example DN of oxTrust configuration entry: ou=oxtrust,ou=configuration,inum=@!1111!0002!4907,ou=appliances,o=gluu

## SCIM Client (Requesting Party) sample code

This is sample SCIM Client code which request user information from server.

    package gluu.scim.client.dev.local;
    
    import gluu.scim.client.auth.UmaScimClientImpl;
    import gluu.scim.client.ScimResponse;

    import javax.ws.rs.core.MediaType;
    
    public class TestUMAScimClient {
	    public static void main(String[] args) {
     
		// public UmaScimClientImpl(String domain, String umaMetaDataUrl, String umaAatClientId, String umaAatClientSecret) 
		
		    final UmaScimClientImpl scimClient = new UmaScimClientImpl ("https://centos65.gluu.info/identity/seam/resource/restv1", "https://centos65.gluu.info/.well-known/uma-configuration",
				    "@!1111!0008!FDC0.0FF5", "secret");

		    try {
			// public ScimResponse retrievePerson(String uid, String mediaType) throws IOException 

			    ScimResponse response1 = scimClient.retrievePerson("@!1111!0008!FDC0.0FF5", MediaType.APPLICATION_JSON);
			    System.out.println(response1.getResponseBodyString());
			

		    } catch (Exception ex) {
			    ex.printStackTrace();
		    }
	    }
    
    }

Values from these example correspond to entries from first section. As you can see in the given code, adding a new user using SCIM needs some specific steps to be followed, which involves user information (domain, Client_id, Client_secret), and then it should be followed by authorization of client against UMA. During the process of UMA authentication, we can call UmaScimClientImpl method *init()*, this method will call *initUmaAuthentication()*, which gets Request Permission Ticket and passes this ticket to SCIM endpoint where it would be authenticated. (To use *init* method, you need to Extend your class from UmaScimClientImpl, as init() is a protected method.)





