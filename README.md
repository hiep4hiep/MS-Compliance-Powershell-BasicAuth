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

### First, authenticate and get access bearer token
Request authentication, use device code to authenticate using web UI to initiate the authentication.
```
curl --location 'https://login.microsoftonline.com/<your-tenant-id>/oauth2/v2.0/devicecode' \
--header 'return-client-request-id: true' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=a0c73c16-a7e3-4564-9a95-2bdf47383716' \
--data-urlencode 'scope=offline_access https://outlook.office365.com/.default'
```

Get bearer access token and its refresh toekn
```
curl --location 'https://login.microsoftonline.com/fa8a7288-73bf-4f16-8551-54ecd2a62606/oauth2/v2.0/token' \
--header 'return-client-request-id: true' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:ietf:params:oauth:grant-type:device_code' \
--data-urlencode 'device_code=<device-code-from-the-1st-curl-request>' \
--data-urlencode 'client_id=a0c73c16-a7e3-4564-9a95-2bdf47383716'
```

When the access_token is expired, get access_token again from refresh_token
```
curl --location 'https://login.microsoftonline.com/organizations/oauth2/v2.0/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=refresh_token' \
--data-urlencode 'client_id=a0c73c16-a7e3-4564-9a95-2bdf47383716' \
--data-urlencode 'scope=offline_access https://outlook.office365.com/.default' \
--data-urlencode 'refresh_token=<your-refresh-token-from-the-2nd-request>'
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


