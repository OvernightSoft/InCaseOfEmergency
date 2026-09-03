### SFC in offline mode
```
sfc /scannow
  /offbootdir=D:\
  /offwindir=D:\Windows
  /offlogfile=<a path>
```

### DISM in offline mode
```
dism /image:D:\
  /cleanup-image
  /restorehealth
  /source:d:\windows
  /scratchdir:<a path>
```

### BOOTREC
```
bootrec /fixmbr
bootrec /fixboot
bootrec /scanos
bootrec /rebuildbcd
```

### BOOTSECT
```
bootsect /NT60 <DRIVELETTER:> (/MBR)
```

### BCDEDIT
```
bcdedit [/store ...] /enum [all]
bcdedit [/store ...] /create /D "<description>" [/application ...]
bcdedit [/store ...] /set {...} <KEY> = <VALUE>
bcdedit [/store ...] /displayorder <GUID> /addlast | /addfirst | / remove
```

References:
- [BCDEdit /set](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/bcdedit--set)


#### Enable / disable boot logging

```
bcdedit [/store ...] /set {default} bootlog yes | no
```


#### Create basic BCD

```
bcdedit /createstore <file>
bcdedit /store <file> /create {bootmgr} /D "Windows Boot Manager"
bcdedit /store <file> /set {bootmgr} device partition=C:\
bcdedit /store <file> /timeout 10
bcdedit /store <file> /create /D "Windows 10" /application osloader
bcdedit /store <file> /default {...}
bcdedit /store <file> /set {...} device partition=D:\
bcdedit /store <file> /set {...} osdevice partition=D:\
bcdedit /store <file> /set {...} path \windows\system32\winload.exe
bcdedit /store <file> /set {...} systemroot \windows
bcdedit /store <file> /displayorder {...} /addlast
```

### List of (improperly) mounted devices

A cloned system may sometimes fail to boot because wrong devices are listed in registry key:

`HKLM\SYSTEM\MountedDevices`