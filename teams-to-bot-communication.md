This example scenario helps understand how the communication flow works for bot application integrated into Microsoft Teams. This article covers the two use-cases that are messaging bots and calling bots - You'll find answers to frequently asked questions and best practices on how-to integrate and secure the network connectivity for your bots for Microsoft Teams.

## Architecture

<image type="content" alt-text="Diagram showing the Teams-to-bot communication flowchart." src="media/teams-to-bot-communication-diagram.png" lightbox="media/teams-to-bot-communication-diagram.png">

*Download a [Visio file](/media/teams-to-bot-communication.vsdx) of this architecture.*

### Dataflow

1. Teams app manifest contains the definition of the bot capabilities and application ID registered in Azure AD - Users in Teams can only send and receive messages to/from your bot if they have the application installed to them in Teams. 

2. The Microsoft Teams client can run from any location or device (web / desktop / mobile) as long as it has access to Microsoft Office 365 endpoints defined in [Managing Office 365 endpoints](https://docs.microsoft.com/en-us/microsoft-365/enterprise/managing-office-365-endpoints) - There are no extra IP / ports / protocol / FQDN required to use bots in Microsoft Teams.

3. Bot messages and calls signalling transit via Microsoft Teams Service
   - 3a - for **messaging bots**, chat messages are sent/received to/from the Microsoft Teams service, hosted by Microsoft.
   - 3b - for **calling bots**, Microsoft Teams Service sends the notification for incoming calls and provides the endpoints for the media streams.

4. The Azure Bot Service is required for the registration of your bot including:
   - The bot name, description and logo
   - The supported authentication type (single-tenant / multi-tenant / user-managed identities)
   - The associated app ID and app registration in Azure AD
   - Activate the channels and set the bot endpoints
   - Other settings like OAuth provider or public access  
   
   Your bot receives messages / notifications coming from Microsoft Teams Service directly. There is no traffic that transit via the Azure Bot Service. The Microsoft service reply URL is in the form https://smba.trafficmanager.net/{region} where region depends on the location for your Office 365 service (e.g. emea / amer / in / apac / ...)
   <br>
   
   **Bot endpoints for Teams messaging (4a) and calling (4b) are independantly configured and don't have the same requirements for network configuration**.

   **4a - messaging bot**
   | Source | Destination | Role | Port | Protocol |
   |---|---|---|---|---|
   | Microsoft Teams Service (52.112.0.0/14) | Your messaging Bot | Communication channel for activity messages | 443 | TCP |
  
   **4b - calling bot**
   | Source | Destination | Role | Port | Protocol |
   |---|---|---|---|---|
   | Microsoft Teams Service (52.112.0.0/14) | Your calling Bot | Media port range | 16384 - 65535 | UDP |
   | Any ([Microsoft IP range](https://www.microsoft.com/en-us/download/details.aspx?id=56519)) | Your calling Bot | 	Media control port for Teams | 8445 | TCP |


5. Your bot needs access to Microsoft services to do operations like validate the JWT token sent in the HTTP Authorization header or facilitate SSO (single-sign-on) As the list of IP addresses may vary over time, it is recommended to implement FQDN based filtering on **login.microsoftonline.com** and **login.botframework.com**. 

6. If your bot requires additional permissions to perform operations on your Office 365 environment, you need to trigger an authentication flow to get the appropriate access token from Azure AD. A best practice is to implement user managed identities that simplifies and secures the management of application secrets. Conversational bot will generally use a delegated permission (on-behalf-of the connected user) whereas calling bots will require application permission to have control over the call (hang-off / redirect / join participants / access the audio stream / ...) - Your bot needs access to the domain **graph.microsoft.com** to query the Microsoft Graph API.

**Teams messaging bots (aka conversation bots)**

This type of bot is used to implement chat based interaction between a user in Teams and your bot. This is usually a 2-way communication channel (user sends a chat message to the bot and gets an answer) but can be configured for notification only (bot sends message but user can't query or answer) - Click here for detailed instructions to [create a Teams conversation bot](https://learn.microsoft.com/en-us/microsoftteams/platform/sbs-teams-conversation-bot)

**Teams calling bots**

This type of bot is used to implement voice based interaction between a user in Teams and your bot. The bot will be able to answer an incoming call or to join a call and manage its lifecyle - Calling bots are also used for compliance reording in regulated industries - Click here for detailed information on the [calls and online meetings bots](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/calls-and-meetings/calls-meetings-bots-overview) and [compliance recording for Microsoft Teams](https://learn.microsoft.com/en-us/MicrosoftTeams/teams-recording-policy)


## Scenario details

Bots allow Teams users to interact with web services through text, interactive cards, and task modules. The Microsoft Bot Framework and Azure Bot Services give you an easy-to-use set of tools for creating and managing these bots.

You can develop bots by using a variety of languages, such as C\#, JavaScript, and Python. After they're developed, you can deploy them to Azure. A key component of a bot is the web app, which contains the core logic and interface that users communicate with. One of the key requirements for the bot to work is that it must expose a publicly accessible HTTPS endpoint.

InfoSec policy commonly requires that all incoming traffic to web apps go through a corporate firewall. This means that all traffic that goes to a bot, and responses from the bot, must route through a corporate firewall, as with any other web app.

### Potential use cases

Organizations can utilize bots for mobile and desktop users. Some examples include:

- Simple queries. Bots can deliver an exact match to a query or a group of related matches to help with disambiguation.
- Multi-turn interactions. By helping anticipate possible next steps, bots make it much easier for people to a complete task flow.
- Reaching out to users. Bots can send a message (aka notification) when something has changed in a document or a work item is closed.
- Bots can be integrated in multiple ways into Microsoft Teams: as a personal application, in a channel or group chat, as a message extension (to easily search & share data) or in a meeting.
- Calling bots are a specific use-case enabled for Microsoft Teams where the bot can respon to incoming calls, manage participants, process audio & video media streams and more.

## Frequently asked questions (FAQ)

### Do user-data (e.g. chat messages) transit via the Azure Bot Service with Microsoft Teams channel? 

**NO** - There is no user-data that transit via the Azure Bot Service for Microsoft Teams channel (both for the messaging and calling endpoints) - For 1st party channel as Microsoft Teams, Outlook, Skype, Search (Preview) and Direct Line Speech, user-data directly go to Microsoft service endpoint and do not transit via the Azure Bot Service.

### Can we "Disable public access and use private access" for bots in Microsoft Teams?

**NO** - Microsoft Teams is SaaS and only provides public endpoints that needs to be joined by Microsoft Teams client - "Disable public access" should only be used in combinaison with [Direct Line App Service extension](https://learn.microsoft.com/en-us/azure/bot-service/dl-network-isolation-concept)

### Can we host a bot for Microsoft Teams outside of Azure?

**It depends on the scenario** - Messaging bots can be hosted on any infrastructure as long as all required FQDN, IP addresses and ports (in & out) are whitelisted - On the contrary, calling bots can only be hosted on Microsoft Azure and specific services - cf. the [Requirements and considerations for application-hosted media bots](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/calls-and-meetings/requirements-considerations-application-hosted-media-bots) page



## Contributors

*This article is maintained by Microsoft. It was originally written by the following contributors.*

Principal author:

- [Alexis Kinzelin](https://www.linkedin.com/in/alijafry) | Cloud Solution Architect

## Next steps

- [Build bots for Microsoft Teams](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/what-are-bots)
- [Connect a bot to Microsoft Teams](https://learn.microsoft.com/en-us/azure/bot-service/channel-connect-teams)
- [Register a calling bot for Microsoft Teams](https://microsoftgraph.github.io/microsoft-graph-comms-samples/docs/articles/calls/register-calling-bot.html)
- [Help secure your Microsoft Teams channel bot and web app behind a firewall](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/teams/securing-bot-teams-channel)

## Related resources

- Visit the [Azure Architecture Center](../../browse/index.yml) to review related architectures and guides.

- [Azure and Microsoft 365 scenarios](https://learn.microsoft.com/en-us/azure/architecture/solutions/microsoft-365-scenarios)

- [Build applications on the Microsoft Cloud](https://learn.microsoft.com/en-us/azure/architecture/guide/microsoft-cloud/overview)

- [Azure Firewall Architecture Guide - Azure Architecture Center](../firewalls/index.yml)

- [Azure Active Directory IDaaS in Security Operations - Azure Example Scenarios](../aadsec/azure-ad-security.yml)




