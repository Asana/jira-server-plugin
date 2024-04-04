# How to Authenticate After Jira Server Is Upgraded to OAuth 2.0 (Usually from 8.x without OAuth 2.0 Support to 9.x)

Notes:

    The usual scenario is when a user has Jira Server/DC 8.x without OAuth 2.0 support (>8.22) and wants to upgrade to 9.x with OAuth 2.0 support.
    The Asana Jira Server plugin is used only for Jira versions with no OAuth 2.0 support.

1. How to Re-Authenticate:

   - Users need to set up an OAuth 2.0 connection between Jira Server/DC and Asana.
   Refer to https://github.com/Asana/jira-server-plugin/blob/main/HowtoSetupOauth2Applink.md for instructions.
   This will create a new application link.
   - Users can delete the old application link (created by the Jira Server plugin) (not required).
   - Users can uninstall/delete the Jira Server plugin from Jira Server/DC since it's no needed anymore.

2. Ensuring the Jira Server Widget Is Working:

   - Check that users can create Jira entities via the Asana Jira Widget.
   - Check that users can link existing entities from Jira to Asana.

3. Ensuring Jira Server Automation Is Working:

   - Check that users can create app rules and automate task creation in Jira from Asana.