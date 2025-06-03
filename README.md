# Upgrade Guide: Business Central 13 → Business Central 24

**Database:** `RBA` (currently on BC 13)  
**Target Version:** BC 24

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1: Download and Install BC 13 Cumulative Update](#step-1-download-and-install-bc-13-cumulative-update)
3. [Step 2: Apply BC 13 License and Point to RBA](#step-2-apply-bc-13-license-and-point-to-rba)
4. [Step 3: Uninstall All Extensions (BC 13)](#step-3-uninstall-all-extensions-bc-13)
5. [Step 4: Clear Old Server- and Debugger-Related Tables](#step-4-clear-old-server--and-debugger-related-tables)
6. [Step 5: Convert the BC 13 Database to BC 14 Format](#step-5-convert-the-bc-13-database-to-bc-14-format)
7. [Step 6: Connect a BC 14 Server Instance to the Converted Database](#step-6-connect-a-bc-14-server-instance-to-the-converted-database)
8. [Step 7: Compile All Objects in the Converted Database](#step-7-compile-all-objects-in-the-converted-database)
9. [Step 8: Prepare Application Object Text Files for BC 14 → BC 24](#step-8-prepare-application-object-text-files-for-bc-14--bc-24)
10. [References](#references)

---

## Prerequisites

- Admin access to SQL Server hosting `RBA`
- Installed BC 13, BC 14, and BC 24 environments
- BC 13 and BC 14 Development Environments available
- Know your custom object ID ranges (e.g., 51511000–51515000)

---

## Step 1: Download and Install BC 13 Cumulative Update

1. Download the latest BC 13 CU from Microsoft.
2. Install it on the server running BC 13.

---

## Step 2: Apply BC 13 License and Point to RBA

```powershell
Import-Module 'C:\Program Files\Microsoft Dynamics 365 Business Central\130\Service\NavAdminTool.ps1'

Start-NAVServerInstance -ServerInstance BC130
Import-NAVServerLicense -ServerInstance BC130 -LicenseFile "C:\BCLICENSE\BC130.flf"
Restart-NAVServerInstance -ServerInstance BC130

Set-NAVServerConfiguration -ServerInstance BC130 -KeyName DatabaseName -KeyValue "RBA"
```

---

## Step 3: Uninstall All Extensions (BC 13)

```powershell
Get-NAVAppInfo -ServerInstance BC130 -Tenant default | % {
  Uninstall-NAVApp -ServerInstance BC130 -Tenant default -Name $_.Name -Version $_.Version -Force
}
```

---

## Step 4: Clear Old Server- and Debugger-Related Tables

Stop the BC 13 instance and run this SQL:

```sql
DELETE FROM [RBA].[dbo].[Server Instance];
DELETE FROM [RBA].[dbo].[Debugger Breakpoint];
```

---

## Step 5: Convert the BC 13 Database to BC 14 Format

1. Open BC 14 Dev Environment as admin.
2. File → Database → Open → select `RBA`.
3. When prompted, choose to convert the database.
4. Choose **Schema Sync Later**.

---

## Step 6: Connect a BC 14 Server Instance to the Converted Database

```powershell
Set-NAVServerConfiguration -ServerInstance BC140 -KeyName DatabaseName -KeyValue "RBA"
Start-NAVServerInstance -ServerInstance BC140
```

---

## Step 7: Compile All Objects in the Converted Database

1. Open BC 14 Dev Environment as admin.
2. Tools → Compile → **Synchronize Schema: Later**

---

## Step 8: Prepare Application Object Text Files for BC 14 → BC 24

### 8.1 Folder Structure

Create folders:

```
C:\BCUPGRADE13TO24
├─ Tables
├─ Table Extensions
├─ Pages
├─ Page Extensions
├─ Reports
├─ Report Extensions
├─ XMLPort
├─ CodeUnits
├─ Queries
```

### 8.2 Export Commands

```powershell
# Tables
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Tables\OldBaseVersion.txt -Filter "Type=Table;Id=51511000..51515000"

# Table Extensions
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Table Extensions\OldBaseVersion.txt -Filter "Type=Table;Modified=1;Id=1..51510999"

# Pages
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Pages\OldBaseVersion.txt -Filter "Type=Page;Id=51511000..51515000"

# Page Extensions
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Page Extensions\OldBaseVersion.txt -Filter "Type=Page;Modified=1;Id=1..51510999"

# Reports
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Reports\OldBaseVersion.txt -Filter "Type=Report;Id=51511000..51515000"

# Report Extensions
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Report Extensions\OldBaseVersion.txt -Filter "Type=Report;Modified=1;Id=1..51510999"

# XMLPorts
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\XMLPort\OldBaseVersion.txt -Filter "Type=XMLport;Id=51511000..51515000"

# Codeunits
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\CodeUnits\OldBaseVersion.txt -Filter "Type=Codeunit;Id=51511000..51515000"

# Queries
Export-NAVApplicationObject -DatabaseServer CHALITO -DatabaseName "RBA" -Path C:\BCUPGRADE13TO24\Queries\OldBaseVersion.txt -Filter "Type=Query;Id=51511000..51515000"
```
### 8.3 Conversion .txt to AL commands
``` powershell
cd "C:\Program Files (x86)\Microsoft Dynamics 365 Business Central\140\RoleTailored Client"

#Reports
txt2al --source=C:\BCUPGRADE13TO24\Reports --target=C:\BCUPGRADE13TO24\Reports\AL

#Report Extensions
txt2al --source=C:\BCUPGRADE13TO24\ReportExtensions --target=C:\BCUPGRADE13TO24\ReportExtensions\AL

#Tables
txt2al --source=C:\BCUPGRADE13TO24\Table --target=C:\BCUPGRADE13TO24\Table\AL

#Table Extensions
txt2al --source=C:\BCUPGRADE13TO24\TableExtensions --target=C:\BCUPGRADE13TO24\TableExtensions\AL

#Pages
txt2al --source=C:\BCUPGRADE13TO24\Pages --target=C:\BCUPGRADE13TO24\Pages\AL

#Page Extensions
txt2al --source=C:\BCUPGRADE13TO24\PageExtensions --target=C:\BCUPGRADE13TO24\PageExtensions\AL

#Query
txt2al --source=C:\BCUPGRADE13TO24\Query --target=C:\BCUPGRADE13TO24\Query\AL

#XMLPort
txt2al --source=C:\BCUPGRADE13TO24\XMLPort --target=C:\BCUPGRADE13TO24\XMLPort\AL

#CodeUnit
txt2al --source=C:\BCUPGRADE13TO24\CodeUnit --target=C:\BCUPGRADE13TO24\CodeUnit\AL 
```

---

## References

- [Upgrading the Data to Business Central](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/upgrade/upgrading-the-data)
- [Upgrading the Application Code in Business Central](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/upgrade/upgrading-the-application-code)
- [Upgrading Customized C/AL Application to Microsoft Base Application version 24](https://learn.microsoft.com/en-us/dynamics365/business-central/dev-itpro/upgrade/upgrade-to-microsoft-base-app-v24)
---

---

**✅ You are now ready to upgrade your RBA BC 13 database to BC 24.**
