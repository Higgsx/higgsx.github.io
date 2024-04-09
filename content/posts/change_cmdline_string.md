+++
title = 'Dancing with PEB: Change CommandLine'
date = 2024-04-09T09:50:51-04:00
draft = false
+++
I am taking Sektor7 Intermediate Malware Development training and as an exercise, I decided to create toy project which will go through all active processes and change their `CommandLine` parameter.

First of all lets have a look at PEB structure of `notepad.exe` process:

![Untitled](/images/peb-images/1.png)

Here, we are mapping _PEB structure fields to actual PEB structure in memory
```
dt _PEB @$peb
```
One of the structure field is ProcessParameters

![Untitled](/images/peb-images/2.png)


ProcessParameters is type of `_RTL_USER_PROCESS_PARAMETERS` and for our interests, we want CommandLine field

![Untitled](/images/peb-images/3.png)

CommandLine field is of type `_UNICODE_STRING` and contains 3 fields: `Length`, `MaximumLength` and `Buffer`

![Untitled](/images/peb-images/4.png)

Here, Buffer is unicode string and our objective is to change this to a unicode string we choose.


code in a github is explained well with comments, but lets sneak peek

Here, we get a handle to remote process via `OpenProcess` WinAPI and request for `PROCESS_ALL_ACCESS`

![Untitled](/images/peb-images/code-1.png)


Here, we are requesting address of `NtQueryInformationProcess` and call this function to retreive PEB address of a remote process.
Some may say why using `GetProcAddress` and `GetModuleHandle` do dynamically retrieve its address, but I had little conflict when I included `winternl.h`
conflicts with `PEstructs.h`. So that's why I decided to dynamically resovle function address.

![Untitled](/images/peb-images/code-2.png)

for such cases we need a function pointer declared before hand:

```c
typedef NTSTATUS (*pNtQueryInformationProcess)(
  HANDLE           ProcessHandle,
  ULONG            ProcessInformationClass,
  PVOID            ProcessInformation,
  ULONG            ProcessInformationLength,
  PULONG           ReturnLength
);
```


Now, we are overwriting CommandLine field and Length field.

![Untitled](/images/peb-images/code-3.png)


Finally we compile with

```
cl.exe /nologo /Ox /MT /W0 /GS- /DNDEBUG /Tp *.cpp /link /OUT:implant.exe /SUBSYSTEM:CONSOLE
```
![Untitled](/images/peb-images/5.png)


![Untitled](/images/peb-images/6.png)


![Untitled](/images/peb-images/7.png)

Errors shown above is about permissions, we are getting because we don't have necessary process token (we are just regular user). In case you want to overwrite most of the process, just run as administrator or spawn cmd.exe via psexec

Here we see overwritten commandlines of several processes shown via Process Hacker

![Untitled](/images/peb-images/8.png)



You can see this toy project on github: [Repository](https://github.com/Higgsx/Toy_Project_CmdLine)