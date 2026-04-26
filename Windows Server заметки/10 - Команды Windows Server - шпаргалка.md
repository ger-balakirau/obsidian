# Команды Windows Server - шпаргалка

Более учебные памятки:

- [[14 - Оснастки Windows - msc команды]]
- [[15 - Команды Windows админа наизусть]]

## Сеть

```cmd
ipconfig /all
ipconfig /flushdns
nslookup domain.local
ping 8.8.8.8
tracert ya.ru
route print
```

PowerShell:

```powershell
Test-NetConnection server -Port 445
Get-NetIPConfiguration
Get-DnsClientServerAddress
```

## Windows NAT

Подробнее: [[11 - Windows NAT через PowerShell]]

```powershell
Get-NetNat
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.0.0.0/8
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/24
Remove-NetNat -Name "LabNAT"
Remove-NetNat -Name "LabNAT" -Confirm:$false
Get-NetNat | Remove-NetNat
Get-NetIPConfiguration
Get-NetAdapter
Get-NetIPAddress
```

## Домен и GPO

```cmd
whoami
whoami /groups
gpupdate /force
gpresult /r
gpresult /h report.html
nltest /dsgetdc:domain.local
```

## Join domain

```powershell
Add-Computer -DomainName domain.local -Credential domain\admin -Restart
```

## Службы

```powershell
Get-Service
Get-Service -Name Spooler
Restart-Service Spooler
Start-Service Spooler
Stop-Service Spooler
```

## События

```powershell
Get-EventLog -LogName System -Newest 30
Get-EventLog -LogName System -EntryType Error -Newest 30
Get-WinEvent -LogName System -MaxEvents 30
```

## Файловые шары

```cmd
net use
net use Z: \\server\share
net use Z: /delete
```

PowerShell:

```powershell
Get-SmbShare
New-SmbShare -Name Share -Path D:\Share
```

## Active Directory PowerShell

```powershell
Get-ADUser username
Get-ADGroup "Group Name"
Get-ADGroupMember "Group Name"
Add-ADGroupMember -Identity "Group Name" -Members username
Disable-ADAccount -Identity username
```

## Минимум на память

```cmd
ipconfig /all
nslookup domain.local
gpupdate /force
gpresult /r
whoami /groups
nltest /dsgetdc:domain.local
```
