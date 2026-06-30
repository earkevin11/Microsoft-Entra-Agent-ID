# Understanding Microsoft-Entra-Agent-ID

Summary
Microsoft Entra Agent ID is a new feature that that will allow administrators to associate identities with AI Agents. These Agent ID’s can be used to build AI agents, so that AI agents can:

authenticate and gain access to resources, like reading a file.
sign-in users, perhaps in order to exchange messages with those users.
assist the user by taking action on their behalf, like sending an email.
receive incoming requests from other agents and clients, and authenticate the caller.
By creating Agent IDs in Entra, administrators, can manage their AI agents separately from their user accounts, service principals, and devices. Allowing them to apply separate controls for agents versus users.

What is Agent ID
Agent identity: The primary account used by an AI agent to authenticate to various systems. Has unique identifiers - the object ID and the app ID (which always have the same value) - which can be reliably used for authentication and authorization decisions. Agent identities can be used to:

Request agent tokens from Entra. The subject of the access token will be the agent identity.
Receive incoming access tokens issued by Entra. The audience of the access token will be the agent identity.
Request user tokens from Entra for an authenticated user. The subject of the token will be a user, while the actor (azp or appId claim) will be the agent identity.
Agent identities do not have a password or any other kind of credential. Instead, agent identities can only authenticate by presenting an access token issued to the service / platform on which the agent runs (see below). Agent identities can only be issued tokens in the Entra tenant where they are created. They cannot access resources or APIs in other tenants.

Concepts
Agent Identity Blueprints: The "parent" of an agent identity, which simultaneously serves three purposes:

The template establishes the "type" or "kind" of agent identity, such as "MyApp Sales Agent" or "MyApp Monitoring Agent". This allows administrators to manage many individual AI agents of a common type as a collection. The blueprint also records attributes, metadata, and settings that are common across all of its agent IDs, such as role definitions.

The service that creates agent identities in tenants typically uses the agent templates to authenticate. Blueprints have an OAuth client ID and credentials that can include: client secrets, certificates, and various federated identity credentials such as managed identities. Services use these credentials to request access tokens from Entra, then use those access tokens to authenticate requests to create / update / delete agent identities. In this sense, the template is the creator of an agent identity.

The service / platform that hosts an AI agent must use the Blueprint during runtime authentication. The service uses a template's OAuth credentials to request an access token, then presents that access token to Entra to request a token as one of its agent identities.

Agent ID User: A secondary account that can be used by an AI agent to authenticate to various systems. These accounts are user objects in a tenant and have most properties of other users, like a manager, UPN, and photo. This makes them compatible with systems that have a hard dependency on user objects, and enable AI agents to connect to these systems. Agent ID users:

Are also typically created by an agent identity template.
Are always associated to a specific agent identity, specified upon creation.
Have distinct unique identifiers, separate from the agent identity.
Can only authenticate by presenting a token issued to the associated agent identity
Agent ID User objects should only be used when an agent needs to connect to certain systems / resources. Examples of such systems include Exchange mailboxes, M365 Groups / Teams groups, and Intune RBAC. Creating an Agent ID User object requires additional authorization be granted to the template or service that creates these identities. This object would typically be associated with Instantiated agents.

Where agents live
Across Microsoft 365 (A365) and Entra, agents are distributed among three main registries.

M365 Onboarding Service (MOS) is responsible for older third-party agents and Declarative Agents (DA).
A365 Onboarding Service (AOS) manages newer first- and third-party agents that utilize Agent Identities and support comprehensive instantiated agent scenarios.
Entra Agent Registry (EAR) contains all agents with Agent Identities and includes agents that either self-register with EAR or are detected through tenant scanning for unsecured agents.
Types of Agents
Interactive Agents:

Takes action based on user interaction / prompts
Act on behalf of the user.
Uses the user’s authorization to perform the requested actions.
Use User Tokens to perform actions
Autonomous Agents:

Agents take action on their own.
Perform actions using their own authorization. This does not require any human authorization.
Access is granted to the agent and does not rely on user permissions to perform a task.
Tokens issued to autonomous agents are often called agent tokens when an agent identity is authenticated, or agent user tokens when an Agent ID user object is authenticated
Instantiated Agent: (Names are still in flux)

Provisioned its own access and resources; achieves goals on own behalf and schedule
Learning-driven, Emulates human decision making
Agent IDs permissions
Customer may observe that permissions for Agent IDs do not appear in the Entra admin center under API permissions or consent views. This behavior is expected and by design. Agent IDs use Microsoft-managed permissions that are preauthorized by Microsoft (explained here  and here ).

These permissions are not tenant-configurable and do not appear in the Entra portal like app registrations.

