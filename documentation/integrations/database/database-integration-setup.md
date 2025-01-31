# Database Integration Setup

{% hint style="success" %}
**This Integration supports multiple instances**

[Check out the instructions to set up multiple instances here](../general/multi-instance-integration/multi-instance-integration-setup.md).
{% endhint %}

### Overview

This document outlines the requirements and setup for the **Bring Your Own Database (BYOD)** integration setup.

### Requirements (pre-Rewst)

To ensure the BYOD integration works seamlessly, there are a few requirements that need configuring on your end:

1. **Database Setup via Powershell:** Initially, you will be required to run the database setup Powershell script. Ensure that you're running this database setup script with Powershell 7.
2. **Password and Script Output:** During the setup, you'll be prompted to create a password for the DB account designated for the integration. Make sure to safely store this password. Additionally, retaining all of the output from the script will be crucial for subsequent steps, so ensure it's accessible.

<details>

<summary>Bring Your Own Database Script</summary>

{% code overflow="wrap" %}
```powershell

### Script created by Adam Willford of the Rewst ROC ###

### For any assistance, please contact the ROC team on roc@rewst.io, or via the-kewp in Slack or Discord ###

### Require PS7 - mostly because otherwise the logo doesn't look good...

#Requires -Version 7.0

### Check if the module required is installed and if not, install it ###

if (!(Get-Module -ListAvailable Az.SQL)) {
    Install-Module Az.SQL -Confirm:$false -Force
}


if (!(Get-Module -ListAvailable Az.Resources)) {
    Install-Module Az.Resources -Confirm:$false -Force
}

Import-Module Az.Sql
Import-Module Az.Resources



### Create a cool logo, of course ###

$Logo = @'
██████  ███████ ██        ██ ███████ ████████ 
██  ██  ██    ██       ██ ██        ██    
██████  █████   ██   █  ██  ███████    ██    
██  ██ ██     ██ ███ ██     ██    ██    
██   ██ ███████  ███  ███  ███████     ██    
                                           
'@

Write-Output $Logo
Write-Host 'Rewst ROC - Connecting to Azure' -ForegroundColor Cyan
Write-Host 'Requesting Information from end user' -ForegroundColor Cyan

### All the variables required for the script.  Note these are all prompt based, so no need to amend the script ###

$userEnteredSubscriptionId = $(Write-Host "Please enter your subscription ID:" -ForegroundColor Green -NoNewLine; Read-Host)
$userEnteredCompanyName = $(Write-Host "Please enter your Company Name:" -ForegroundColor Green -NoNewLine; Read-Host)
$userEnteredResourceGroupName = $(Write-Host "Enter a unique new Resource Group name:" -ForegroundColor Green -NoNewLine; Read-Host)
$userEnteredLocation = $(Write-Host "Enter the datacenter location to store the database.  Note for 'US East' you would enter 'eastus'. Locations can be seen here: https://learn.microsoft.com/en-us/azure/availability-zones/az-overview:" -ForegroundColor Green -NoNewLine; Read-Host)
# $userEnteredServerName = $(Write-Host "Please enter the SQL Server Name required (a-z, 0-9 and '-' only):" -ForegroundColor Green -NoNewLine; Read-Host)
$userEnteredAdminUsername = $(Write-Host "Please enter the SQL Administrator Username:" -ForegroundColor Green -NoNewLine; Read-Host)
$userEnteredAdminPassword = $(Write-Host "Please enter the SQL Administrator password. (This is the last one, promise):" -ForegroundColor Green -NoNewLine; Read-Host -AsSecureString)
[pscredential]$credObject = New-Object System.Management.Automation.PSCredential ($userEnteredAdminUsername, $userEnteredAdminPassword)
### Connect to the Azure instance based on the provided information ###

Connect-AzAccount -Subscription $userEnteredSubscriptionId | Out-Null

### Create the Resource Group ###

try {
    Write-Host "Creating resource group..." -ForegroundColor Cyan
    New-AzResourceGroup -Name $userEnteredResourceGroupName -Location $userEnteredLocation -Tag @{Owner = "Rewst-ROC" } | Out-Null
    Start-Sleep 15
    Write-Host "Successfully created resource group $userEnteredResourceGroupName" -ForegroundColor Blue
} catch {
        Write-Error "There was an error creating the resource group.  The error, if supplied, was:  $($_.Exception.Message)"
        exit
}

### If the resource group exists, crack on ###
if (Get-AzResourceGroup -Name $userEnteredResourceGroupName) {
    ### Create the SQL Server ###
    try {
        $SQLServerCreationInformation = @{
            ResourceGroupName           = $userEnteredResourceGroupName
            ServerName                  = "$($userEnteredCompanyName.ToLower().trim())-database" -replace '[^a-zA-Z0-9-]',''
            Location                    = $userEnteredLocation
            SqlAdministratorCredentials = $credObject
        }
        Write-Host "Creating primary server...(Note this may take several minutes, because Microsoft)" -ForegroundColor Cyan
        New-AzSqlServer @SQLServerCreationInformation | Out-Null
        Write-Host "Successfully created SQL Server - $($SQLServerCreationInformation.ServerName)" -ForegroundColor Blue
    } catch {
        Write-Error "There was an error creating the SQL Server $(($SQLServerCreationInformation.ServerName))  The error, if supplied, was:  $($_.Exception.Message)"
        exit
    }
        
    ### Create a firewall rule to only allow connectons in from Rewst ###

    try {
        $SQLServerFirewallRule = @{
            ResourceGroupName = $userEnteredResourceGroupName
            ServerName        = $SQLServerCreationInformation.ServerName
            FirewallRuleName  = 'Rewst - Allowed IPs'
            StartIpAddress    = '3.139.170.31'
            EndIpAddress      = '3.139.170.31'
        }
        Write-host "Configuring server firewall rule..." -ForegroundColor Cyan
        New-AzSqlServerFirewallRule @SQLServerFirewallRule | Out-Null
        Write-Host "Successfully created Firewall Rule - $($SQLServerFirewallRule.ServerName)" -ForegroundColor Blue
    } catch {
        Write-Error "There was an error creating the firewall rule $(($SQLServerFirewallRule.FirewallRuleName)).  The error, if supplied, was:  $($_.Exception.Message)"
        exit
    }

    ### Create the actual database on the newly created server ###

    try {
        $SQLServerDatabaseInformation = @{
            ResourceGroupName  = $userEnteredResourceGroupName
            ServerName         = $SQLServerCreationInformation.ServerName
            DatabaseName       = 'Rewst-Database'
            Edition            = 'GeneralPurpose'
            ComputeModel      = 'Serverless'
            ComputeGeneration = 'Gen5'
            vCore              = 2
            MinimumCapacity    = 2
            AutoPauseDelayInMinutes = 60
        }
        Write-host "Creating a gen5 2 vCore serverless database..." -ForegroundColor Cyan
        New-AzSqlDatabase @SQLServerDatabaseInformation | Out-Null
        Write-Host "Successfully created Database - $($SQLServerDatabaseInformation.ServerName)" -ForegroundColor Blue
    } catch {
        Write-Error "There was an error creating the database $($SQLServerDatabaseInformation.DatabaseName) on $($SQLServerDatabaseInformation.ServerName).  The error, if supplied, was:  $($_.Exception.Message)"
        exit

    }
} else {
    Write-Error "The Resource Group - $($userEnteredResourceGroupName) was not found and therefore we cannot continue."
    exit
}

### Output User Information ###

Write-Host "Congratulations, you can now go and configure the database in the Rewst integration itself.  The following details will be required:" -ForegroundColor Cyan
Write-Host '--------------------------------------------------------' -ForegroundColor White
Write-Host "Database Config Name: Rewst Cache - Database" -ForegroundColor Green
Write-Host "Database Type: MSSQL" -ForegroundColor Green
Write-Host "Hostname: $($SQLServerDatabaseInformation.ServerName).database.windows.net" -ForegroundColor Green
Write-Host "Port: 1433" -ForegroundColor Green
Write-Host "Username: $userEnteredAdminUsername" -ForegroundColor Green
Write-Host "Password: [Entered During Prompt Phase, we don't want to show it here for obvious reasons]" -ForegroundColor Green
Write-Host "Database Name: $($SQLServerDatabaseInformation.DatabaseName)" -ForegroundColor Green
Write-Host '--------------------------------------------------------' -ForegroundColor White

```
{% endcode %}

</details>

### Integration

Once you've set up the database and have the necessary credentials, follow the below steps to configure a new integration:

1. **Log in** to the [Rewst platform](https://app.rewst.io).
2. **Click on** the _"Integrations"_ menu on the left sidebar.
3. **Click** or search for "SQL Database".
4. **Complete** the form with the details you created.
5. **Click** Save.

{% hint style="warning" %}
If you're a DattoRMM customer looking to use a BYOD integration as a workaround to improve the speed of data returned by DattoRMM's APIs, please refer to the [BYOD for DattoRMM](../rmm/datto-rmm/byod-for-dattormm.md) documentation.
{% endhint %}

