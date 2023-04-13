---
layout: default
title: Powershell
permalink: /powershell/
---

```powershell
#Get the List of all Users in the Active Directory and Save to Current User's Desktop
$datetime = Get-Date -Format "yyyyMMddHHMM"
$file = "$HOME\Desktop\UserAccounts_" + $datetime + ".csv"
Get-ADUser -Filter * -Properties * | Select-Object DisplayName, SamAccountName, whenCreated, PasswordLastSet, LastLogonDate, Enabled | Format-Table | Export-CSV -Path $path$file  -NoTypeInformation

# Count the number of Domain Users
Get-ADUser -Filter * -Properties * | Select-Object DisplayName, SamAccountName, whenCreated, PasswordLastSet, LastLogonDate, Enabled | Measure-Object | Select-Object -expand count

# Domain Password Policy Parameters based on the active directory group policy
Get-ADDefaultDomainPasswordPolicy
```

## Windows Convinience Scripts

#### Port Listening

Port listening involves actively observing a network port for incoming traffic.  
It's a common practice used by servers and computers to receive and process data sent over a particular port.

```powershell
# Query Listening TCP Daemons
Write-Host "TCP Daemons"
Get-NetTCPConnection | Where-Object { $_.LocalAddress -eq "0.0.0.0" -and $_.State -eq "Listen" } |
    Select-Object LocalAddress, LocalPort,
        @{Name="PID"; Expression={ $_.OwningProcess }},
        @{Name="UserName"; Expression={ $Processes[[int]$_.OwningProcess].UserName }},
        @{Name="ProcessName"; Expression={ $Processes[[int]$_.OwningProcess].ProcessName }},
        @{Name="Path"; Expression={ $Processes[[int]$_.OwningProcess].Path }} |
        Sort-Object -Property LocalPort,UserName | Format-Table -AutoSize

# Query Listening UDP Daemons
Write-host "UDP Daemons"
Get-NetUDPEndpoint | Where-Object { $_.LocalAddress -eq "0.0.0.0" } |
Select-Object LocalAddress,LocalPort,
@{Name = "PID"; Expression = { $_.OwningProcess } },
@{Name = "UserName"; Expression = { $Processes[[int]$_.OwningProcess].UserName } },
@{Name = "ProcessName"; Expression = { $Processes[[int]$_.OwningProcess].ProcessName } },
@{Name = "Path"; Expression = { $Processes[[int]$_.OwningProcess].Path } } |
Sort-Object -Property LocalPort, UserName | Format-Table -AutoSize
```

#### Reboot Check

Reboot check is the process of verifying if a system or device has been successfully restarted after a reboot or shutdown.  
This is commonly done to ensure that any changes or updates take effect.

```powershell

$computerList = [System.Collections.ArrayList]::new()
$computers = Get-ADComputer -Filter 'operatingsystem -like "*server*" -and enabled -eq "true"' -Properties Name, IPv4Address | Select-Object -expandProperty IPv4Address
$computerList = $computers

$pendingRebootTests = @(
    @{
        Name = 'RebootPending'
        Test = { Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing'  Name 'RebootPending' -ErrorAction Ignore }
        TestType = 'ValueExists'
    }
    @{
        Name = 'RebootRequired'
        Test = { Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update'  Name 'RebootRequired' -ErrorAction Ignore }
        TestType = 'ValueExists'
    }
    @{
        Name = 'PendingFileRenameOperations'
        Test = { Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager' -Name 'PendingFileRenameOperations' -ErrorAction Ignore }
        TestType = 'NonNullValue'
    }
)

#Create a PowerShell Remoting session
foreach ($computer in $computerList){
    $session = New-PSSession -Computer $computer
    foreach ($test in $pendingRebootTests) {
        Invoke-Command -Session $session -ScriptBlock $test.Test
    }
}
```
