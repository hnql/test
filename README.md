# service-cloud-connector: configuration guide
**What is the service-cloud-connector?**   
A [Salesforce Labs](https://twitter.com/salesforce_labs?lang=en) project that facilitates an "above the API" integration between Salesforce B2C Commerce and Service Clouds.

## Setting Up Service Cloud

### Pre-Installation Configuration

###### The following configuration steps must be performed before deploying it to your Salesforce instance.

#### Create an SCC Integration User Profile
Before deploying the service-cloud-connector, create a custom profile by cloning the System Administrator profile.

1. From Setup, enter Profiles in the quick find box, then select **Profiles**.
1. From the profiles list, select **Clone** from the System Administrator profile.
1. In the clone form, enter the following values:

	| Property | Value |
	|------------ | ---------- |
	| Profile Name | SCC Integration User |
	| Existing Profile| System Administrator |

	> The Profile Name *must* be *SCC Integration User* as the deployment package depends on this profile existing before installation.

1. To save your changes, lick **Save**.
1. Verify that the Edit view for the SCC Integration User is displayed after saving your changes.

	> If not, find the 'SCC Integration' user in the 	profile listing and click **Edit** next to 	the profile to open this view.

1. Click **System Permissions** to view system-level permissions for the profile.
1. Ensure that 'API Enabled' is selected for this profile.

#### Create an SCC Integration User
With the SCC Integration User Profile created, create the SCC Integration User.  The SCC Integration user
is used by Service Cloud to connect to Salesforce B2C Commerce.  This user must have a
User License set to Salesforce, and leverage the newly created SCC Integration User custom profile.

1. From Setup, enter Users in the quick find box, then select **Users**.
1. Click **New User** at the top of the users listing.
1. Create a user using the following properties:

	| Property | Value |
	|------------ | ---------- |
	| First Name | User |
	| Last Name | SCCIntegration |
	| Alias | uscci |
	| Email | system email associated to the account |
	| Username | sccintegrationuser@domainname.com |
	| Nickname | sccintegrationuser |
	| Role | <None Specified> |
	| User License | Salesforce |
	| Profile | SCC Integration user |
	| Active | Checked |

	> Remember to use a valid email address for the 	account's email, as the account requires verification. The email 	and username for this user-account should both be 	personalized to the organization.

1. Verify that the account details are correct.
1. Click **Save** button to create the account.
1. Verify that you have received the new-user email.
1. Complete the new-user registration.
1. Log in as the SCC Integration User.

### Deploying the service-cloud-connector Codebase to Service Cloud
The service-cloud-connector's Service Cloud codebase can be deployed to any Salesforce environment leveraging the [Apache Ant](https://ant.apache.org/) installer, located in the [sfsc](../../settings/sfsc) directory.

> If you don't have Ant installed, visit the [Apache Ant](https://ant.apache.org/) homepage for details on how to do so.  If you are a Mac user, you can use [Homebrew to install Ant](https://stackoverflow.com/questions/3222804/how-can-i-install-apache-ant-on-mac-os-x), to set up Apache Ant.  Additionally, the [Apache Ant FAQ](https://ant.apache.org/faq.html) is a great reference for troubleshooting and configuration details.

In a terminal session, enter the following command to verify that Apache Ant is correctly set up:

```
ant -version
```

Which should return the following result:

```
Apache Ant(TM) version 1.10.7 compiled on September 1 2019
```

With Ant properly installed, create the deployment properties configuration file.

#### Setup the Deployment Properties Configuration File
The service-cloud-connector uses a [build.properties](../../settings/sfsc/salesforce_ant_48.0/build.properties.sample) file to define the configuration settings necessary to enable the deployment of the service-cloud-connector to a Salesforce instance.  Follow these instructions to configure the build.properties file to map to your Salesforce environment.

1. Duplicate the [build.properties.sample](../../settings/sfsc/salesforce_ant_48.0/build.properties.sample) file found in the [salesforce_ant_48.0](../../settings/sfsc/salesforce_ant_48.0) directory.
1. Rename the copy of the build.properties.sample file to build.properties.
1. Open the newly created build.properties file.
1. Uncomment line 11, and replace it with the Administrator Account username for the Salesforce instance where the Service Cloud Connector is deployed.
	
	```
	# Manual Deployment: Configuration Step 1
	# - Uncomment the sf.username and sf.password lines
	# -------------------------------------------
	# Specify the login credentials for the desired Salesforce organization
	# The sf.password field must be comprised of the account-password + the account securityToken
	# sf.username = Insert Salesforce username here
	# sf.password = Insert Salesforce password here
	```

	> This should be deployed leveraging a user account that is different from the SCC Integration User.

1. Uncomment line 12, and replace it with the Administrator Account's password and [Security Token](https://help.salesforce.com/articleView?id=user_security_token.htm&type=5).

	> For guidance on how to reset your Security Token, visit the section of this document titled [Reset Your Security Token](#reset-your-security-token).

1. Uncomment line 27, and replace it with the authentication URL for your Salesforce instance. If you are unsure of what authentication URL to use,  visit [What are Salesforce instances?](https://success.salesforce.com/answers?id=90630000000gugYAAQ) and [How to identify the type of Salesforce Org](https://salesforce.stackexchange.com/questions/50/can-we-determine-if-the-salesforce-instance-is-production-org-or-a-sandbox-org).  These pages explain how to determine your instance-type and the authentication URL to use.

	```
	# Manual Deployment: Configuration Step 2
	# - Uncomment the sf.serverurl line
	# -------------------------------------------
	# Use 'https://login.salesforce.com' for production or developer edition (the default if not specified).
	# Use 'https://test.salesforce.com' for all sandbox environments.
	# sf.serverurl = https://test.salesforce.com
	```

1. Save the changes that were made to the build.properties file.

Before the execution of the deployment script, validate that the three edits included in this section are accurate and contain the correct Administrative Account username and password, Security Token, and authentication URL.

#### Execute the Deployment Script
The deployment is performed from a command-line or terminal session, depending on the platform of your workstation.

> When executed, the script deploys the full-contents of the package included in this repository to the configured Salesforce instance.  It overwrites any classes, objects, and configurations that exist for the items in the package. Remember that if you redeploy to an environment, you are forced to reconfigure that environment.

1. Open a command prompt or terminal session and navigate to the [salesforce_ant_48.0](../../settings/sfsc/salesforce_ant_48.0) directory.
2. From the command prompt, enter the following command:

	```
	ant deployCodeNoTestLevelSpecified
	```

The deployment script typically takes a handful of minutes to execute -- and providea a message stating the deployment was successful if no errors are encountered during the deployment. Once the success message has been generated, move forward with the execution of the post installation script.

### Execute the Post Installation Script
The first post-installation activity that must be completed is the manual execution of the post-installation script.  The post-install script performs
several configuration activities that must be completed before configuring Service Cloud. This includes:

1. Create Setting leveraged by the service-cloud-connector to manage its operation.
1. Seed the Custom Field Mappings used to map Salesforce B2C Commerce objects to Service Cloud objects.
1. Create the Default PersonAccount leveraged by the service-cloud-connector as a default customer mapping.

These instructions explain how to execute this script from the [Salesforce Developer Console](https://help.salesforce.com/articleView?id=code_dev_console.htm&type=5). Visit the [Trailhead Module Developer Console Basics](https://trailhead.salesforce.com/en/content/learn/modules/developer_console) for a tutorial of the various capabilities present within the [Developer Console](https://help.salesforce.com/articleView?id=code_dev_console.htm&type=5).

1. Log out as the SCC Integration user.
1. Click the Setup gear icon, and select **Developer Console**.
1. From the Developer Console, open the debug menu and select **Open Execute Anonymous Window**.
1. Paste the following snippet of code in the modal-dialog labeled 'Enter Apex Code':

	```apex
	// Execute the post-installation script to insert custom-settings data
	SCCConnectorPostInstallScript.insertCustomSettingsData();
	```

1. Click **Execute**.

	> Executing the insertCustomSettingsData() function seeds the Service Cloud instance with constants required by the service-cloud-connector.  You can view the generated log-file for execution details, making sure that the logs do not contain any references to 'error', 'exception', or 'fatal' keywords.

1. To test that the custom settings data was inserted successfully, execute the following [SOQL database query](https://trailhead.salesforce.com/content/learn/modules/apex_database/apex_database_dml).

	```sql
	/* Retrieve the Account field-mapping configuration values
	   Inserted as Custom Settings to Service Cloud */
	select  Id,
	        Field_Api_Name__c,
	        CC_Attribute__c,
	        Enable_Patch__c,
	        Enable_Sync__c,
	        CreatedById,
	        CreatedDate
	from    AccountFieldMapping__c
	```
	
With the post-installation script created, you can now begin to configure Service Cloud.

### Determine Which Callback URL to Use in the Connected App
As a precursor to creating the Connected App used by Service Cloud, you need to identify the type of Salesforce Org you're currently running.  This is necessary as each org-type (Development or Production) has a unique callback URL.  The two callback URLs are outlined in the following table.

> The deployment package should automatically set this value. This section has been included in the documentation supplementarily, describing how callback URLs are environment driven.

| Callback URL Type | URL |
|-------------------|-----|
| Production | https://login.salesforce.com/services/oauth2/success |
| Development | https://test.salesforce.com/services/oauth2/success |

If you are unsure what type of environment you currently have, review the following articles:

- [What are Salesforce instances?](https://success.salesforce.com/answers?id=90630000000gugYAAQ) provides an explanation about the different Salesforce instances that exist and how to tell what instance you're running by inspecting your instance URL.
- [How to identify the type of Salesforce Org](https://salesforce.stackexchange.com/questions/50/can-we-determine-if-the-salesforce-instance-is-production-org-or-a-sandbox-org) explains how to execute an SOQL query to identify your org's instance type.  If this query returns true, your Org is a sandbox.  Otherwise, your Org is a production instance.

	```sql
	/* Execute this ad hoc query to determine if you're running on a sandbox */
	SELECT IsSandbox FROM Organization LIMIT 1
	```

Once you have identified the instance-type of your Salesforce Org, use the corresponding Callback URL when modifying the CommerceCloudConnector Connected App as part of the next set of installation instructions.

### Modify the CommerceCloudConnector Connected App
With the SCC Integration user created, log in as that user.  The SCC Integration user must be logged in to modify the [Connected App](https://developer.salesforce.com/page/Connected_Apps) that is  used to facilitate connections to Salesforce B2C Commerce.

1. Click the Setup gear icon, and click **Setup**.
1. From the 'App' menu, select **App Manager**.
1. In the listing of applications, find the Connected App named CommerceCloudConnector.
1. Select **Edit** option from the dropdown menu on the right to edit the properties of the Connected App.
1. Verify and update the properties of the CommerceCloudConnector Connected App using the following table.

	> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be pre-set by the deployment package and post-installation script.

	| Property | Modify? |Value |
	|------------ |:-----:|---------- |
	| Connected App Name | No | CommerceCloudConnector |
	| API Name | No | CommerceCloudConnector |
	| Contact Email | Yes | system email associated to the account |
	| Enable OAuth Settings | Yes | checked |
	| Enable for Device Flow | Yes | checked |
	| Callback URL | Yes | Specify the Production or Development [Callback URL](#determine-which-callback-url-to-use-in-the-connected-app) |
	| Select OAuth Scopes | No | Full access (full) |
	| Require Secret for Web Server Flow | No | checked |
	
	> Specify a valid email address for the 'Contact Email' field.

	> The callback URL used here is automatically seeded by the deployment package.  If the URL does not align with your Salesforce Org type, update it appropriately using the guidance provided via the section titled [Determine Which Callback URL to Use in the Connected App](#determine-which-callback-url-to-use-in-the-connected-app).

1. Confirm that the form fields are completed and accurate.
1. Click **Save**.

### Configuring the Connected App
With the CommerceCloudConnector [Connected App](https://developer.salesforce.com/page/Connected_Apps) updated, it must be configured and associated to the internal org-user that is used to facilitate authenticated communications with Salesforce B2C Commerce.

#### Relax the CommerceCloudConnector's IP Restrictions
The first change we must make to the Connected App is to [relax IP restrictions](https://help.salesforce.com/articleView?id=connected_app_continuous_ip.htm&type=5).  This change allows the integration user to log in to the configured Salesforce B2C Commerce instance regardless of any [IP restrictions enabled on the user profile](https://salesforce.stackexchange.com/questions/83193/relax-ip-restrictions-in-connected-app).

To enable this as part of the Connected App's policy:

1. Click the Setup gear icon, and click **Setup**.
1. From the 'App' menu, select **App Manager**.
1. Find the CommerceCloudConnector connected app, in the application listing.
1. Select the 'manage' option for CommerceCloudConnector connected app.
1. From the properties display, click **Edit Policies** option.
1. Set the IP Relaxation Oauth policy property to 'Relax IP restrictions'.

	| Property | Value |
	|------------ | ---------- |
	| IP Relaxation | Relax IP restrictions |

1. Click **Save** to save your changes.

### Assign the Connected App to the SCC Integration User Profile
With the CommerceCloudConnector Connected App updated and configured, you can now assign it to the SCC Integration User profile.

1. From Setup, enter Profiles in the quick find box, then select **Profiles**.
1. From the profiles listing, open the SCC Integration User profile.
1. From the apps menu click the **Assigned Connected Apps**.
1. Find the 'Assigned Connected Apps' heading and click **Edit**.
1. From the edit view of the SCC Integration user profile, select the CommerceCloudConnector connected app.
1. Click **Add** to move the CommerceCloudConnector connected app to the 'Enabled Connected Apps' field.
1. Click **Save**.

The CommerceCloudConnector now appears as a connected app associated to this profile.

### Configure the Remote Site Settings
The [Remote Site Settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_callouts_remote_site_settings.htm) are used to configure the location and configuration properties of the Salesforce B2C Commerce endpoint to which Service Cloud connects to.

1. From Setup, enter Remote Site Settings in to the quick find box, then select **Remote Site Settings**.
1. From the listing, locate the remote site named 'SCCSG'.
1. Click **Edit** for this remote site, and update the following properties:

	> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be pre-set by the deployment package and post-installation script.

	| Property | Modify? | Value |
	|------------ |:----:| ---------- |
	| Remote Site Name | No | SCCSG |
	| Remote Site URL | Yes | https://xxxx-dw.demandware.net |
	| Description | Yes |  |
	| Active |  No | checked |
	| Disable Protocol Security |  No | unchecked |

	> Replace the default remote-site URL (https://xxxx-dw.demandware.net) with the URL representing the Salesforce B2C Commerce environment being integrated with Service Cloud.

1. Click **Save**.

The SCCSG record should now appear with the updated remote site URL.

### Configure the Named Credentials
Service Cloud leverages [named credentials](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) attached to the SCC Integration User's profile, which are now available to customize.

1. From Setup, enter Named Credentials in the quick find box, then select **Named Credentials**.
1. From the listing, verify that the *SFCCClientCreds*, *SFCCClientCredsBearer*, and *SFCCUserCred* credentials are present in the credential listing.

	> The __SFCCClientCreds__ credential should not be modified. It is used internally by the service-cloud-connector, and is already configured.

#### Update the SFCCClientCredsBearer Named Credential
The SFCCClientCredsBearer [named credential](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) interacts with the Salesforce B2C Open Commerce API (OCAPI).  Update the URL property to represent the Salesforce B2C Commerce environment being integrated with Service Cloud.

> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be pre-set by the deployment package and post-installation script.

| Property | Modify? | Value |
|------------ |:-------:|---------- |
| Label | No | SFCCClientCredsBearer |
| Name | No | SFCCClientCredsBearer |
| URL | *Yes* | https://xxxx-dw.demandware.net |
| Identity Type | No | Named Principal |
| Authentication Protocol | No | No Authentication |
| Generate Authorization Header | No | false |
| Allow Merge Fields in HTTP Header | No | checked |
| Allow Merge Fields in HTTP Body | No | checked |

> Replace the default remote-site URL (https://xxxx-dw.demandware.net) with the URL representing the Salesforce B2C Commerce environment being integrated with Service Cloud.

1. Click **Save**.



#### Update the SFCCUserCreds Named Credential
The SFCCUserCredsBearer [named credential](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) is used to enable the order-on-behalf of (OOBO) functionality within Service Cloud.  Update the properties of the named credential using the following values.

| Property | Modify? | Value |
|------------ |:---:| ---------- |
| Label | No | SFCCUserCreds |
| Name | No | SFCCUserCreds |
| URL | *Yes* | https://xxxx-dw.demandware.net |
| Identity Type | No | Per User |
| Authentication Protocol | No | No Authentication |
| Generate Authorization Header | No | checked |
| Allow Merge Fields in HTTP Header | No | unchecked |
| Allow Merge Fields in HTTP Body | No | unchecked |

> Replace the default remote-site URL (https://xxxx-dw.demandware.net) with the URL representing the Salesforce B2C Commerce environment being integrated with Service Cloud.

1. Click **Save**.


#### Configure the Authentication Settings for External Systems
To enable the OOBO functionality, the SFCCUserCreds [named credential](https://help.salesforce.com/articleView?id=named_credentials_about.htm&type=5) must be mapped to a specific Salesforce B2C Commerce environment Business Manager user account.  
To create this account mapping:

1. Log in as the SCC Integration User.
1. In the header, click the user profile icon, and click **Settings**.
1. From the 'My Personal Information' menu, click [Authentication Settings for External Applications](https://help.salesforce.com/articleView?id=external_authentication.htm&type=0).
- From the list of settings, click **New**.
- Create a set of credentials using the following property values:

	> Only modify the properties where the 'Modify?' column has a value of 'Yes'.  The other values should automatically be configured by the deployment package and post-installation script.

	| Property | Modify? | Value |
	|------------ |:---:| ---------- |
	| External System Definition | No | Named Credential |
	| Named Credential | No | SFCCUserCreds |
	| User | *Yes* | Select the SCC Integration User |
	| Authentication Protocol | *Yes* | Password Authentication |
	| Username | *Yes* | Specify the Salesforce B2C Commerce Account Username |
	| Password | *Yes* | Specify the Salesforce B2C Commerce Account Password:OCAPI Client Secret |

	> For the user, click the magnifying glass to view the list of Service Cloud users and select the SCCIntegration user.

	> The credentials entered here must represent an adequately permissioned Salesforce B2C Commerce environment Business Manager account.

	> The password MUST be in the format of *Account Password:OCAPI Client Secret*.  These two values must be delimited by a colon, and are used by Service Cloud to request an [authorization grant via OCAPI](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/OCAPI/19.5/usage/OAuth.html?cp=0_12_2_21) to interface with the [Data API](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/OCAPI/19.5/usage/DataAPIResources.html?cp=0_12_4).

1. Validate that the form values match these properties.
1. Confirm that the Salesforce B2C Commerce environment account exists.
1. Click **Save**.


### Update SFCC Site Information via the SCC Connector Custom Settings
To facilitate the communication between clouds, [custom settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_customsettings.htm) must be configured to point to the Salesforce B2C Commerce environment.

#### Configure the SFCC Configuration Custom Setting

1. From Setup, enter Custom Settings in the quick find box, then select **Custom Settings**.
1. Click **Manage** for the 'SFCC Configuration' setting.
1. Click **New**.
1. Create a configuration using the following values:

	> Only the fields marked as required should be completed.

	| Required | Property | Value |
	|:--------:| ------------------- | ---------- |
	| X        | Name     | [your-storefront-site-id](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/Admin/ManagingSitesInBM.html?cp=0_4_5) |
	| X        | Customer List Id | [your-storefront-customerList-id](documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/Customers/CustomerLists.html?cp=0_3_6_3) |
	|          | Master Catalog Id |  |
	|          | PriceBook Id |  |
	|          | Product Detailview |  |
	|          | Product Listview |  |
	| X        | SFCC Site URL | https://your-b2c-commerce-domain-dw.demandware.net  |
	|          | Sales Catalog Id |  |
	| X        | Site Id | your-site-id |
	| X        | Time | 1 |
	
	

	> Check your Salesforce B2C Commerce site configuration for the [siteId](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/Admin/ManagingSitesInBM.html?cp=0_4_5) and [customerListID](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/Customers/CustomerLists.html?cp=0_3_6_3) mapping properties.
	
	> Specify your Salesforce B2C Commerce environment URL for the SFCC Site URL property.

1. Click **Save** to save your changes.

	Once these changes have been saved, you are returned back to the listing of [custom settings](https://developer.salesforce.com/docs/atlas.en-us.apexcode.meta/apexcode/apex_customsettings.htm) in the Service Cloud environment.

#### Configure SFCC Integration Creds Custom Setting
The SFCC Integration Creds custom setting is used to store the clientId and secret used to interact with Salesforce B2C Commerce Open Commerce API (OCAPI).

1. Log out as the SCC Integration User.
1. Log in as the Service Cloud System Administrator.
1. From Setup, enter Custom Settings in the quick find box, then select **Custom Settings**.
1. Locate the 'SFCC Integration Creds' custom setting from the displayed list, and click **Manage**.
1. Click the top-most **New** to create the default credentials.
1. Update the credential properties with the following values:
	
	| Property      | Value        |
	| ------------- | ------------ |
	| ClientId      | your-site-clientid|
	| Client Secret | your-site-clientsecret |
	
	> For sandbox environments, the default values of __aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa__ and __aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa__ can be used.
	
	> The [clientId and clientSecret](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/OCAPI/19.5/usage/ClientApplicationIdentification.html?cp=0_12_2_3) must be created in Account Manager before being added to the storefront configuration.  For details on managing clientIds, read [Add a ClientID for OCAPI](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/AccountManager/AccountManagerAddAPIClientID.html?cp=0_7_11) and [Enable and Disable a ClientID for OCAPI](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/AccountManager/AccountManagerEnableDisableClientID.html?cp=0_7_12) via the platform documentation.
	
	> For production environments, these values are used to request an OAuth2 authorization token. Review the [OCAPI OAuth 2.0 platform documentation](https://documentation.b2c.commercecloud.salesforce.com/DOC1/topic/com.demandware.dochelp/OCAPI/19.5/usage/OAuth.html?cp=0_12_2_21) for details on how this is performed.

1. Click **Save** to save your changes.

	Once these changes have been saved, the default organization level values should display the clientId and client secret that were entered.

### CORS and Content Security
Lastly, the Service Cloud environment must be configured to allow communication with the Salesforce B2C Commerce environment.

#### Setup CORS
Update the [CORS settings](https://help.salesforce.com/articleView?id=extend_code_cors.htm&type=0) for the Salesforce environment to whitelist the Salesforce B2C Commerce URL.

1. From Setup, enter CORS in the quick find box, then select **CORS**.
1. From the list of whitelisted origins, click **New**.
1. Create a whitelist entry using the following properties:
	
	| Property      | Value        |
	| ------------- | ------------ |
	| Original URL Pattern   | https://xxxx-dw.demandware.net |
	
	> Enter the full domain name of the Storefront URL that is connecting to Service Cloud.  Be sure to include the https:// domain prefix.

1. Click **Save** to save your changes.

	Saving these changes should return you to the listing of whitelisted origins.  Confirm that the URL entered appears in this listing.

#### Content Security
Similar to the CORS setting, a [trusted site](https://help.salesforce.com/articleView?id=csp_trusted_sites.htm&type=0) must be defined within Service Cloud that maps to the Salesforce B2C Commerce Environment.    
To create a trusted site:

1. From Setup, enter CSP Trusted Sites in the Quick Find box, then select **CSP Trusted Sites**.
1. From the list of trusted sites, click **New Trusted Site**.
1. Create a trusted site entry using the following properties:

	| Property      | Value        |
	| ------------- | ------------ |
	| Trusted Site Name | CC_Endpoint |
	| Trusted Site URL  | https://xxxx-dw.demandware.net |
	| Active | checked |
	| Context | All |
	
	> For the Trusted Site URL, enter the full domain name of the Storefront URL that is connecting to Service Cloud.  Be sure to include the https:// domain prefix.

1. Click **Save** to save your changes.

	Saving these changes returns you to the listing of trusted sites.  Confirm that the entered site definition entered appears in this listing.

### Reset Your Security Token

>To successfully configure Salesforce B2C Commerce to interact with Service Cloud REST services, a security token must be generated and saved.

> Complete these activities before starting the Salesforce B2C Commerce configuration instructions.

The SCC Integration User account is used to facilitate service interactions between Service Cloud and Salesforce B2C Commerce.  To enable Salesforce B2C Commerce to connect to Service Cloud's REST Services, a Salesforce [Security Token](https://help.salesforce.com/articleView?id=user_security_token.htm&type=5) must be generated for the SCC Integration User account and passed into Service Cloud authentication requests.

To generate a new security token, execute the following set of instructions:

1. Log in as the SCC Integration User.
1. In a new browser window, go to the following URL:

	###### https://*your-salesforce-environment-domain*/_ui/system/security/ResetApiTokenEdit

	> Replace 'your-salesforce-environment-domain' with your actual Salesforce environment's domain.

1. Click **Reset Security Token**.

	This forces the existing Salesforce environment to generate a new security token for the logged in user (in this case, the SCC Integration User).

1. Check the email address associated to the SCC Integration User for the security token email.
1. When the token is received, store it in a secure place.

The generated security token is used when configuring the Salesforce B2C Commerce environment's site preferences that store the authentication settings for the Service Cloud environment. 

Save the generated security token, as it is used in future configuration, test-suite, and validation steps.


## Setting Up Salesforce B2C Commerce

###### This guide captures the configuration and installation activities necessary to enable the service-cloud-connector's integration with Salesforce B2C Commerce.

### What You Need to Get Started

Before moving forward with the installation instructions, verify that your cross-cloud environment meets all installation prerequisites.  

#### Installation Prerequisites

- A Salesforce B2C Commerce sandbox running a storefront based on [SiteGenesis](https://github.com/SalesforceCommerceCloud/sitegenesis) OR [Storefront Reference Architecture](https://github.com/SalesforceCommerceCloud/storefront-reference-architecture).
- A Service Cloud sandbox mirroring your production org with [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5) and [Lightning](https://www.salesforce.com/campaign/lightning/) enabled.

#### Disqualifying Configurations

- A Salesforce B2C Storefront running on the Pipelines.
- A Service Cloud environment that runs on Salesforce Classic.
- A Service Cloud environment that does not support [PersonAccounts](https://help.salesforce.com/articleView?id=account_person.htm&type=5).

> All installation prerequisites must be met in order for the service-cloud-connector to successfully deliver its complete feature-set.

### Configuring a Salesforce B2C Commerce Environment
This section of the document describes how to configure a Salesforce B2C Commerce environment to enable the service-cloud-connector's feature-set.

Be sure to select the SiteGenesis/SFRA-based site that will leverage the connector via the Site Selector found in the Business Manager header.

> In order to move forward with the configuration activities, ensure that you have access to a Business Manager account with Administrative privileges.

#### Import the Service Cloud Connector's Site Preferences
The service-cloud-connector introduces three site preferences to manage the operation of the service-cloud-connector within the Salesforce B2C Commerce environment.  This section of the document provides guidance on how to upload, import, and configure the site preferences.

##### Upload the System Object Definitions Import File
The following instructions describe how to upload the site preferences definition file so that it can be imported.

1. In Business Manager, select **Administration > Site Development > Import & Export**.
1. Click **Upload** in the 'Import & Export Files' section.
1. Click **Choose File** in the 'Upload Import Files' section.
1. Locate and select the [system-object](../../settings/sfsc/sfcc/meta/system-objecttype-extensions.xml) type definitions in the sfcc [meta](../../settings/sfsc/sfcc/meta) directory.
1. With the import file selected, click **Upload** to upload the import file.

##### Import the System Object Definitions
The following instructions describe how to import the system object definitions containing the service-cloud-connector's site preferences.

> The [system-object](../../settings/sfsc/sfcc/meta/system-objecttype-extensions.xml) import file must be uploaded to Business Manager as a pre-requisite to the import process.

1. In Business Manager, select **Administration > Site Development > Import & Export**.
1. Click **Import** in the 'Meta Data' section.
1. From the System Type Extension Import page, select the file named [system-objecttype-extensions.xml](../../settings/sfsc/sfcc/meta/system-objecttype-extensions.xml) and click **Next**.
1. The import process attempts to validate the [system-objecttype-extensions.xml](../../settings/sfsc/sfcc/meta/system-objecttype-extensions.xml) file.  Confirm that the file is validated without any errors, and click **Next** to continue.
1. Allow the import file to complete its processing, and confirm that the import is successful.

	> The 'Status' section shows the import progress of the file being processed.  A successful import has a process status of Success.

##### Configure the Imported Site Preferences
The following instructions describe how to import and configure the service-cloud-connector's site preferences.

1. In Business Manager, select **Merchant Tools > Site Preferences > Custom Preferences**.
1. Select the 'ServiceCloudConnector' site preference group displayed on the 'Custom Site Preference Groups' page.
1. Update the site preference using the following values:

	| Preference Name | Value | Description |
	| --------------- | ------| ------ |
	| Is async mode for Service Cloud? | No |  When set to 'No', the Service Cloud Connector drives cross-cloud data synchronization via its scheduled jobs; otherwise, synchronization is processed in real-time with the scheduled jobs operating as a fallback. |

1. Click **Save** at the top of the Custom Preferences display to save your changes.

	Confirm that the ServiceCloudConnector's custom preferences have been saved.  Once the preferences have been saved, continue with the import of the [custom object](../../settings/sfsc/sfcc/meta/custom-objecttype-definitions.xml).

#### Import the Service Cloud Connector's Custom Object
The service-cloud-connector introduces a custom object used to store the generated [Service Cloud AuthToken](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1) retrieved from Service Cloud during the authentication process.  This section provides guidance on how to import the [custom object definition](../b2c/sfcc/sites/site_template/meta/custom-objecttype-definitions.xml).

##### Upload the Custom Object Definition Import File
The following instructions describe how to upload the custom object definition file so that it can be imported.

1. In Business Manager, select **Administration > Site Development > Import & Export**.
1. Click **Upload** in the 'Import & Export Files' section.
1. Click **Choose File** in the 'Upload Import Files' section.
1. Locate and select the [custom-object](../../settings/sfsc/sfcc/meta/custom-objecttype-definitions.xml) definition file in the sfcc [meta](../../settings/sfsc/sfcc/meta) directory.
1. With the import file selected, click **Upload** to upload the import file.

##### Import the Custom Object Definition
The following instructions describe how to import the system object definitions containing the service-cloud-connector's site preferences.

> The [custom-object](../../settings/sfsc/sfcc/meta/custom-objecttype-definitions.xml) import file must be uploaded to Business Manager as a pre-requisite to the import process.

1. In Business Manager, select **Administration > Site Development > Import & Export.**
1. Click **Import** in the 'Meta Data' section.
1. From the import display, select the file named [custom-objecttype-definitions.xml](../../settings/sfsc/sfcc/meta/custom-objecttype-definitions.xml) and click **Next**.
1. The import process attempts to validate the [custom-objecttype-definitions.xml](../../settings/sfsc/sfcc/meta/custom-objecttype-definitions.xml) file.  Confirm that the file is validated without any errors, and click **Next** to continue.
1. Allow the import file to complete its processing, and confirm that the import is successful.

	> The 'Status' section shows the import progress of the file being processed.  A successful import has a process status of __Success__.

Once the import is complete, move forward with the importing of the Service Cloud service definition.

#### Import the Services Definition
The service-cloud-connector requires a service that must be configured to interact with the Service Cloud instance.

##### Upload the Service Definition
The following instructions provide guidance on how to import and configure the service definition.

1. In Business Manager, select **Administration > Operations > Import & Export**.
1. Click **Upload** in the 'Import & Export' section.
1. Click **Choose File** displayed in the 'Upload Import Files' section.
1. Locate and select the [services](../../settings/sfsc/sfcc/services.xml) definition file in the sfcc [sfcc](../../settings/sfsc/sfcc) directory.
1. With the import file selected, click **Upload** to upload the import file.

##### Import the Service Definition
The following instructions describe how to import the service definition leveraged by the service-cloud-connector.

> The [services](../../settings/sfsc/sfcc/services.xml) import file must be uploaded to Business Manager as a pre-requisite to the import process.

1. In Business Manager, select **Administration > Operations > Import & Export**.
1. Click **Import** in the 'Services' section.
1. Select the file named [services.xml](../../settings/sfsc/sfcc/services.xml) and click **Next**.
1. The import process attempts to validate the [services.xml](../../settings/sfsc/sfcc/services.xml) file.  Confirm that the file is validated without any errors, and click **Next** to continue.
1. From the 'Import' step, select **Merge** as the import mode, and click **Import** to process the uploaded file.
1. Allow the import file to complete its processing, and confirm that the import is successful.

	> The 'Status' section shows the import progress of the file being processed.  A successful import has a process status of __Success__.

Once the import is complete, move forward with the configuration of the Service Cloud service definition.

#### Configure the Service Definition
The following instructions describe how to configure the service definition leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Services**.
1. Click the **Credentials** tab.
1. Edit the _servicecloud.auth-SiteId_ and _servicecloud.rest-SiteId_ service definitions by clicking on the service names.
1. Update the suffix of the credential name and replace *SiteId* with your site's siteId.
1. Update the suffix of each service name and the log-prefix definition and replace *SiteId* with your site's siteId.
1. Click the **Credentials** tab.
1. Edit the _servicecloud.auth-SiteId_ service credentials by clicking the credential name.
1. Update the suffix of the credential name and replace *SiteId* with your site's siteId.
1. Configure the credentials for the service-cloud-connector service using the following properties:

	| Property Name | Value | Description |
	| ---------------- | -------------------------- | --------------------- |
	| URL | https://__auth-domain__/services/oauth2/token | Enter the Salesforce authentication url; use __login.salesforce.com__ for production and __test.salesforce.com__ for development sandboxes.|
	| User | Service Cloud SCC Integration User ID | Specify the username of the SCC Integration User for the Service Cloud environment. |
	| Password| Service Cloud SCC Integration User Password + SecurityToken | Specify the password and securityToken of the SCC Integration User for the Service Cloud environment. |
	| Consumer Key | ex. 9srSPoex34FV0eWhMyij4PEG4Cddk | Specify the Consumer Key associated to the Service Cloud CommerceCloudConnector Connected App. |
	| Consumer Secret| ex. 2928758854267628764 | Specify the Consumer Secret associated to the Service Cloud CommerceCloudConnector Connected App. |

	> Remember that the URL should use the *login* or *test* domain prefix depending on the Salesforce environment being connected via the service-cloud-connector.  You can review the guidance provided in the [Setup Service Cloud](./setupServiceCloud.md#determine-which-callback-url-to-use-in-the-connected-app) documentation to determine which URL to use.

	> The Consumer Key and Consumer Secret used for [OAuth2 authentication](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1) and can be retrieved by logging into Service Cloud and [viewing](https://help.salesforce.com/articleView?id=000205876&language=en_US&type=1) the CommerceCloudConnected Connected App via 'Setup'.

1. Once the service credentials and Service Cloud Connector consumer properties have been updated, click **Apply** to save your changes.

Among the imported service-cloud-connector services is a service named *servicecloud.rest*.  Do not modify the configuration of this service, as its profile and credentials are generated programmatically at runtime.

### Configuring the OCAPI Settings for the Service Cloud Connector
This section of the document describes how to configure the OCAPI Shop and Data API settings to support access to the resources required by the service-cloud-connector.

#### Configure the Shop API
The Open Commerce Shop API configuration settings should be extended to allow access form the connected Salesforce environment and include the resources required by the service-cloud-connector.

##### Configure the Allowed Origins for the Shop API
The OCAPI Shop API configuration should be extended to enable access from the connected Service Cloud environment.

1. In Business Manager, select **Administration > Site Development > Open Commerce API Settings**.
1. Select the API type Shop, and the storefront site that will be connected via the service-cloud-connector.
1. Specify version-format 18.8 for the OCAPI API resource-configuration document.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	  
	  ]
	}
	```

1. For this new version, specify the 'client_id' that will be used to access the Data API
1. In the 'allowed_origins' property of the JSON configuration file, include the Service Cloud instance's lightning and classic domain name that will be connected.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	    {
        "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "allowed_origins":["https://mydomain.lightning.force.com", "https://mydomain.my.salesforce.com"]
	    }
	  ]
	}
	```

	> Remember to append the new Service Cloud origins to the existing collection of origins.

	> At a minimum, the origin should include the instance_url returned by the [Salesforce remote authorization request](./validateEnvironments.md#supported-test-scenarios).  We recommend including the lightning and classic urls for your instance.

	> View this article to learn more about setting up a [my domain](https://help.salesforce.com/articleView?id=domain_name_overview.htm&type=5) for your instance.

##### Configure the Resource Definitions for the Shop API
The OCAPI Shop API resource definitions should be extended to include the resources required by the service-cloud-connector.

1. Add the following resource definitions to the existing 'resources' property of the JSON configuration file (See: [wapi_shop_config.json](../../settings/sfsc/sfcc/ocapi-settings/wapi_shop_config.json)).

	```json
	{
      "_v":"18.8",
      "clients":[
        {
          "allowed_origins":[
            "https://teamspirit-wiredbeans--sandbox06.lightning.force.com",
            "https://teamspirit-wiredbeans--sandbox06.cloudforce.com",
            "https://teamspirit-wiredbeans--sandbox02.lightning.force.com",
            "https://teamspirit-wiredbeans--sandbox02.cloudforce.com",
            "https://teamspirit-wiredbeans--scdevelope.lightning.force.com",
            "https://teamspirit-wiredbeans--scdevelope.cloudforce.com"
          ],
          "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          "resources":[
            {
              "resource_id":"/customers/*",
              "methods":[
                  "get"
              ],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/customers/*/auth",
              "methods":[
                  "post"
              ],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/orders/*",
              "methods":[
                  "get",
                  "patch"
              ],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/orders/*/payment_methods",
              "methods":[
                  "get"
              ],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/sessions",
              "methods":[
                  "post"
              ],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            }
          ]
        }
      ]
    }
	```


	> When adding the service-cloud-connector resources,  review your existing resources and ensure that you do not overwrite them wholesale. Only add the resources that are not already included in the existing OCAPI resources.

1. With both changes applied to the Shop API configuration .json file, click **Save** to save your changes.

#### Configure the Data API
Similar to the Shop API, the Open Commerce Data API configuration settings should be extended to allow access form the connected Salesforce environment and include the resources required by the service-cloud-connector.

##### Configure the Allowed Origins for the Data API
The OCAPI Data API configuration should be extended to enable access from the connected Service Cloud environment.

1. In Business Manager, select **Administration > Site Development > Open Commerce API Settings**.
1. Select the API type Data, and Global (organization wide) from the OCAPI toolbar.
1. Specify version-format 18.8 for the OCAPI API resource-configuration document.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	  
	  
	  ]
	}
	```

1. For this new version, specify the 'client_id' that will be used to access the Data API
1. In the 'allowed_origins' property of the JSON configuration file, include the Service Cloud instance domains added as part of the Shop API configuration.

	```json
	{
	  "_v":"18.8",
	  "clients": [
	    {
        "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "allowed_origins":["https://mydomain.lightning.force.com", "https://mydomain.my.salesforce.com"]
	    }
	  ]
	}
	```

	> Remember to append the new Service Cloud origin to the existing collection of origins.

	> At a minimum, the origin should include the instance_url returned by the [Salesforce remote authorization request](./validateEnvironments.md#supported-test-scenarios).  We recommend including the lightning and classic urls for your instance.

	> View this article to learn more about setting up a [my domain](https://help.salesforce.com/articleView?id=domain_name_overview.htm&type=5) for your instance.

##### Configure the Resource Definitions for the Data API
The OCAPI Data API resource definitions should be extended to include the resources required by the service-cloud-connector.

1. Add the following resource definitions to the existing 'resources' property of the JSON configuration file (See: [wapi_data_config.json](../../settings/sfsc/sfcc/ocapi-settings/wapi_data_config.json)).

	```json
	{
      "_v":"18.8",
      "clients": [
        {
          "allowed_origins":[
            "https://teamspirit-wiredbeans--sandbox06.lightning.force.com",
            "https://teamspirit-wiredbeans--sandbox06.cloudforce.com",
            "https://teamspirit-wiredbeans--sandbox02.lightning.force.com",
            "https://teamspirit-wiredbeans--sandbox02.cloudforce.com",
            "https://teamspirit-wiredbeans--scdevelope.lightning.force.com",
            "https://teamspirit-wiredbeans--scdevelope.cloudforce.com"
          ],
          "client_id":"aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
          "resources":[
            {
              "resource_id":"/customer_lists/*",
              "methods":["get"],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/customer_lists/*/customer_search",
              "methods":["post"],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/customer_lists/*/customers/*",
              "methods":["get","put","patch","delete"],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/customer_lists/*/customers/*/addresses",
              "methods":["get","post"],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/customer_lists/*/customers/*/addresses/*",
              "methods":["get","patch","delete"],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            },
            {
              "resource_id":"/sites/*/promotion_search",
              "methods":["post"],
              "read_attributes":"(**)",
              "write_attributes":"(**)"
            }
          ]
        }
      ]
    }
	```

	> When adding the service-cloud-connector resources, review your existing resources and ensure that you do not overwrite them wholesale.  Only add the resources that are not already included in the existing OCAPI resources.

1. With both changes applied to the Data API configuration .json file, click **Save** to save your changes.

## Set up the Salesforce B2C Commerce service-cloud-connector Jobs

###### This guide captures the configuration and installation activities necessary to set up the int\_service\_cloud cartridge and configure the storefront jobs used to synchronize data across both clouds.

> The setup of the connector cartridge's jobs and storefront extensions depend on these two steps being completed.  That said, complete the installation instructions found in these two pages before proceeding with the cartridge setup.

### Install the int\_service\_cloud Cartridge

To enable the features of the service-cloud-connector, the [int\_service\_cloud](../int_service_cloud) cartridge must be installed in your storefront and set up in the storefront's cartridge path.

1. Upload the cartridge folder to your current code-version within your environment.
1. Log into Business Manager with an account that has Administrative rights to the site being integrated with Service Cloud.

#### Trigger the Recognition of the Service Cloud Connector's Job-Steps

The two jobs used to synchronize customer and order data with Service Cloud depend on two job-step definitions. The job-step definitions must be parsed by the Salesforce B2C Commerce Cloud instance where the service-cloud-connector is being deployed. Follow these instructions to trigger the parsing of these job-steps from the [int\_service\_cloud](../int_service_cloud) cartridge.

1. In Business Manager, select **Administration > Site Development > Code Deployment**.
1. Set another code-version to be the active code version.  
This code-version should be different from the version that currently contains the [int\_service\_cloud](../int_service_cloud) cartridge uploaded to your Salesforce B2C Commerce instance.
1. Set the code-version where the [int\_service\_cloud](../int_service_cloud) cartridge was deployed as the Active code version by clicking **Activate**.

	> Toggling the code-versions force Salesforce B2C Commerce to scan the cartridge path for the site and parse any job-steps configured within the cartridges identified in the cartridge-path.
1. Confirm that the selected code-version is now flagged as Active in the code-versions listing (this should be the version with the [int\_service\_cloud](../int_service_cloud) cartridge).

1. Open a new browser and load the homepage of your site to confirm that your site is able to successfully parse the cartridge path.  If the homepage for the site you edited successfully renders, continue to the next section of this document.

### Create the Customer Synchronization Job

The service-cloud-connector uses the customer-synchronization job to synchronize customer profiles updated in Commerce Cloud with Service Cloud. 

#### Create the Stub Customer-Job Definition

Complete these instructions to create the customer-synchronization job leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Jobs**, and click **New Job**.
1. Enter the following properties:

	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-customer-sync|
	| Description   | This job is used to synchronize customer data changed in Salesforce B2C Commerce Cloud with Service Cloud. |
	
1. Click **Create** to create the job stub.

#### Configure the scc-customer-sync Job's Properties

Complete these instructions to create the customer-synchronization job leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Jobs**, and click **scc-customer-sync**.
1. From the 'scc-customer-sync' job schedules display, click the **Job Steps** tab.
1. On the Job Steps tab, click the Configure a Step icon centered on the display.
1. In the Select and Configure Step sidebar, select the job-step labeled Custom-SCC-SyncCustomers, and enter the foollowing configuration properties:

	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-customer-sync-jobstep|
	| Description   | This job-step is used to synchronize customer data changed in Salesforce B2C Commerce with Service Cloud. |
	| Restart Enforced | unchecked |
	
	> Do not configure any error handling behavior.

1. Click **Assign**.

	> The job-step should appear in the 'scc-customer-sync' display highlighted in red.  The job step has an error, as its scope has not been configured.

1. From the 'scc-customer-sync' Job Steps tab, click the **Scope** button.
1. Select the site being integrated with Service Cloud (ex. SiteGenesis/RefArch).
1. Click **Assign** to associate this site to the job-step.

	Assigning the site being integrated with Service Cloud completes the configuration of scc-customer-sync job.

	> The scc-customer-sync job leverages the [customerSync.js](../b2c/sfcc/cartridges/int_service_cloud/cartridge/scripts/jobs/customerSync.js) script to perform the Salesforce B2C Commerce to Service Cloud customer synchronization.  View the [steptypes.json](../b2c/sfcc/cartridges/int_service_cloud/steptypes.json) file for custom job-step configuration details.

With this completed, now create the order synchronization job.

### Create the Order Synchronization Job

Similar to the customer synchronization job, the service-cloud-connector makes use of a job to synchronize orders placed via Commerce Cloud with Service Cloud.  

#### Create the Stub Order-Job Definition

Complete these instructions to create the order-synchronization job leveraged by the service-cloud-connector.

1. In Business Manager, select **Administration > Operations > Jobs**, and click **New Job**.
1. Enter the following properties:
	
	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-order-sync|
	| Description   | This job is used to synchronize order placed via Salesforce B2C Commerce with Service Cloud. |

- Click **Create**.

#### Configure the scc-order-sync Job's Properties

Complete these instructions to create the order-synchronization job leveraged by the service-cloud-connector.


1. In Business Manager, select **Administration > Operations > Jobs**, and click **scc-order-sync**.
1. From the 'scc-order-sync' job schedules display, click the **Job Steps** tab.
1. On the Job Steps tab, click the Configure a Step icon centered on the display.
1. In the Select and Configure Step sidebar, select custom.SCC-SyncOrders, and enter the following configuration properties:

	| Property Name | Property Value |
	|---------------|------------------------|
	| ID            | scc-order-sync-jobstep|
	| Description   | This job-step is used to synchronize orders placed via Salesforce B2C Commerce with Service Cloud. |
	| Restart Enforced | unchecked |

	> Do not configure any error handling behavior.

1. Click **Assign**.

	> At this point, the job-step should appear in the 'scc-order-sync' display highlighted in red.  The job step has an error, as its scope has not been configured.

1. From the 'scc-order-sync' Job Steps tab, click the **Scope** button.
1. Select the site being integrated with Service Cloud (ex. SiteGenesis/RefArch).
1. Click **Assign** to associate this site to the job-step.

	Assigning the site being integrated with Service Cloud completes the configuration of scc-order-sync job.

> The scc-order-sync job leverages the [orderSync.js](../int_service_cloud/cartridge/scripts/jobs/orderSync.js) script to perform the Salesforce B2C Commerce Cloud to Service Cloud customer synchronization.  View the [steptypes.json](../int_service_cloud/steptypes.json) file for custom job-step configuration details.

### Scheduling the Customer and Order Synchronization Jobs

In your PIG, configure the Customer and Order jobs to execute at a high frequency and regular interval (ex. once every five minutes).  In a non-production environment (ex. a development sandbox), remember to execute these jobs manually.

> Development environments cannot schedule jobs.  When testing in your development environment, manually execute these jobs after performing customer or order-related transactions.

### Configure the Synchronization of Customer and Order Information

The custom site preference 'Is async mode for Service Cloud?' can be used to modify the behavior of the synchronization jobs and how they trigger data-synchronization between Salesforce B2C Commerce and Service Clouds.

| Preference Value | Impact on Behavior |
|------------------|--------------------|
| None | Select either 'Yes' or 'No'. This site preference should not be configured with a value on 'None'.   |
| Yes  | This causes all data-synchronizations to be driven by the service-cloud-connector jobs.  No real-time synchronization is performed when interacting with customer or order data. |
| No   | This causes all data-synchronizations in real-time.  The service-cloud-connector jobs only process synchronization failures. |

Synchronization jobs are always running.  What this preference does is slightly modify which records are synchronized when the job executes.  We recommend defaulting this value to No, which enables real-time synchronization, which is ideal for development environments.

> These jobs should be scheduled to run __every five minutes__ ensuring that changes made by customers (ex. customer registration, profile update, order placement, etc.) are synchronized with Service Cloud in a timely fashion.  This interval should consider the creation or update frequency of these storefront objects -- as a shorter interval impacts storefront performance. Adjust the interval accordingly based on your storefront's customer or order frequency, and validate that the setting does not negatively impact storefront performance before deploying this change to production.

### Configure the Customer and Order Synchronization Batch Size

The custom site preference 'Bulk Call Threshold' can be used to manage the size of the customer and order batches sent to Service Cloud for every job-instance executed by Salesforce B2C Commerce Cloud.

| Preference Value | Site Preference Description |
|------------------|--------------------|
| 10 | This site preference describes the value used to determine what portion of customers or orders that have updates and are ready to by synchronized with Service Cloud should be sent via service-cloud-connector's synchronization jobs. |

The business rules used to determine scheduled job batch-size are:

- The job calculates the total number of items (customers and orders) that have updates to share with Service Cloud.
- That total is divided by the 'Bulk Call Threshold' preference value.
- The product of that is then rounded up to the next highest integer.
- The rounded value represents the maximum batch size for the service-cloud-connector's scheduled jobs.

> For example: 100 items found and the preference value is 10; 100 / 10 = the first 10 items of the 100 identified are batched to Service Cloud

> For example: 9 items found and the preference value is 10; 9 / 10 = the first (1) item of the 100 identified are batched to Service Cloud

You can increase the size of batches containing orders or customers by decreasing this preference value.

| Preference Value | Impact on Behavior |
|----------:|--------------------|
| 1 | Sends 100% of all identified items in the batch to Service Cloud |
| 2 | Sends 50% of all identified items in the batch to Service Cloud  |
| 4 | Sends 25% of all identified items in the batch to Service Cloud  |
| 10 | Sends 10% of all identified items in the batch to Service Cloud  |

> If the total number of identified items is less than this preference value, then only one item is included in the batch used to send updates to Service Cloud.

### Install the plugin\_service\_cloud and configure Cartridge path

To enable the features of the service-cloud-connector, the [plugin\_service\_cloud](../plugin_service_cloud) and [int\_service\_cloud](../int_service_cloud) cartridges must be installed in your storefront and setup in the storefront's cartridge path.

1. Upload the cartridge folder to your current code-version within your environment.
1. Log into Business Manager, using an account that has Administrative rights to the site being integrated with Service Cloud.
1. In Business Manager, select **Administration > Site > Manage Sites**.
1. Select the site being integrated to Service Cloud.
1. Select the Settings tab.
1. Update the cartridge path to include the 'plugin\_service\_cloud' and 'int\_service\_cloud' cartridges.  Be sure to include the cartridges near the front of the cartridge-path before any core cartridges.

	```
	plugin_service_cloud:int_service_cloud:app_storefront_base
	```

	> In this example, the SFRA cartridge path is represented. Remember to place the *plugin\_service\_cloud:int\_service\_cloud* reference before the main controller cartridge for your storefront.

1. Click **Apply** to save your changes to the cartridge path.
