#### [CVE Report](https://github.com/pxcs/CVE-29343-Sysmon-list/)

<p align="center">
  <img width="300" height="300" src="/images/kali-linux.svg">
</p>

## OSCP and Vulnerability lessons.

After exploiting a vulnerability, a cyberattack can run malicious code, install malware, and even steal sensitive data. Learning this can help us [Attackers](https://github.com/pxcs/) to move into start targeting our victims. Healthcare organizations are prime targets for cyber attacks due to the sensitivity of the data they possess, including patient records and financial information. Cyber criminals exploit vulnerabilities using various tactics to gain unauthorized access.

#### Every contribution is appreciated

#### Here what you should pay attentions to!

> [!IMPORTANT]
> Hacking is not always a crime. In ethical hacking, a hacker is legally permitted to exploit security networks. In other words, the hacker has the appropriate consent or authorization to hack into a system. But sometimes we didnt have that kind of permission, so as long as Attackers report their action it should be ethical, stay safe.
<hr>

> [!NOTE]
> This repository was only for history learning and educational purposes only.

Thank you for reading.

## CVE-2014-2024

#### This is PoC for arbitrary file write bug in Sysmon version 14.14

After last patch Sysmon would check if Archive directory exists and if it exists it would check if archive directory is owned by NT AUTHORITY\SYSTEM and access is only granted to NT AUTHORITY\SYSTEM. 
If both conditions are true then Sysmon will write/delete files in that directory.

As its not possible to change ownership of file/directories as a low privilege user I had to find directory that is owned by SYSTEM but gives low privilege user (or any group low privilege user is a member of) full access or at least WRITE_DAC|DELETE|FILE_WRITE_ATTRIBUTES.

I could not find such directory on default installation but was able to create one by abusing Windows service tracing and RasMan service.

### SysmonEoP

Proof of Concept for arbitrary file delete/write in Sysmon (CVE-2022-41120/CVE-2022-44704)

### Vulnerability

Vulnerability is in code responsible for ClipboardChange event that can be reached through RPC. 
Local users can send data to RPC server which will then be written in C:\Sysmon directory (default ArchiveDirectory) and deleted afterwards.
In version before 14.11 Sysmon would not check if directory was created by low privilege user or if it's a junction which can be abused to perform arbitrary file delete/write (kinda limited as you can only write strings) in context of NT AUTHORITY\SYSTEM user.
In version 14.11/14.12, after initial fix, Sysmon would check if directory exists and would refuse to write/delete files if directory exists.
This patch was bypassed by letting Sysmon create C:\Sysmon directory first (using CreateDirectory API) and opening handle on it before SetFileSecurity is called and change DACL's on C:\Sysmon directory.

### Exploitation

All testing was done on Windows 10.

In my PoC I have chained arbitrary file delete/write to first delete setup information file of printer driver and then write modified .INF file (as spooler service is enabled by default and low privilege users can re-install printer drivers on windows clients).
Setup information files can be abused to perform all kind of operations such service creation, registry modification, file copy etc.
I choose to copy some of printer default DLL's in c:\windows\system32 and set permissions on it so that low privilege users can modify it, this is done using CopyFiles directive [@](https://learn.microsoft.com/en-us/windows-hardware/drivers/install/inf-copyfiles-directive). Once file is copied it is overwritten with DLL that will spawn elevated cmd.exe process.
It is possible to abuse just arbitrary file delete for LPE by abusing windows installer behavior (trick found by [@KLINIX5](https://twitter.com/KLINIX5) and is documented by ZDI here [@](https://www.zerodayinitiative.com/blog/2022/3/16/abusing-arbitrary-file-deletes-to-escalate-privilege-and-other-great-tricks).

#### Vulnerable versions and pre-requirements

All testing was done on versions 13.34-14.12.
I don’t know exactly lowest version that is vulnerable, but I believe that versions 12.0 - 14.12 are vulnerable as ClipboardChange event was introduced in version 12.0.
In order to exploit this vulnerability events that use ArchiveDirectory should not be enabled (ClipboardChange and FileDelete I believe) as if those two are used then ArchiveDirectory will be created and have secure permissions.

### Workaround

If you are using vulnerable version and cannot update you can create ArchiveDirectory (C:\Sysmon by default) and set permissions that will only allow access to NT AUTHORITY\SYSTEM account.

##### Timeline

- 2022/06/13 - Vulnerability reported to Microsoft
- 2022/06/16 - Vulnerability confirmed.
- 2022/11/08 - Patch and CVE released.
- 2022/11/08 - Bypass reported to Microsoft.
- 2022/11/11 - Microsoft cannot reproduce vulnerability, asks for different PoC.
- 2022/11/11 - I send same PoC and suggest that sysmon is either not installed on testing VM or installation was corrupted.
- 2022/11/15 - Microsoft confirmed bypass.
- 2022/11/28 - Microsoft release v14.13 that patched vulnerabilty (CVE will be released in December Patch Tuesday)

#### Links & Resources
[Community](https://www.zerodayinitiative.com/blog/2022/3/16/abusing-arbitrary-file-deletes-to-escalate-privilege-and-other-great-tricks)

### Table of Contents

- [Basics](https://www.coursera.org/courses?query=ethical%20hacking)
- [Information Gathering](https://www.coursera.org/courses?query=ethical%20hacking)
- [Vulnerability Analysis](https://www.coursera.org/courses?query=ethical%20hacking)
- [Web Application Analysis](https://www.coursera.org/courses?query=ethical%20hacking)
- [Database Assessment](https://www.coursera.org/courses?query=ethical%20hacking)
- [Password Attacks](https://www.coursera.org/courses?query=ethical%20hacking)
- [Exploitation Tools](https://www.coursera.org/courses?query=ethical%20hacking)
- [Post Exploitation](https://www.coursera.org/courses?query=ethical%20hacking)
- [Exploit Databases](https://www.coursera.org/courses?query=ethical%20hacking)
- [Payloads](https://www.coursera.org/courses?query=ethical%20hacking)
- [Wordlists](https://www.coursera.org/courses?query=ethical%20hacking)
- [Reporting](https://www.coursera.org/courses?query=ethical%20hacking)
- [Social Media Resources](https://www.coursera.org/courses?query=ethical%20hacking)
- [Commands](https://www.coursera.org/courses?query=ethical%20hacking)
- [Basics](https://www.coursera.org/courses?query=ethical%20hacking)
- [curl](https://www.coursera.org/courses?query=ethical%20hacking)
- [Chisel](https://www.coursera.org/courses?query=ethical%20hacking)
- [File Transfer](https://www.coursera.org/courses?query=ethical%20hacking)
- [Kerberos](https://www.coursera.org/courses?query=ethical%20hacking)