The best way to understand what permissions the identities for this particular agent have been granted is through the official public documentation: Microsoft Entra Agents 

Agent ID Portal
Overview Page Admins can view metrics for the following:

Recently Created Agents: Agents created in the last 30 days
Unmanaged Agents: Agents with no owners or sponsors. For information on agent owners and sponsors see: Administrative relationships in Microsoft Entra Agent ID (Owners, sponsors, and managers) 
Active Agents: Agents enabled for access
No Identities: Agent ID's are an optional field. Third party agents created through OpenID for example may not have an Agent Identity associated with it. This tile will show the number of agents that exist in a tenant that do not have an Agent ID. See: Manage agents with no agent identities 
Types of Agents: This tile will show the number of Agents Identities (with no user), Agent Identities (with user), Agents with service principles, and Agents with no Identities.
Agent Blueprints: This is where admins will able to see Agent Blueprints that have been published to their tenant. Clicking the View All button will launch the Agent Blueprints blade, where admins can see the details of each blueprint. Note: The Agent Blueprints Blade can also be launched from the All Agent Identities page.
Agent ID Overview

Agent Blueprint Blade

Agent Blueprints Details: From the Agent Blueprints Blade, admins can view the details of each agent blueprint. This information includes the following:
Create Date
Number of linked Agent Identities
The Blueprint IDT
The blueprint Object ID
Blueprint's Access: Permissions that have been granted to a blueprint when it was published. This includes both Admin Consent and User Consent
Owners and Sponsors: A sponsor is required for all Blueprints. If a blueprint is missing a sponsor the administrator can add one here.
Audit Logs: To view Audit logs for an agent, Administrators can click the Audit Logs tab and click View Audit Logs to be redirected to the specific audit logs for the agent.
Sign-in Logs: Administrators, can access the Sign-in events page for the specific blueprint from this tab.
Enable or Disable and Agent Blueprint: Admins can disable an agent blueprint from the Agent Blueprint Details.
Agent Blueprint Details

All Agent Identities

Admins will be able to see all Agent Identities in their tenant. From here they can enable or disable these identities as well as launch the Agent Blueprints blade.

Agent Registry / Agent Collections - These pages will be changing, and are covered in Microsoft Entra Agent ID Agent Registry

Known Issues
Not able to delete Service Principal - Unable to complete the request due to data validation error
Issue

Global Admin is trying to delete SP using portal and the deleting fails with the data validation error.

Error1

Looking at the SP properties we can see "ServicePrincipalType:ServiceIdentity" and there is no Managed Identity associated with this SP (can be validated in ASC).

In the browser trace we can see the error - "Graph call failed with httpCode=BadRequest, errorCode=Request_BadRequest, errorMessage=Agent Identities are not supported on the API version used in this request., reason=Bad Request, correlationId ="

Root cause

Currently management of Agent IDs is supported only via Graph beta endpoint and some portions of the Portal UI are not making correct calls.

For example for the error above the portal is making this call - DELETE https://main.iam.ad.ext.azure.com/api/ManagedApplications/.....isOnPremApp=false

Workaround

Use Graph calls to beta endpoint to manage Agent ID Service Principal - Create Agent Identities in Microsoft Agent Identity Platform 

More info

Case example - 2604160050000143 (the IcM supposed to be opened to notify PG about the issue)

Case Routing
Note, some features of Agent ID are being moved to other admin portals. The support boundaries will change once this update has occurred.

Issues with Agent ID (Preview) Overview Page: Cases for issues with the Agent ID Overview Page, should be routed to the following SAP: Microsoft Entra Agent ID (Preview) / All Agent Identities.

Issues with Agents not showing on the All Agent Identities (Preview) page: should be routed to the following SAP: Microsoft Entra Agent ID (Preview) / All Agent Identities.

Issues with missing agents: Cases for issues with agents missing from the Agent Identity interfaces: should be routed to Microsoft Entra Agent ID (Preview) / Agent Without Identities

Issues with creating Blueprints and Instantiation agents with Graph API and PowerShell: TBD.

Escalations
ICM Path:

App, Blueprints Service Principals, and Agent Identities:


 Owning Service: AAD Applications

 Owning Team: Triage
Auth issues, Token issuance issues:


 Owning Service: ESTS

 Owning Team: ESTS
User Object Issues:


 Owning Service: IAM Services

 Owning Team: Users and Tenants
Training
Deep Dive: 309569 Microsoft Entra Agent ID

Format: Self-paced eLearning

Duration: 78 minutes

Audience: Cloud Identity Support Team, Account Management, App Experience, M365 Identity

Region: All regions

Course Location: Cloud Academy 

Note: To access the course, click Login with your company account and type Microsoft in the Company Subdomain field, then click Continue.
