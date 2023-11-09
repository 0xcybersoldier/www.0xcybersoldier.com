# System Information and Control: ZwQuerySystem Information

```
ReturnLength : 
```

```
Optionally points to a variable that receives the number of bytes actually returned to
SystemInformation; if SystemInformationLength is too small to contain the available
information, the variable is normally set to zero except for two information classes
(6 and 11) when it is set to the number of bytes required for the available information.
If this information is not needed, ReturnLength may be a null pointer
```

**Return Value :**&#x20;

```
Returns STATUS_SUCCESS or an error status, such as STATUS_INVALID_INFO_CLASS,
STATUS_NOT_IMPLEMENTED or STATUS_INFO_LENGTH_MISMATCH.
```

**Related Win32 Functions :**&#x20;

```
GetSystemInfo, GetTimeZoneInformation, GetSystemTimeAdjustment, PSAPI functions,
and performance counters
```

_Remarks :_&#x20;

```
ZwQuerySystemInformation is the source of much of the information displayed by
“Performance Monitor” for the classes Cache, Memory, Objects, Paging File, Process,
Processor, System, and Thread. It is also frequently used by resource kit utilities that
display information about the system.
The ReturnLength information is not always valid (depending on the information
class), even when the routine returns STATUS_SUCCESS.When the return value indicates
STATUS_INFO_LENGTH_MISMATCH, only some of the information classes return an estimate
of the required length.
Some information classes are implemented only in the “checked” version of the
kernel. Some, such as SystemCallCounts, return useful information only in “checked”
versions of the kernel.
Some information classes require certain flags to have been set in NtGlobalFlags at
boot time. For example, SystemObjectInformation requires that
FLG_MAINTAIN_OBJECT_TYPELIST be set at boot time.
Information class SystemNotImplemented1 (4) would return STATUS_NOT_IMPLEMENTED
if it were not for the fact that it uses DbgPrint to print the text “EX:
SystemPathInformation now available via SharedUserData.” and then calls
DbgBreakPoint.The breakpoint exception is caught by a frame based exception handler
(in the absence of intervention by a debugger) and causes ZwQuerySystemInformation
to return with STATUS_BREAKPOINT.
```

**ZwSetSystemInformation  :**

```
ZwSetSystemInformation sets information that affects the operation of the system.
NTSYSAPI
NTSTATUS
NTAPI
ZwSetSystemInformation(
IN SYSTEM_INFORMATION_CLASS SystemInformationClass,
IN OUT PVOID SystemInformation,
IN ULONG SystemInformationLength
);
```

**Parameters :**&#x20;

```
SystemInformationClass:
The type of system information to be set.The permitted values are a subset of the
enumeration SYSTEM_INFORMATION_CLASS, described in the following section.
```

```
SystemInformation:
Points to a caller-allocated buffer or variable that contains the system information to
be set
```

```
SystemInformationLength:
The size in bytes of SystemInformation, which the caller should set according to the
given SystemInformationClass.
```

<pre><code><a data-footnote-ref href="#user-content-fn-1">Return Value</a> : 
Returns STATUS_SUCCESS or an error status, such as STATUS_INVALID_INFO_CLASS,
STATUS_NOT_IMPLEMENTED or STATUS_INFO_LENGTH_MISMATCH.
</code></pre>

```
Related Win32 Functions:
SetSystemTimeAdjustment
```

**Remarks :**&#x20;

```
At least one of the information classes uses the SystemInformation parameter for both
input and output
```

**SYSTEM\_INFORMATION\_CLASS :**&#x20;

```
The system information classes available in the “free” (retail) build of the system are
listed below along with a remark as to whether the information class can be queried,
set, or both. Some of the information classes labeled “SystemNotImplementedXxx” are
implemented in the “checked” build, and a few of these classes are briefly described
later
```

```
typedef enum _SYSTEM_INFORMATION_CLASS {
```

<table><thead><tr><th></th><th></th><th>Query</th><th data-type="number">Set</th></tr></thead><tbody><tr><td>SystemBasicInformation,</td><td>// 0</td><td>Y</td><td>null</td></tr><tr><td>SystemProcessorInformation,</td><td>// 1</td><td>Y</td><td>null</td></tr><tr><td>SystemPerformanceInformation,</td><td>// 2</td><td>Y</td><td>null</td></tr><tr><td>SystemTimeOfDayInformation,</td><td>// 3</td><td>Y</td><td>null</td></tr><tr><td>SystemNotImplemented1,</td><td>// 4</td><td>Y</td><td>null</td></tr><tr><td>SystemProcessesAndThreadsInformation,</td><td>// 5</td><td>Y</td><td>null</td></tr><tr><td>SystemCallCounts,</td><td>// 6</td><td>Y</td><td>null</td></tr><tr><td>SystemConfigurationInformation,</td><td>// 7</td><td>Y</td><td>null</td></tr><tr><td>SystemProcessorTimes,</td><td>// 8</td><td>Y</td><td>null</td></tr><tr><td>SystemGlobalFlag,</td><td>// 9</td><td>Y</td><td>1</td></tr><tr><td>SystemNotImplemented2,</td><td>// 10</td><td>Y</td><td>null</td></tr><tr><td>SystemModuleInformation,</td><td>// 11</td><td>Y</td><td>null</td></tr></tbody></table>

[^1]: 
