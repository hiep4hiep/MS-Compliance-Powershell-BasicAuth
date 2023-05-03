# MS-Compliance-Powershell-BasicAuth


## Open Powershell console and start typing these command one by one:

### Install ExchangeOnlineManagement module
```Install-Module -Name ExchangeOnlineManagement```

### Use ExchangeOnlineManagement module
```Import-Module ExchangeOnlineManagement```

### Connect to Sec & Compliance Powershell
Continue typing these commands to Powershell. Make sure you change the value of `<your-password>, <your-username>, <your-tenant-id>` to your specific tenant information.
  
```
$password = "<your-password>"
$username = "<your-username>"
$tid = "<your-tenant-id>"

$secPassword = ConvertTo-SecureString "$password" -AsPlainText -Force
$delegated_cred = New-Object System.Management.Automation.PSCredential ("$username", $secPassword)
$connection_uri = "https://ps.compliance.protection.outlook.com/powershell-liveid/"
$azure_ad_authorization_endpoint_uri = "https://login.microsoftonline.com/$tid"
Connect-IPPSSession -Credential $delegated_cred -WarningAction:SilentlyContinue -ConnectionUri $connection_uri -AzureADAuthorizationEndpointUri $azure_ad_authorization_endpoint_uri | Out-Null
```

### Verify whether you can run a command
```Get-ComplianceSearch```

Example:
```
Get-ComplianceSearch

Name              RunBy JobEndTime           Status
----              ----- ----------           ------
example1          User1 6/12/2022 1:14:00 am Completed
test02                                       NotStarted
test002                                      NotStarted
suspicious emails                            NotStarted
```


### Run a Compliance Search
```
$search_name = "<search-name-here>"
$Search=New-ComplianceSearch -Name $search_name -ExchangeLocation All -ContentMatchQuery 'from:<from-address> AND subject:"<subject-keywords>"' 
Start-ComplianceSearch -Identity $Search.Identity
```

### Verify the Compliance Search has run completely
```
Get-ComplianceSearch -Identity $search_name
```

### Trigger soft delete
```
New-ComplianceSearchAction -SearchName $search_name -Purge -PurgeType SoftDelete -Confirm:$false
```

### Disconnect the session
```
Disconnect-ExchangeOnline -Confirm:$false -WarningAction:SilentlyContinue
```




# Appendix: install Powershell on RHEL 8

### Register the Microsoft RedHat repository
`curl https://packages.microsoft.com/config/rhel/8/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo`

### Install PowerShell
`sudo dnf install --assumeyes powershell`

### Install PSWSMan module
- Get into pwsh
```
Install-Module -Name PSWSMan
Install-WSMan
```

# Appendix: Script to run via WinRM
Triggering the script via WinRM will not keep the session, hence we need to start the session and disconnect session everytime we trigger a command.
### Step 1: Start a compliance search
```
$password = "<your-password>"
$username = "<your-username>"
$tid = "<your-tenant-id>"

$secPassword = ConvertTo-SecureString "$password" -AsPlainText -Force
$delegated_cred = New-Object System.Management.Automation.PSCredential ("$username", $secPassword)
$connection_uri = "https://ps.compliance.protection.outlook.com/powershell-liveid/"
$azure_ad_authorization_endpoint_uri = "https://login.microsoftonline.com/$tid"
Connect-IPPSSession -Credential $delegated_cred -WarningAction:SilentlyContinue -ConnectionUri $connection_uri -AzureADAuthorizationEndpointUri $azure_ad_authorization_endpoint_uri | Out-Null
$search_name = "<search-name-here>"
$Search=New-ComplianceSearch -Name $search_name -ExchangeLocation All -ContentMatchQuery 'from:<from-address> AND subject:"<subject-keywords>"' 
Start-ComplianceSearch -Identity $Search.Identity
Disconnect-ExchangeOnline -Confirm:$false -WarningAction:SilentlyContinue

```

### Step 2: Check if the compliance search has finished
```
$password = "<your-password>"
$username = "<your-username>"
$tid = "<your-tenant-id>"

$secPassword = ConvertTo-SecureString "$password" -AsPlainText -Force
$delegated_cred = New-Object System.Management.Automation.PSCredential ("$username", $secPassword)
$connection_uri = "https://ps.compliance.protection.outlook.com/powershell-liveid/"
$azure_ad_authorization_endpoint_uri = "https://login.microsoftonline.com/$tid"
Connect-IPPSSession -Credential $delegated_cred -WarningAction:SilentlyContinue -ConnectionUri $connection_uri -AzureADAuthorizationEndpointUri $azure_ad_authorization_endpoint_uri | Out-Null
$search_name = "<search-name-here>"
Get-ComplianceSearch -Identity $search_name
Disconnect-ExchangeOnline -Confirm:$false -WarningAction:SilentlyContinue
```

