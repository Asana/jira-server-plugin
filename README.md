# Asana for Jira Server: Admin Guide

Welcome to the Asana for Jira Server Admin Guide! Here we will help you identify if your Jira Server configuration is 
compatible with the **Asana for Jira Server Integration** and walk through a few common configuration scenarios that 
might help you troubleshoot getting set up.

Note that every Jira Server deployment can be configured differently, so while the Asana for Jira Server Integration 
supports a number of different configurations, not all Jira Servers will be capable of working with the integration. 
Before we move forward, here are some requirements that should help you determine if your Jira Server will be compatible
with the integration:

# Requirements
## 1. You are using a supported version of Jira Server

Currently, the Asana integration supports the following version of Jira Server: 
[Jira Server 8.12](https://confluence.atlassian.com/jirasoftwareserver0812)

It is possible that other versions of Jira Server will work with the integration and we do not explicitly restrict the 
integration from being used with other minor Jira Server 8 versions. However, keep in mind that these are the two 
versions that we test against and you may experience unexpected issues with other versions.

## 2. Your Jira Server’s Rest API is accessible to the integration via HTTPS

The Asana integration is a cloud-based service that securely communicates with your Jira Server’s 
[REST API](https://developer.atlassian.com/server/jira/platform/rest-apis/) via an 
[Application Link](https://confluence.atlassian.com/applinks/application-links-documentation-165120834.html). The 
following endpoints must be accessible to the integration via HTTPS (where “base_url” is your Jira Server’s 
[externally accessible](#3-your-jira-servers-dns-name-resolves-to-an-externally-valid-ip) hostname):

- `GET https://<base_url>/`
- `GET https://<base_url>/oauth_authorize`
- `GET https://<base_url>/oauth_token`
- `GET https://<base_url>/plugins/servlet/oauth/request-token`
- `GET https://<base_url>/plugins/servlet/oauth/authorize`
- `GET https://<base_url>/rest/api/2/issue`
- `GET https://<base_url>/rest/api/2/issue/createmeta`
- `GET https://<base_url>/rest/api/2/issue/${jiraIssueKey}/attachments`
- `GET https://<base_url>/rest/api/2/serverInfo`
- `POST https://<base_url>/asana_auth/create_applink`

If you’d like to specify the Asana integration on an “allow list” so that you can continue to restrict other external 
traffic to these endpoints, the integration will use the following static IP address:

- `52.42.159.79`

## 3. Your Jira Server’s DNS name resolves to an externally valid IP

The Asana integration uses [OAuth](https://developer.atlassian.com/server/jira/platform/oauth/) to authenticate 
requests from individual users in your Jira Server instance. OAuth1 requires that each request includes a signature 
parameter that includes a valid hostname for your Jira Server. 

This means that the external DNS name for your Jira Server must resolve to an externally accessible IP address. If 
your Jira Server’s DNS name resolves to an IP address that is only accessible from a corporate intranet or VPN, you 
will not be able to install the Asana integration without making some DNS modifications.

If you are open to making those changes, check out [this section](#Configuring-DNS-resolution) of this guide.

# Common scenarios
## Configuring DNS resolution

It’s common for some Jira Servers to have a DNS name which actually resolves to an internal IP address that is only 
accessible from a corporate intranet or VPN.

This presents a problem for the Asana integration’s 
[OAuth requirements](#3-your-jira-servers-dns-name-resolves-to-an-externally-valid-ip). If you’re in this situation 
you have two paths forward:

##### Configure your DNS servers to resolve the same DNS name in different way for internal and external clients
This should be relatively straight forward. Say your Jira Server currently lives at `jira.example.com` . You’d have 
to configure your DNS server to resolve `jira.example.com` to one IP address for intranet users and another, 
externally valid IP for external internet users (like the Asana integration).

##### Use different DNS names for internal and external clients
In this situation, you’ll set up a separate DNS name to use with external clients like the Asana integration.

Let’s say that we have the following hostname: `https://jira.example.com:2991`. This hostname resolves to an IP addresses that are valid on your company intranet. For the sake of example, let’s say those IP addresses are:

- `172.31.68.196`
- `172.31.67.104`

What we’ll want to do is set up a different DNS name, say `https://jira-external.example.com:2990` to use with the 
Asana integration. We’ll do this by setting up a net work load balancer (NLB) that is configured to listen on a 
particular port and redirect to our two internal IP addresses `172.31.68.196` and `172.31.67.104`. Note that the NLB 
is just a TCP load balancer, it should not modify any headers and should forward requests unchanged to the internal IP 
addresses and with the same hostname.

Finally, you’ll have to modify your Jira Server configuration files to validate requests to a hostname that is 
different than the one that Jira is using as its base URL. This can be achieved by modifying the `server.xml` file in 
your Jira Server instance to include the following configuration:

```xml 
<?xml version="1.0" encoding="utf-8"?>
<Server port="{{ atl_tomcat_mgmt_port | default('8005') }}" shutdown="SHUTDOWN">
    <Service name="Catalina">
        <Connector scheme="{{ atl_tomcat_scheme | default(catalina_connector_scheme) | default('http') }}" proxyName="jira-externlb.integrations.asana.plus" proxyPort="2991" />
    </Service>
</Server>
```

For more details on where to place or find your `server.xml`, please consult this 
[article](https://confluence.atlassian.com/kb/reverse-proxy-and-application-link-troubleshooting-guide-719095279.html) 
in Atlassian’s documentation.

## Testing locally

It may be helpful to test the integration in a local staging environment before attempting to configure it with your 
production Jira Server instance.

1. Start your local server using [atlas-run](https://developer.atlassian.com/server/framework/atlassian-sdk/atlas-run/):
    ```text
    $ atlas-run
    [INFO] Starting jira on the tomcat8x container on ports 2990 (http), 52221 (rmi) and 8009 (ajp)
    [INFO] [talledLocalContainer] Tomcat 8.x started on port [2990]
    [INFO] jira started successfully in 377s at http://localhost:2990/jira
    [INFO] Type Crtl-C to shutdown gracefully
    ```
2. Use ngrok to start a tunnel your local server’s port:
    ```text
    $ ngrok http 2990
    ```
   This should output an alias similar to `http://ac385719247c.ngrok.io`
3. Navigate to http://localhost:2990/jira and login as a user with admin permissions.
    ![http://localhost:2990/jira](./assets/a8xWgf6I.png)
4. Create a sample project in Jira
    ![Create a sample project in Jira 1](./assets/0e2imt6w.png)
    ![Create a sample project in Jira 2](./assets/h3u5GxEU.png)
5. Go to the **Adminstration** panel and under **General configuration** set the Base URL to your ngrok alias + 
“/jira”. For example: http://ac385719247c.ngrok.io/jira
    ![Set Base URL](./assets/-x3cBbRk.png)
6. Install the Asana for Jira Server plugin
    
    If you receive an error stating that your “Jira URL isn’t accessible,” return to step 5
7. Navigate to an Asana project where you would like to install the Asana for Jira Server integration
    1. Click on the drop-down arrow next to your project header
    2. Select **Add apps**
    3. Select **_Jira Server_**
    4. You’ll be prompted by a full screen installation wizard
    5. Authorize your Jira Server account
    ![Authorize your Jira Server account](./assets/4dWRGLDY.png)
    6. After you’ve authorized, open a task in your project and you should see a Jira Server field: Click it and 
    select **Create new issue** from the dropdown
    ![Asana - Add Jira Issue](./assets/eR1RggpY.png)
8. A form should load allowing you to fill out the details of a new issue. If this was successful then you’ve correctly
configured the Asana for Jira Server integration for your local Jira Server instance.
    ![Create new issue](./assets/efxw3b5g.png)