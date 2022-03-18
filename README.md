# Notify-ASCAlertsAzureResource
author: Giulio Astori

This Logic App is used to disable AAD account and revoke all bearer token.<br>
Note: This playbook will not be able to disable users if they are eligible or have active admin roles.<br>
If user have manager, manager will be notified that the user have been disabled in Azure AD. otherwise the notification is sent to the email confugured during the provisioning.<br>


<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FGastori%2FLogicApps%2Fmain%2Fmdc-disable-account-revoke-token%2Fazuredeploy.json" target="_blank">
    <img src="https://aka.ms/deploytoazurebutton"/>
</a>
<a href="https://portal.azure.us/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FGastori%2FLogicApps%2Fmain%2Fmdc-disable-account-revoke-token%2Fazuredeploy.gov.json" target="_blank">
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazuregov.png"/>
</a>

**Additional Post Install Notes:**

The Logic App creates and uses a Managed System Identity (MSI) to authenticate and authorize against management.azure.com (or management.usgovcloudapi.net if in Azure Gov) to obtain PrincipalIDs assigned to the Azure Resource. The MSI is also used to authenticate and authorize against graph.windows.net to obtains RBAC Objects by PrincipalIDs. 

1. Assign API permissions to the managed identity so that we can search for user's manager. You can find the managed identity object ID on the Identity blade under Settings for the Logic App. If you don't have Azure AD PowerShell module, you will have to install it and connect to Azure AD PowerShell module. https://docs.microsoft.com/powershell/azure/active-directory/install-adv2?view=azureadps-2.0
```powershell
$MIGuid = "<Enter your managed identity guid here>"
$MI = Get-AzureADServicePrincipal -ObjectId $MIGuid

$GraphAppId = "00000003-0000-0000-c000-000000000000"
$PermissionName1 = "User.Read.All"
$PermissionName2 = "User.ReadWrite.All"
$PermissionName3 = "Directory.Read.All"
$PermissionName4 = "Directory.ReadWrite.All"

$GraphServicePrincipal = Get-AzureADServicePrincipal -Filter "appId eq '$GraphAppId'"
$AppRole1 = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName1 -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole1.Id

$AppRole2 = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName2 -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole2.Id

$AppRole3 = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName3 -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole3.Id

$AppRole4 = $GraphServicePrincipal.AppRoles | Where-Object {$_.Value -eq $PermissionName4 -and $_.AllowedMemberTypes -contains "Application"}
New-AzureAdServiceAppRoleAssignment -ObjectId $MI.ObjectId -PrincipalId $MI.ObjectId `
-ResourceId $GraphServicePrincipal.ObjectId -Id $AppRole4.Id

2. Open the playbook in the Logic App Designer and authorize Azure AD and Office 365 Outlook Logic App connections<br><br>
