# MS-Compliance-Powershell-BasicAuth


## Open Powershell console and start typing these command one by one:

### Install ExchangeOnlineManagement module
```Install-Module -Name ExchangeOnlineManagement```

### Use ExchangeOnlineManagement module
```Import-Module ExchangeOnlineManagement```

### Connect to Sec & Compliance Powershell
```
$password = "<your-password>"
$username = "<your-username>"
$tid = "<your-tenant-id>"

$secPassword = ConvertTo-SecureString "$password" -AsPlainText -Force
$delegated_cred = New-Object System.Management.Automation.PSCredential ("$username", $secPassword)
$CommandName = "Get-ComplianceSearch"
$connection_uri = "https://ps.compliance.protection.outlook.com/powershell-liveid/"
$azure_ad_authorization_endpoint_uri = "https://login.microsoftonline.com/$tid"
Connect-IPPSSession -Credential $delegated_cred -CommandName $CommandName -WarningAction:SilentlyContinue -ConnectionUri $connection_uri -AzureADAuthorizationEndpointUri $azure_ad_authorization_endpoint_uri | Out-Null
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

### Disconnect the session
```Disconnect-ExchangeOnline -Confirm:$false -WarningAction:SilentlyContinue```

