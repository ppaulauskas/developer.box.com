# Shield Alert Collection
Box Shield Alerts that were introduced to our Enterprise Event Stream are unique in providing actionable notification rather than a snapshot of user and content actions in enterprises with [Box Shield][box-shield] enabled. Considering this, giving them special attention in your development process can lead significant additional value.

This guide will cover a few different topics around Box Shield events and applications:
1. Authentication Options - Deeper look into the differences between 3-Legged OAuth 2.0 and JWT Based OAuth 2.0
1. Scopes - The scopes required at the user level and application level to retrieve / view these events
1. Box Shield Events - Deeper dive into the different event types / schema delivered by Shield

## Authentication Options
There are two cases to consider here - whether you are building a new integration and need guidance on which option to choose or if you currently have an existing integration but want to explore making a change. If you are have an existing integration and have no interest in changing your authentication, this section may be unnecessary. We would still suggest giving the section on JWT auth a glance as it provides some benefits over 3-Legged OAuth.

Box integrations mainly utilize two authentication options:
1. JWT OAuth 2.0 - an access pattern where an admin enterprises an application into the enterprise, and the application performs actions through an automation user with application permissions driving what actions are allowed.
1. 3-Legged OAuth 2.0 - an access pattern where a Box enterprise user authorizes an application to perform actions on their behalf, and the permissions are based on the authenticated user in combination with the application permissions.

Historically, a lot of our integrations are built on 3-Legged OAuth as we added support for JWT based authentication at a later time, but JWT provides a number of benefits over 3-Legged OAuth in targeted use cases such as event collection. If you are building a new integration, this section will outline the benefits of either approach for your consideration, but even if you have an existing application with 3-Legged OAuth, it may be worth giving this section a read to see if JWT would be a better fit for your application and consider a switch.

### JWT Based OAuth 2.0
This approach for authentication and token management is more programmatic and can be automated with less potential issues requiring user interaction. This would be the recommended approach for creating enterprise integrations that are scanning the enterprise event stream and monitoring for Box Shield events. It provides the admin of the Box enterprise more direct oversight over what the application is doing, uses a dedicated user specifically for scanning events, and can retrieve tokens on-demand without having to have an end-user manually refresh tokens. Rather than using an explicit grant from one of the users in the enterprise, a JWT based application gets added by the main admin of the enterprise through the admin console, the admin can review the scopes of the application and disable it if needed at any point.

The overall set up of this application looks as follows:
- Developer creates a JWT application, is granted a client id and secret
- Relevant scopes are selected in the application
- A Public Key can be uploaded or Box can generate a public / private key pair for you
- One an enterprise wants to add your product, the main admin just needs to add your client id in the Apps section
- They admin then provides you with their enterprise id, and your application is set to retrieve tokens on-demand
- As tokens are needed, your application creates a JWT payload and sends it to Box, Box will return an access token valid for one hour, but you can make a new request at any point without invaliding the previous access tokens, potentially having multiple access tokens in rotation at the same time if needed.

JWT-based authentication is a much more robust solution for processes that require a constant connection and near-zero downtime for token management. Since tokens can be retrieved by your application on-demand without any end-user interaction, if there is a system hiccup or a token is invalidated, you can have a new one almost immediately to continue your scanning processes. The other benefit of this approach is that during the step when the enterprise admin adds your client id through the Box Admin Console, a dedicated user is created in the enterprise specifically for your application. You can think of that user as a co-admin level automation account which has their access privileges defined by the scopes within your application. If you have the scope for enterprise events checked in your application, you don't have to worry about incorrect permissions at the user level in a Box enterprise.

### 3-Legged OAuth 2.0
This is a common authentication flow across many industries. The gist of this authentication method is that a user is explicitly presented with an access grant dialogue and the scopes outlined, once they grant access the application is sent a security code that is then exchanged for an access token and a refresh token. The application scopes and the user permissions defined by the admin in the Box enterprise work in concert, taking the least privileged approach. For example, a user that is a standard Box user will not be able to access admin-only permissions, such as polling enterprise events, even if the application itself is scoped at that level.

The overall set up of this application looks as follows:
- Developer creates an OAuth application, is granted a client id and secret
- Relevant scopes are selected in the application, a redirect uri is set up
- When a customer needs to authenticate, they are sent to an auth url
- The user is presented with the name of the application and the scopes requested after logging in if no session exists
- Once the application is granted access, a code is sent to the redirect uri
  - This would likely be a developer-owned service that can accept the code and perform the token exchange
  - This code is only valid for around 60 seconds max
