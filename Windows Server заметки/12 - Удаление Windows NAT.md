# Удаление Windows NAT

Эта заметка про удаление NAT, созданного через `New-NetNat`.

## Посмотреть NAT

Сначала посмотри, какие NAT-правила есть:

```powershell
Get-NetNat
```

Подробно:

```powershell
Get-NetNat | Format-List *
```

## Удалить NAT по имени

Пример для NAT с именем `LabNAT`:

```powershell
Remove-NetNat -Name "LabNAT"
```

PowerShell попросит подтверждение.

## Удалить без подтверждения

```powershell
Remove-NetNat -Name "LabNAT" -Confirm:$false
```

## Проверить, что NAT удалён

```powershell
Get-NetNat
```

Если список пустой, NAT-правил больше нет.

## Удалить все NAT-правила

Осторожно: команда удалит все NAT, которые видит Windows.

```powershell
Get-NetNat | Remove-NetNat
```

Без подтверждений:

```powershell
Get-NetNat | Remove-NetNat -Confirm:$false
```

## Типовой сценарий пересоздания NAT

Если нужно изменить подсеть, проще удалить старый NAT и создать новый:

```powershell
Remove-NetNat -Name "LabNAT" -Confirm:$false
New-NetNat -Name "LabNAT" -InternalIPInterfaceAddressPrefix 10.10.0.0/24
```

## Если NAT не удаляется

Запусти PowerShell от имени администратора.

Проверь имя:

```powershell
Get-NetNat
```

Проверь, нет ли опечатки:

```powershell
Get-NetNat -Name "LabNAT"
```

Если правило есть, но удаление не проходит, попробуй перезапустить службу Host Network Service:

```powershell
Restart-Service hns
```

Потом снова:

```powershell
Remove-NetNat -Name "LabNAT"
```

## Главное запомнить

- `Get-NetNat` показывает NAT.
- `Remove-NetNat -Name "LabNAT"` удаляет конкретный NAT.
- `Get-NetNat | Remove-NetNat` удаляет все NAT-правила.
- Для удаления нужен PowerShell от администратора.
- После удаления проверь результат через `Get-NetNat`.
