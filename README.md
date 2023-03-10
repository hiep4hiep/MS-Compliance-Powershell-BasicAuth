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


### Run a Compliance Search
```
$cmd_params = @{
            "Name" = $search_name
            "Case" = $case
            "ContentMatchQuery" = $kql
            "Description" = $description
            "AllowNotFoundExchangeLocationsEnabled" = $allow_not_found_exchange_locations
            "ExchangeLocation" = $exchange_location
            "ExchangeLocationExclusion" = $exchange_location_exclusion
            "PublicFolderLocation" = $public_folder_location
            "SharePointLocation" = $share_point_location
            "SharePointLocationExclusion" = $share_point_location_exclusion
}
$response = New-ComplianceSearch @cmd_params

return $response
```


### Disconnect the session
```Disconnect-ExchangeOnline -Confirm:$false -WarningAction:SilentlyContinue```




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
