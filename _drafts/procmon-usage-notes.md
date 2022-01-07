# Process monitor (procmon) usage notes

## Operation types

```
CloseFile
CreateFile
CreateFileMapping
DeviceIoControl
FileSystemControl
FlushBuffersFile
Load Image
LockFile
NotifyChangeDirectory
Process Create
Process Exit
Process Profiling
Process Start
RegFlushKey
RegUnloadKey
QueryAllInformationFile
QueryAttributeInformationVolume
QueryAttributeTagFile
QueryBasicInformationFile
QueryDeviceRelations
QueryDirectory
QueryEAFile
QueryFileInternalInformationFile
QueryFullSizeInformationVolume
QueryInformationVolume
QueryNameInformationFile
QueryNetworkOpenInformationFile
QueryNormalizedNameInformationFile
QueryObjectIdInformationVolume
QueryOpen
QueryPositionInformationFile
QueryRemoteProtocolInformation
QuerySecurityFile
QuerySizeInformationVolume
QueryStandardInformationFile
QueryStreamInformationFile
ReadFile
RegCloseKey
RegCreateKey
RegDeleteKey
RegDeleteValue
RegEnumKey
RegEnumValue
RegLoadKey
RegOpenKey
RegQueryKey
RegQueryKeySecurity
RegQueryMultipleValueKey
RegQueryValue
RegSetInfoKey
RegSetKeySecurity
RegSetValue
SetAllocationInformationFile
SetBasicInformationFile
SetDispositionInformationFile
SetDispositionInformationEx
SetEndOfFileInformationFile
SetPositionInformationFile
SetRenameInformationFile
SetSecurityFile
SetStorageReservedIdInformation
TCP Accept
TCP Connect
TCP Disconnect
TCP Receive
TCP Reconnect
TCP Retransmit
TCP Send
TCP TCPCopy
Thread Create
Thread Exit
UDP Receive
UDP Send
UnlockFileSingle
QueryEaInformationFile
WriteFile
QueryDeviceInformationVolume
QueryNetworkPhysicalNameInformationFile
```

Source: https://gist.github.com/mgeeky/f0d13172d557e5860c0301dbf847de60#file-procmon_operationst-txt

## Locating registry changes

You can use the following methods:
- Filter the `RegCreateKey`, `RegDeleteKey`, `RegDeleteValue`, `RegSetInfoKey`, `RegSetKeySecurity`, `RegSetValue` operations when capturing.
- Use [RegFromApp](http://www.nirsoft.net/utils/reg_file_from_application.html) from NirSoft: RegFromApp monitors the Registry changes made by the application that you selected, and creates a standard RegEdit registration file (.reg) that contains all the Registry changes made by the application. You can use the generated .reg file to import these changes with RegEdit when it's needed.
