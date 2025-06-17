# 🔐 CheckMailboxFullAccess.ps1

PowerShell script to connect to Exchange Online and check which users have **Full Access** permissions to a shared mailbox in Microsoft 365.

---

## 📦 Features

- ✅ Installs the `ExchangeOnlineManagement` module if missing
- 🔐 Connects securely to Exchange Online
- 📋 Lists all users with Full Access permissions on a specified shared mailbox
- 🚪 Cleanly disconnects session after execution

---

## 🧰 Prerequisites

- PowerShell 5.1+ or PowerShell Core
- Microsoft 365 admin credentials
- Internet connection
- Windows OS (recommended; cross-platform may require additional handling)

---

### 💻 Script

```powershell
# CheckMailboxFullAccess.ps1
# Purpose: Check which users have Full Access permissions to a shared mailbox in Exchange Online.

# Step 1: Ensure ExchangeOnlineManagement module is installed
if (-not (Get-Module -ListAvailable -Name ExchangeOnlineManagement)) {
    Write-Output "Installing ExchangeOnlineManagement module..."
    Install-Module ExchangeOnlineManagement -Scope CurrentUser -Force
} else {
    Write-Output "ExchangeOnlineManagement module already installed."
}

# Step 2: Import module
Import-Module ExchangeOnlineManagement

# Step 3: Connect to Exchange Online (admin login)
try {
    Connect-ExchangeOnline -UserPrincipalName (Read-Host "Enter your admin email address") -ErrorAction Stop
    Write-Output "Connected to Exchange Online."
} catch {
    Write-Error "❌ Failed to connect to Exchange Online. Check your credentials or network."
    exit 1
}

# Step 4: Prompt for the shared mailbox email address
$mailbox = Read-Host "Enter the email address of the shared mailbox to check permissions for"

# Step 5: Query Full Access permissions
try {
    $permissions = Get-MailboxPermission -Identity $mailbox | Where-Object {
        $_.AccessRights -contains "FullAccess" -and $_.IsInherited -eq $false
    }

    if ($permissions) {
        Write-Host "`n📋 Full Access permissions for mailbox: $mailbox`n" -ForegroundColor Cyan
        $permissions | Select-Object User, AccessRights, IsInherited | Format-Table -AutoSize
    } else {
        Write-Output "✅ No Full Access permissions found for mailbox '$mailbox'."
    }
} catch {
    Write-Error "❌ Error retrieving permissions. Make sure the mailbox exists and you have proper rights."
}

# Step 6: Disconnect the session
Disconnect-ExchangeOnline -Confirm:$false
```

---

# Export Microsoft 365 Group Membership for a User (PowerShell)

This script uses Microsoft Graph PowerShell SDK to export all Microsoft 365 (Entra ID) groups that a user is a member of, including Group Name and Group ID.

## 🧰 Prerequisites

- PowerShell 7+ (macOS/Linux/Windows)
- Microsoft Graph PowerShell Module

<pre lang="markdown">
```powershell
<#
.SYNOPSIS
    Exports Microsoft 365 groups a user is a member of, including Group Name and ID.
.DESCRIPTION
    Uses Microsoft Graph PowerShell to query Entra ID (Azure AD) and export group membership to CSV.
.NOTES
    Requires Microsoft.Graph module and proper permissions.
#>

# Parameters
param (
    [Parameter(Mandatory = $true)]
    [string]$UserPrincipalName,

    [Parameter()]
    [string]$OutputPath = "$HOME/Downloads/user_groups.csv"
)

# Ensure Microsoft.Graph module is installed
if (-not (Get-Module -ListAvailable -Name Microsoft.Graph)) {
    Install-Module Microsoft.Graph -Scope CurrentUser -Force
}

# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Group.Read.All", "User.Read.All"

# Get the User ID
try {
    $UserId = (Get-MgUser -UserId $UserPrincipalName).Id
} catch {
    Write-Error "Could not find user: $UserPrincipalName"
    exit
}

# Get group IDs the user is a member of
$GroupIds = Get-MgUserMemberOf -UserId $UserId | Select-Object -ExpandProperty Id

# Fetch group names and IDs
$Groups = foreach ($GroupId in $GroupIds) {
    try {
        $Group = Get-MgGroup -GroupId $GroupId
        [PSCustomObject]@{
            GroupName = $Group.DisplayName
            GroupId   = $Group.Id
        }
    } catch {
        Write-Warning "Failed to fetch details for Group ID: $GroupId"
    }
}

# Export to CSV
$Groups | Export-Csv -Path $OutputPath -NoTypeInformation
Write-Output "✅ Export complete: $OutputPath"

# Disconnect
Disconnect-MgGraph
```
</pre>