- Developer creates a payload using the code, client id, and client secret to exchange for tokens and makes a POST request to Box.
- Box returns an access token valid for approximately an hour and a refresh token valid for one successful refresh of the token in the next 60 days.
  - Once refreshed, a new access token and refresh token is returned
  - A valid token refresh is determined by utilizing the new access token at least once after refresh

Overall, the 3-legged OAuth process is a valid method for generating tokens and starting the scanning process, but we have seen some issues in the past that are mitigated by utilizing the JWT approach. While using this approach gives the end-user that is logging in a view into the scopes that are being requested by the application, it's not necessarily as transparent to the admin. This method is beneficial as it only gives access to the individual user that is authenticating / authorizing the application, and that user's permissions can be controlled from the admin perspective. In many cases enterprises choose to use either the main admin or a co-admin account to be a dedicated account for reading the enterprise event stream into a third party integration.

### Summary
While either one of these authentication methods will work for your application, there are a number of benefits for utilizing JWT over 3-Legged OAuth.
- With JWT permissions are controlled in the application you create and you don't have to worry about a co-admin having incorrect permissions for your application to function.
- JWT generates a dedicated account for your application, so you don't have to share rate limits with other applications that may be running on a co-admin account
- Token management is much simpler with JWT as you can get a new access token whenever needed without having an end-user go through authentication / authorization if something goes wrong
- An admin directly approves a JWT application to have access to their Box enterprise, therefore you don't need to worry about external applications being disabled or blocked by default and having to have your application whitelisted specifically.

Both of these approaches have a "secret" component to the authentication process. In the case of 3-Legged OAuth, the client secret is necessary to exchange an access code for a token pair, while in the case of JWT the private key that is owned by the partner is needed when creating the payload for retrieving tokens. This is worth to take into consideration when creating your application, depending on where these variables may need to be stored. While we don't suggest it, some partners have asked each customer to create their own application in Box, and not have a centralized API key.

## Scopes
Next are the scopes that your application will need for the scanning process.

In general, Box uses a least-privileged approach when it comes to actions that a user may take through the API. We'll take a look at the access level the individual user that is authorizing the application has within their enterprise - including things like content, groups, collaboration, any co-admin privileges - and the scopes that the application is requesting and calculate the most-restrictive set of permissions as to what API calls can be made. For example, if the user is a co-admin, but does not have access to generate enterprise reports but the application does have that scope enabled, the tokens will not be able to be used to pull Box Enterprise Events. Even though the user consents to the application requesting for permission to pull these events, the user did not have the permissions in the first place. While we will not block the user from granting the application access, the actions the application can perform will be limited to what the user already had access to, but not more.

When it comes to scopes, there are three variables we'll need to consider:
1. Application scopes - what is needed at the application level, regardless of authentication model used
1. Scanning Co-Admin Permissions - if using a dedicated co-admin for the purpose of scanning the Enterprise Event Stream, what permissions are needed
1. Incident Response Co-Admin Permissions - if a different co-admin is used for viewing the Shield Alerts in the Box web application, what permissions do they need

### Application Scopes
When setting up an application, whether you choose to utilize 3-Legged OAuth 2.0 or JWT Based OAuth 2.0, you will have the same selection of scopes for your application. Depending on how deep your integration integrates into Box, the scopes that you may need will vary. For the purpose of this guide where we look only into collecting the enterprise event stream and capturing Shield events, you can actually scope your application down to a very specific permission set.

The only required permission for your application to be able to retrieve enterprise events at the application level is called "Manage Enterprise Properties." This will allow your application to query the enterprise event stream if the user that authorized your application has the relevant permissions as well.

Many integrations choose to perform actions beyond just querying the Box Enterprise Event stream and gathering Box Shield events, so you may need to modify these scopes as you see fit. For example, if you want to also be able to retrieve user information or file information through the respective API endpoints, you may need scopes such as "Manage Users" or "Read all files and folders stored in Box," but those are going to be dependent on the desired functionality. Note that you will need to make sure that the authorizing user also has the relevant co-admin permissions if you're looking to expand the scope of the application.

For further reading on application scopes, you can refer to our scope documentation here - [Box Scopes][box-scopes]

### Scanning Co-Admin Permissions
If you choose to use a 3-Legged OAuth 2.0 authentication approach, you will need to have a co-admin or the main admin of the enterprise authorize your application. Note that in Box, there are essentially three types of users
- The main admin of the enterprise who has access to all users, settings and content
- A Co-Admin with elevated permissions defined by the main admin of the enterprise - these can vary from managing users and groups to being able to poll the enterprise event stream or changing enterprise properties
  - More about Co-Admin permissions can be found here - [Co-Admin Permissions][box-coadmin]
