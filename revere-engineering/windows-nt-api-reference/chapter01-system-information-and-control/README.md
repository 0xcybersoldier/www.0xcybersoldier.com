---
description: >-
  The system services described in this chapter operate on the system as a whole
  rather than on individual objects within the system.They mostly gather
  information about the performance and operation of
---

# Chapter01 - System Information and Control

[**ZwQuerySystemInformation**](#user-content-fn-1)[^1] **:**&#x20;

```
ZwQuerySystemInformation queries information about the system.
NTSYSAPI
NTSTATUS
NTAPI
ZwQuerySystemInformation(
IN SYSTEM_INFORMATION_CLASS SystemInformationClass,
IN OUT PVOID SystemInformation,
IN ULONG SystemInformationLength,
OUT PULONG ReturnLength OPTIONAL
);
```



**Parameters :**&#x20;

```
SystemInformationClass
The type of system information to be queried.The permitted values are a subset of
the enumeration SYSTEM_INFORMATION_CLASS, described in the following section.
```

```
SystemInformation
Points to a caller-allocated buffer or variable that receives the requested system
information.
```

```
SystemInformationLength
The size in bytes of SystemInformation, which the caller should set according to the
given SystemInformationClass.
```



[^1]: 