### Step 3: Start the soft delete action on the compliance search
```
$password = "<your-password>"
$username = "<your-username>"
$tid = "<your-tenant-id>"

$secPassword = ConvertTo-SecureString "$password" -AsPlainText -Force
$delegated_cred = New-Object System.Management.Automation.PSCredential ("$username", $secPassword)
$connection_uri = "https://ps.compliance.protection.outlook.com/powershell-liveid/"
$azure_ad_authorization_endpoint_uri = "https://login.microsoftonline.com/$tid"
Connect-IPPSSession -Credential $delegated_cred -WarningAction:SilentlyContinue -ConnectionUri $connection_uri -AzureADAuthorizationEndpointUri $azure_ad_authorization_endpoint_uri | Out-Null
$search_name = "<search-name-here>"
New-ComplianceSearchAction -SearchName $search_name -Purge -PurgeType SoftDelete -Confirm:$false
Disconnect-ExchangeOnline -Confirm:$false -WarningAction:SilentlyContinue
```


# Appendix: If the organization does not allow using basic Auth for service account, then use Application Delegated method instead
https://learn.microsoft.com/en-us/powershell/exchange/app-only-auth-powershell-v2?view=exchange-ps
Note that the app only method with certificate authentication does not support commands other than Get-ComplianceSearch. So the Delegated scenario is required here. The below steps are used for Delegated scenario, where we:
- Request access to the common Application by Microsoft for this flow (the Application ID is a0c73c16-a7e3-4564-9a95-2bdf47383716). This application is Microsoft owned.
- Get access-token and use the token to connect to PS
- Instead of using Connect-IPPSSession, we connect and authenticate a PSSession to https://ps.compliance.protection.outlook.com/powershell-liveid using Oauth. It then redirect and create a session for Security & Compliance Powershell.

### First, authenticate and get access bearer token
Request authentication, use device code to authenticate using web UI to initiate the authentication.
```
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("return-client-request-id", "true")
$headers.Add("Content-Type", "application/x-www-form-urlencoded")
$body = "client_id=a0c73c16-a7e3-4564-9a95-2bdf47383716&scope=offline_access%20https%3A%2F%2Foutlook.office365.com%2F.default"
$response = Invoke-RestMethod 'https://login.microsoftonline.com/fa8a7288-73bf-4f16-8551-54ecd2a62606/oauth2/v2.0/devicecode' -Method 'POST' -Headers $headers -Body $body
$response | ConvertTo-Json
```

Get bearer access token and its refresh toekn
```
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("return-client-request-id", "true")
$headers.Add("Content-Type", "application/x-www-form-urlencoded")

$body = "grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Adevice_code&device_code=<device-code-from-the-1st-request>&client_id=a0c73c16-a7e3-4564-9a95-2bdf47383716"

$response = Invoke-RestMethod 'https://login.microsoftonline.com/fa8a7288-73bf-4f16-8551-54ecd2a62606/oauth2/v2.0/token' -Method 'POST' -Headers $headers -Body $body
$response | ConvertTo-Json
```

As the access_token is expired in about 1-1.5 hour, get access_token again from refresh_token
```
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Content-Type", "application/x-www-form-urlencoded")

$body = "grant_type=refresh_token&client_id=a0c73c16-a7e3-4564-9a95-2bdf47383716&scope=offline_access%20https%3A%2F%2Foutlook.office365.com%2F.default&refresh_token=<refresh-token-from-the-2nd-request>"

$response = Invoke-RestMethod 'https://login.microsoftonline.com/organizations/oauth2/v2.0/token' -Method 'POST' -Headers $headers -Body $body
$response | ConvertTo-Json
```

### Second, authenticate and get access bearer token
```
$bearer_token = '<your-bearer-token>'
$upn='<upn-of-the-user-to-delegate>'
$tokenValue = ConvertTo-SecureString "Bearer $bearer_token" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($upn, $tokenValue)

$SccSession = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://ps.compliance.protection.outlook.com/powershell-liveid?BasicAuthToOAuthConversion=true" -Credential $credential -AllowRedirection -Authentication Basic
Import-PSSession -Session $SccSession
```
### After you have a session, run other Compliance Search commands as usual


