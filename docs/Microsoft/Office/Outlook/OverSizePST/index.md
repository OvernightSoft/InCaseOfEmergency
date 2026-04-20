When a PST or OST reaches its max size (50GB, by default), Outlook becomes unbearably slow, to the point that performing the required PST maintenance (deleting messages or moving/archiving them to a different PST) requires several minutes.

A workaround is:

- temporarily increase the PST/OST max size
- perform maintenance
- compact PST/OST under its 50GB limit
- reduce its max size to the default

The max size of a PST/OST can be changed by setting the `MaxLargeFileSize` DWord value in the `HKCU:\Software\Microsoft\Office\<version>\Outlook\PST` registry key.

This Powershell script does exactly this. It supports the parameters:
- *action* (mandatory): 
    - enable: to create the `MaxLargeFileSize` value 
    - disable: to remove it
- *size*: the desired max size, in MB. By default its value is 60000

#### PSTBigsize.ps1

```powershell

param(
  [Parameter(Mandatory)]
  [ValidateSet('enable','disable')]
  [string]$action,
  [ValidateRange(0,100000)]
  [int32]$size=60000
)

$basePath = 'hkcu:\software\microsoft\office'

$versions=get-childitem $basePath | where-object {$_.name -like '*.0'}

$propName='MaxLargeFileSize'
$propValue=$size

$versions | foreach-object {
  $path = join-path $basePath ($_.pschildname + "\outlook\pst")
  if (test-path $path) {
    Write-Debug "$path exists"
    if ($action -eq 'enable') {
      Write-Debug "Trying to create/update $propName to $propValue in $path"
      Set-ItemProperty -Path $path -Name $propName -Value $propValue -type DWord
      if ($?) {
        Write-Host "$propName set to $propValue in $path" -ForegroundColor green
      } else {
        write-Host "Error setting $propName to $propValue in $path" -ForegroundColor red
      } 
    } else {
      if ($null -eq (Get-ItemProperty -path $path -name $propName -ErrorAction SilentlyContinue)) {
        Write-Host "$propName is missing in $path" -ForegroundColor Yellow
      } else {
        Remove-ItemProperty -path $path -Name $propName
        if ($?) {
          Write-Host "$propName removed from $path" -ForegroundColor green
        } else {
          Write-Host "Error removing $propName from $path" -ForegroundColor red
        }
      }
    }
  } else {
    Write-Debug "$path does not exists" 
  }
}

```

You can quickly use it with these two companion batch files

#### PSTEnlarge.cmd

```
powershell -executionPolicy bypass -file %~dp0PSTBigSize.ps1 -action enable 
```

#### PSTReduce.cmd

```
powershell -executionPolicy bypass -file %~dp0PSTBigSize.ps1 -action disable 
```
