# azure-ad-utils-linux

> 本リポジトリは、[azure-activedirectory-powershell](https://github.com/AzureAD/azure-activedirectory-powershell/tree/master/Modules/AzureADUtils) にて公開されている AzureADUtils を Linux 用に移植したものです。Red Hat Linux Enterprise 7.3 にて動作確認済みです。

Linux 上に、AzureADUtilsLinux.psm1 を配置し、以下の手順で実行ください。

## PowerShell のインストール

Red Hat Enterprise Linux (RHEL) 7 Linux 向け PowerShell Core は公式 Microsoft リポジトリに公開されており、インストール (と更新) を簡単に行うことが可能です。

```
# Register the Microsoft RedHat repository
curl https://packages.microsoft.com/config/rhel/7/prod.repo | sudo tee /etc/yum.repos.d/microsoft.repo

# Install PowerShell
sudo yum install -y powershell

# Start PowerShell
pwsh
```

## AzureADUtilsLinux.psm1 の取得

以下は、適当な Web サーバー上にファイルを配置し取得した際の例です。

```
PS /home/contoso> wget https://www.example.com/AzureADUtilsLinux.psm1

--2019-07-24 04:08:48--  https://www.example.com/AzureADUtilsLinux.psm1
Resolving www.example.com (www.example.com)... 52.176.224.96
Connecting to jwww.example.com (www.example.com) |52.176.224.96|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 25423 (25K) [application/octet-stream]
Saving to: 'AzureADUtilsLinux.psm1'

100%[==============================================================================>] 25,423      --.-K/s   in 0s

2019-07-24 04:08:48 (357 MB/s) - 'AzureADUtilsLinux.psm1' saved [25423/25423]
```

## PowerShell の実行

### AzureADUtilsLinux のインポート

以下のコマンドで、インポート処理を行います。この時、Import-Module でエラーが発生しますが、無視いただいて結構です。

```
PS /home/contoso> Import-Module .\AzureADUtilsLinux.psm1
Current module is not part of the Powershell Module path. Please run Install-AzureADUtilsLinuxModule, restart the PowerShell session and try again..

PS /home/contoso> Install-AzureADUtilsLinuxModule
Active Directory Authentication Library Nuget doesn't exist. Downloading now ...
nuget.exe not found. Downloading from https://clicktime.symantec.com/3PB2fUnEJGcef2RptcmkrF97Vc?u=http%3A%2F%2Fwww.nuget.org%2Fnuget.exe ...
/home/contoso/WindowsPowerShell/Modules/AzureADUtilsLinux/Nugets/nuget.exe update -self

Import-Module : The specified module 'AzureADUtilsLinux' was not loaded because no valid module file was found in any module directory.
At /home/contoso/AzureADUtilsLinux.psm1:650 char:5
+     Import-Module AzureADUtilsLinux

Function        Get-AzureADAppStaleLicensingReport                 0.0        AzureADUtilsLinux
Function        Invoke-AzureADGraphAPIQuery                        0.0        AzureADUtilsLinux
Function        Join                                               0.0        AzureADUtilsLinux
Function        New-AzureADApplicationCertificateCredential        0.0        AzureADUtilsLinux
Function        Remove-AzureADOnPremUsers                          0.0        AzureADUtilsLinux
```

### モジュールが読み込まれたことの確認

Get-Module を実行し、AzureADUtilsLinux が結果に含まれることを確認します。

```powershell
PS /home/contoso> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        AzureADUtilsLinux                   {Get-AzureADAppAssignmentReport, Get-AzureADAppStaleLicensin…
Manifest   6.1.0.0    Microsoft.PowerShell.Management     {Add-Content, Clear-Content, Clear-Item, Clear-ItemProperty…}
Manifest   6.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object…}
Script     2.0.0      PSReadLine                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PSRe…
```

### AzureADUtilsLinux の実行

以下のようにして AzureADUtilsLinux を実行します。事前に Azure AD 上にアプリケーションを登録いただき、client id と client secret をご準備ください。

```powershell
$ClientID     = "<Application ID>"
$ClientSecret = ConvertTo-SecureString -String "<Client Secret>" -AsPlainText -Force
$tenantdomain   = "yourtenant.onmicrosoft.com"
$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $ClientID, $ClientSecret

# Get Access Token
$AccessToken = Get-AzureADGraphAPIAccessTokenFromAppKey -TenantDomain $tenantdomain -ClientCredential $Credential

# Set date range
$daysago = "{0:s}" -f (get-date).AddDays(-7) + "Z"

# Getting logs
Invoke-AzureADGraphAPIQuery -TenantDomain $tenantdomain -AccessToken $AccessToken -GraphQuery "/activities/signinEvents?api-version=beta&`$filter=signinDateTime gt $daysago" | Export-Csv -Path "signin.csv"

# Getting logs
Invoke-AzureADGraphAPIQuery -TenantDomain $tenantdomain -AccessToken $AccessToken -GraphQuery "/activities/audit?api-version=beta&`$filter=activityDate gt $daysago" | Export-Csv -Path "audit.csv"
```