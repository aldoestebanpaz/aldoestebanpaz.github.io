# Some Poweshell useful commands

## Basic commands

### grep equivalent

Select-String or sls is the command that allows filteting. e.g.:

```powershell
git help -a | sls credential-
```

### Get file size

```powershell
# size in bytes
(Get-Item $file).Length
# size in KB
(Get-Item $file).length/1KB
# size in MB
(Get-Item $file).length/1MB
# size in GB
(Get-Item $file).length/1GB
```

### Calculate hash of files

```powershell
# default, SHA256
Get-FileHash $file | Select-Object Hash
# SHA384 -- Accepted values: SHA1, SHA256, SHA384, SHA512, MD5
Get-FileHash $file -Algorithm SHA384 | Select-Object Hash
```

### Calculate hash of string

```powershell
$stream = [System.IO.MemoryStream]::new()
$streamWriter = [System.IO.StreamWriter]::new($stream)
$streamWriter.write("Hello world")
$streamWriter.Flush()
$stream.Position = 0
Get-FileHash -InputStream $stream | Select-Object Hash
```

### Check hash of streams

```powershell
$webClient = [System.Net.WebClient]::new()
$fileUrl = 'https://github.com/PowerShell/PowerShell/releases/[...].deb'
$publishedHash = '8E28E54D601F0751922DE24632C1E716B4684876255CF82304A9B19E89A9CCAC'
$FileHash = Get-FileHash -InputStream ($webClient.OpenRead($fileUrl))
$FileHash.Hash -eq $publishedHash
```

## Analysis

### Getting information about the Shell

```powershell
# Getting powershell version
`$PSVersionTable`
# also
`Get-Host`
```

### Get command behind an alias

The following are some examples of using Get-Alias:

```powershell
# sls -> Select-String
Get-Alias -Name sls

# iwr -> Invoke-WebRequest
Get-Alias -Name iwr
```

### Getting information about Modules

```powershell
# List the modules that have been imported into the current session
Get-Module

# List the modules that are available to be imported from the paths specified in the PSModulePath environment variable ($env:PSModulePath)
Get-Module -ListAvailable

# Get commands in the module
Get-Command -Module oh-my-posh
```

## Tools for development

### Generate GUID

```powershell
[guid]::NewGuid()
```

## Date, time and an timezone

### Show all timezones

```powershell
$timezones = [System.TimeZoneInfo]::GetSystemTimeZones()
foreach ($tz in $timezones.GetEnumerator()) {
    Write-Host "Offset: $($tz.BaseUtcOffset); Id: $($tz.Id); StandardName: $($tz.StandardName)"
}
```

### Show a date and time in different timezones

The following code parse an ISO 8601 date and time and shows the equivalent date and time for the different timezones:

```powershell
# usage:
#     powershell -executionpolicy ByPass -File .\timezone_converter.ps1 -datetime <ISO datetime>
#     example:
#      > powershell -executionpolicy ByPass -File .\timezone_converter.ps1 -datetime 2021-12-23T22:00:03.819157-08:00
#      Input:     2021-12-23T22:00:03.819157-08:00
#      Parsed as: 2021-12-23T22:00:03.8191570-08:00
#      2021-12-23T18:00:03.8191570-12:00    (Offset: -12:00:00; Id: Dateline Standard Time; StandardName: Dateline Standard Time)
#      2021-12-23T19:00:03.8191570-11:00    (Offset: -11:00:00; Id: UTC-11; StandardName: UTC-11)
#      2021-12-23T20:00:03.8191570-10:00    (Offset: -10:00:00; Id: Aleutian Standard Time; StandardName: Aleutian Standard Time)
#      ...

param ([parameter(Mandatory=$true)] [string] $datetime)

Write-Host "Input:     $datetime"
$datetimeoffset = [System.DateTimeOffset]::Parse($datetime)
Write-Host "Parsed as: $($datetimeoffset.ToString("o"))"

$timezones = [System.TimeZoneInfo]::GetSystemTimeZones()
foreach ($tz in $timezones.GetEnumerator()) {
    $datetimeoffsetForTz = $datetimeoffset.ToOffset($tz.BaseUtcOffset)
    Write-Host "$($datetimeoffsetForTz.ToString("o"))    (Offset: $($tz.BaseUtcOffset); Id: $($tz.Id); StandardName: $($tz.StandardName))"
}
```

## JSON files

Suppose you have a JSON file list.json with the following format:

```
{
    ...
	"list_items": [
        ...
		{
            ...
			"created_at": "2021-12-23T22:58:43.000000-07:00"
		},
        ...
    ]
}
```

### Getting an item from a list

The following script prints the content of the item for the index number passed as a parameter:

```powershell
# usage:
#     powershell -executionpolicy ByPass -File .\show_item.ps1 -index <INDEX>
# example:
#     > powershell -executionpolicy ByPass -File .\show_item.ps1 -index 1999
#     ... created_at
#     --- ----------
#     ... 2021-12-23T22:58:43.000000-07:00

param ([parameter(Mandatory=$true)] [int] $index)

$json = Get-Content '.\list.json' | ConvertFrom-Json
$json.list_items[$index]
```

### Sorting a list of objects

The following script will sort the objects inside list_items and will save the result into sorted_list.json:

```powershell
# usage:
#     powershell -executionpolicy ByPass -File .\sort.ps1
# example:
#     > powershell -executionpolicy ByPass -File .\sort.ps1
#     Input list size: 2000
#     Sorted list size: 2000

$json = Get-Content '.\list.json' | ConvertFrom-Json
Write-Host "Input list size: $(($json.list_items | measure).Count)"
$json.list_items = $json.list_items | Sort-Object created_at
Write-Host "Sorted list size: $(($json.list_items | measure).Count)"
$json | ConvertTo-Json | Out-File '.\sorted_list.json'
```

### Slicing a list

The following script will extract a list from list_items given an index as the starting point and will save the result into sorted_list.json:

```powershell
# usage:
#     powershell -executionpolicy ByPass -File .\extract_list.ps1 -from <INDEX>
# example:
#     > powershell -executionpolicy ByPass -File .\extract_list.ps1 -from 1
#     Original list size: 2000
#     Sublist size: 1999

param ([parameter(Mandatory=$true)] [int] $from)

$json = Get-Content '.\list.json' | ConvertFrom-Json
Write-Host "Original list size: $(($json.list_items | measure).Count)"
$json.list_items = $json.list_items[$from..($json.list_items.Count - 1)]
Write-Host "Sublist size: $(($json.list_items | measure).Count)"
$json | ConvertTo-Json | Out-File '.\extracted_list.json'
```