- A regular Managed User who only has access to the content and collaborations that they create / are invited to

For this type of integration, you will need either a main admin or a co-admin to authorize your application. As noted in the "Authentication Options" section, we would suggest using a JWT-Based approach, which would automatically create a co-admin automation account for your application with the permissions defined in your application, but if you choose to go the 3-Legged route, you'll need to make sure that the correct permissions are set up for the co-admin as well.

Much like for application scopes, the co-admin account can be granted very targeted permissions for it to be able to query Box Shield events through the Enterprise Event Stream. The only permissions that a co-admin would need is "Run new reports and access existing reports" which would grant them the ability to pull data from that endpoint. As mentioned in the Application Scopes, you may need other permissions if your application wants to perform actions beyond just the event stream.

### Incident Response Co-Admin Permissions
In some cases, users may want to have a co-admin account that is shared across the incident response / security teams to be able to review any Box Shield alerts directly through the dashboard in the Box web application. If this is the case, the co-admin accounts created for this will need to have a specific permission enabled for them. Note that this isn't required for your integration to function, but since Box Shield alerts include a link to the event directly in the Box Shield Dashboard, the user navigating to this link will need to have the relevant access to be able to view it. To avoid confusion, this is worth mentioning in this guide.

The base scope that this co-admin persona would need to have would be "View Shield lists and alerts for your company." This would allow them to follow the link from the Box Shield event that is created and ingested by your application and perform any remediation steps that may be required. As a general piece of advice, these co-admin accounts would likely need elevated permissions beyond just viewing the alerts to be able to do things such as disable accounts, remove content, etc. but these permissions are likely going to be decided on a case-by-case basis by the admin of each enterprise. While this is not directly related to the function of the application, providing this guidance to prospective customers may be beneficial to prevent unnecessary confusion.

### Summary
For your application to function correctly, you will need to make sure that there is a correct set of permissions either from just your application scopes or both user and application combined. Utilizing JWT Authentication will make this process generally much smoother with less potential variables of what can go wrong, but either authentication option will work, and will only require a one-time set up before your application can function indefinitely.

## Box Shield Events
Enterprises that have purchased the Box Shield add-on are able to configure their own access policies and notifications. As such, there is not going to be a default set of alerts that you will see across all enterprises utilizing this, but to simplify this from an integration perspective, we have grouped all different alerts under a single "event_type" in our Enterprise Event Stream. There is a large set of different events that are reported on within the enterprise event stream that can be viewed in our [Box Events][box-events]

Note that when pulling events from this stream, you are also able to filter to just specific event types, so your integration can decide (or let the user decide if you so choose) what event types they are interested in capturing and logging. The specific "event_type" for all alerts that are generated by Box Shield is "SHIELD_ALERT" and within that alert, you will find an "additional_details" section that will narrow it down to the specific Box Shield alert type, as well as provide information relevant to that specific event.

There are four different types of alerts that are currently reported on, though more events are planned to be added:
- Suspicious Location
- Suspicious Session
- Anomalous Download
- Malicious Content
Note that when setting up these Box Shield alerts, for your integration to be able to capture this information, the admin needs to have checked the box to "Publish alerts to Box Event Stream" when creating the Threat Detection policies. A breakdown of the different Threat Policies and admin-facing set up of these rules can be found in our [Threat Detection][box-threat]

All Box Shield alerts when they are delivered to the Enterprise Event stream will contain the same set of basic information about the event, but majority of the specific information will be found under the "additional_details" subsection of the event. Items such as the user that caused this event to be created, the ip address, and when the event was created may be found in the general section of the event. We have a general overview of the different types of events and the information that they deliver in our developer documentation. The rest of this guide will go into much more specific detail into the specific events and their payloads, so if you find the information on our developer documentation sufficient, you may not need to refer to the rest of this unless questions come up. The basic overview of SHIELD_ALERT events can be found in our [Box Shield Alert Events][box-shield-alerts]

[box-shield]: https://www.box.com/shield
[box-scopes]: https://developer.box.com/guides/api-calls/permissions-and-errors/scopes/
[box-coadmin]: https://support.box.com/hc/en-us/articles/360044194393-Granting-And-Modifying-Co-Admin-Permissions
[box-events]: https://developer.box.com/guides/events/for-enterprise/#event-types 
[box-threat]: https://support.box.com/hc/en-us/articles/360044196113-Using-Threat-Detection
[box-shield-alerts]: https://developer.box.com/guides/events/shield-alert-events/


