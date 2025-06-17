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
