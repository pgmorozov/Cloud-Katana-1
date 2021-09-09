# Demos

## Requirements

### Install Microsoft Authentication Libraries (MSAL)

Locally, open PowerShell as Administrator and run the following commands to install and import the PowerShell MSAL.PS moule:

```PowerShell
Install-PackageProvider NuGet -Force
Install-Module PowerShellGet -Force -AllowClobber

Install-Module -name MSAL.PS -Force -AcceptLicense
Import-Module MSAL.PS
```

### Set Variables

```PowerShell
$functionAppName = 'FUNCTION-APP-NAME'
$azureFunctionUrl = "https://$functionAppName.azurewebsites.net"
$OrchestratorUrl = "$azureFunctionUrl/api/orchestrators/Orchestrator"
```

### Get Function App Access Token

```PowerShell
$cloudkatanaClientAPPId = 'xxxx'
$tenantId = 'xxxx'

$results = Get-FuncAppToken -AppId $cloudkatanaClientAPPId -FunctionAppUrl $azureFunctionUrl -TenantId $tenantId -verbose
$accessToken = $results.AccessToken
```

The first time you use Cloud Katana, you will have to accept the permissions requested to access the Azure AD application exposing Cloud Katana APIs and enabling authentication and authorization via Azure AD. Click `Accept` and you will get an access token back to use while interacting with Cloud Katana's serverless API:

![](../../images/CloudKatanaClientRequest.png)

### Set Authorization Header

```PowerShell
$headers = @{
  Authorization = "Bearer $accessToken"
}
```

## Examples

### Get All Azure AD Users

```PowerShell
# HTTP Body
$body = @(
  @{
    Platform = 'Azure'
    Tactic = 'discovery'
    Procedure = 'Get-CKAzADUsers'
  }
) | ConvertTo-Json -Depth 4

# Execute Simulation
$simulationResults = Invoke-RestMethod -Method Post -Uri $OrchestratorUrl -Body $body -Headers $headers -ContentType 'application/json'
$simulationResults

# Sleep
Start-Sleep -s 5

# Explore Results
$outputResults = (Invoke-RestMethod -Uri $simulationResults.statusQueryGetUri -Headers $headers).output
$outputResults | Format-List
$outputResults | Where-Object {$_.userPrincipalName -like '*simulandlabs*'} | Select-Object userPrincipalName
```

### Get All Azure AD Applications

```PowerShell
# HTTP Body
$body = @(
  @{
    Platform = 'Azure'
    Tactic = 'discovery'
    Procedure = 'Get-CKAzADApplication'
  }
) | ConvertTo-Json -Depth 4

# Execute Simulation
$simulationResults = Invoke-RestMethod -Method Post -Uri $OrchestratorUrl -Body $body -Headers $headers -ContentType 'application/json'
$simulationResults

# Sleep
Start-Sleep -s 5

# Process Results
$outputResults = (Invoke-RestMethod -Uri $simulationResults.statusQueryGetUri -Headers $headers).output
$outputResults | Select-Object displayName
```

### Update Application Credentials

```PowerShell
# HTTP Body
$body = @(
  @{
    Platform = 'Azure'
    Tactic = 'persistence'
    Procedure = 'Add-CKAzADAppPassword'
    Parameters = @{
        appObjectId = 'AZURE-AD-APP-OBJECT-ID'
        displayName = 'BlackHatSecret'
    }
  }
) | ConvertTo-Json -Depth 4

# Execute Simulation
$simulationResults = Invoke-RestMethod -Method Post -Uri $OrchestratorUrl -Body $body -Headers $headers -ContentType 'application/json'
$simulationResults

# Sleep
Start-Sleep -s 5

# Process Results
$outputResults = (Invoke-RestMethod -Uri $simulationResults.statusQueryGetUri -Headers $headers).output
$outputResults | Format-list
```

### Get User's Mailbox

```PowerShell
# HTTP Body
$body = @(
  @{
    Platform = 'Azure'
    Tactic = 'collection'
    Procedure = 'Get-CKMailboxMessages'
    Parameters = @{
      userPrincipalName = 'USER-NAME@DOMAIN.com'
    }
  }
) | ConvertTo-Json -Depth 10

# Execute Simulation
$simulationResults = Invoke-RestMethod -Method Post -Uri $OrchestratorUrl -Body $body -Headers $headers -ContentType 'application/json'
$simulationResults

# Sleep
Start-Sleep -s 5

# Process Results
$outputResults = (Invoke-RestMethod -Uri $simulationResults.statusQueryGetUri -Headers $headers).output
$outputResults | Select-Object bodyPreview
```