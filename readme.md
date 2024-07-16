## Overview

This solution provides SharePoint Online Administrators with an evergreen SharePoint list detailing all Entra ID App Registrations in a tenant which have with one or more SharePoint Online related site consented permissions.  The solution is packaged as an unmanaged Power Platform Solution package.  It contains four Power Automate Workflows use to inventory Entra ID App Registrations, import SharePoint Online sites (sites.selected) associations and import app registration sign-in activity.

These workflows are highly optimized and have been tested on multiple tenants (inculding multi-geo) with 100k+ site collections and 15k+ (total) App Registrations. 

### Dashboard Details

#### App Registration Details
Once the flows are running and data is being imported, users with access to the target SharePoint list will be able to view the important details of all app registrations in the tenant with at least one SharePoint Online related permission.  The list contains basic details about the app registration; title, object id, client id, name, created date.  For app registrations that have been associated with a SharePoint Online site using the Sites.Selected permission, the *SharePoint Site* column will contain all sites assocated with the app registration.   
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-1.png" width="800"></kbd>
</p>

#### Permission Details
The list will all consented SharePoint Online related permissions for both the Delegated and Application scope types for the Microsoft Graph Delegated and SharePoint Online APIs. Additionally, the *Last Sign-In Activity Date*, *Secret Expiration Date*, *Certificate Expiration Date* and the *Internal Notes* fields are also provided. 
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/list-2.png" width="800"></kbd>
</p>

### Power Automate Flow Details

Each Power Automate Flow is documented here:
- [Provision SharePoint List Schema Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ProvisionSharePointListSchemaFlowDetails.md)
- [Import Site Collection Associations Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ProvisionSharePointListSchemaFlowDetails.md)
- [Import Application Registrations Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ImportApplicationRegistrationsFlowDetails.md)
- [Import Application Registrations Last Sign-in Activity Flow Detail](https://github.com/joerodgers/sharepoint-app-registrations/blob/main/ImportApplicationRegistrationsSignInFlowDetails.md)

## Setup

#### Licening
All of the Power Automate Flows in this package leverage one or more premium connectors, therefore the owner of the solution package must have a Power Automate Premium license assigned to their user account. 

### Entra Id Application Registration
This solution package relies entirely on a single Entra ID application registration for authentication to the Microsoft Graph.  All authentication is performed using an app-only context with a Client Id and Secret. To operate in a least privileged configuration, the associated app registration requires the following permissions:

| Application Permission | Justification |
|------------------------|---------------|
|Application.Read.All    | Required to read all applications from Entra ID using Microsoft Graph’s applications endpoint. |
|Sites.Read.All          | Required to read all site collections from Microsoft Graph’s GetAllSites endpoint. |
|Sites.Selected          | “Manage” permissions to the “host” site collection to allow list provisioning, list item operations. |

Example reference configuration:
<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/entra-perms.png" width="800"></kbd>
</p>

### Site Collection Permissions
To grant the "Manage" permission to the app registration to the “host” SharePoint site collection, execute the following PnP code, substituting the necessary variable values:
``` PowerShell
#requires -modules "PnP.PowerShell"

$tenant   = "contoso"
$clientId = "00000000-0000-0000-0000-000000000000" 
$siteUrl  = "https://contoso.sharepoint.com/sites/sitename"


Connect-PnPOnline `
    -Url "$tenant-admin.sharepoint.com" `
    -Interactive `
    -ForceAuthentication


$r = Grant-PnPAzureADAppSitePermission `
            -DisplayName "SharePoint App Registrations Dashboard" `
            -AppId $clientId `
            -Site $siteUrl `
            -Permissions "Write"


Set-PnPAzureADAppSitePermission `
            -PermissionId $r.Id `
            -Site $siteUrl `
            -Permissions "Manage"

```
 
### Solution Import

#### Power Platform Environment Creation (optional)
Optionally, create a new Power Platform Environment using the following article as reference:
[Create and manage environments in the Power Platform admin center](https://learn.microsoft.com/en-us/power-platform/admin/create-environment)

#### Environment Solution Import
In the target environment, import the solution package using the following article as reference:
[Import Solution](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/import-update-export-solutions)
 
### Solution Environment Variable Configuration
After the solution has been successfully imported, select the Application Principal Reporting solution.  

<p align="center" width="100%">
    <kbd><img src="https://github.com/joerodgers/sharepoint-app-registrations/blob/main/assets/solution-overview.png" width="800"></kbd>
</p>

Configure the following environment variables:

| Environment&nbsp;Variable      | Description |
|---------------------------|-------------|
| Tenant Id                 | Microsoft 365 Tenant Id |
| Client Id                 | Application Registration Client/App Id |
| Secret&nbsp;(Azure&nbsp;Key&nbsp;Vault)  | All flows check for the existence of the AKV secret value first. If the AKV secret is not configured or fails,the flows fall back to the reading the Secret (Plaintext) environment variable. Do not configure this variable if you plan to only use the Secret (Plaintext) environment variable for secret storage. |
| Secret (Plaintext)        | This value is a fallback value if the Secret (Azure Key Vault) variable is not configured. |
| Site URL                  | URL of the SharePoint site hosting the list/dashboard |
| List Name                 | Default value is "SharePoint Application Principals" "

## Power Automate Flow Details


