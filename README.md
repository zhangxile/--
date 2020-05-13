# 进程隐藏
##进程伪装
对病毒木马来说，最简单的进程伪装方式就是修改进程名。进程伪装指的是可以修改任意指定进程信息，即该进程信息再系统中显示的是另一个进程的信息。这样指定进程和伪装进程相同，但实际，执行的操作是不同的。
######先用OpenProcess把进程打开
######1.用快照函数CreateToolhelp32Snapshot，然后枚举进程，比较
        CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0)
        Process32First/Process32Next
######2.用GetModuleFileNameEx(当前进程用GetModuleFileName，多一个进程句柄参数)
        DWORD GetModuleFileNameEx(
          HANDLE hProcess,
          HMODULE hModule,
          LPTSTR lpFilename,
          DWORD nSize
        );
######3.用GetProcessImageFileName(WinXP以后新加的一个函数，在psapi.dll中)
        DWORD GetProcessImageFileName(
          HANDLE hProcess,
          LPTSTR lpImageFileName,
          DWORD nSize
        );

        
先用ObReferenceObjectByHandle把进程句柄转换成PEPROCESS，然后将PEPROCESS传递给SeLocateProcessImageName，SeLocateProcessImageName会判断SeAuditProcessCreationInfo成员是否已经初始化，没有则调用。PsReferenceProcessFilePointer获得进程对应的文件句柄，并将文件句柄传递。
SeInitializeProcessAuditName，SeInitializeProcessAuditName最后调用ObQueryNameString将文件句柄转换成文件名。在驱动中调用一次NtQueryInformationProcess，然后去修改EPROCESS中的SeAuditProcessCreationInfo成员完成进程名的伪装。 
##总结：
实现进程隐藏的方法有很多，例如：1.进程伪装
2.傀儡进程
3.进程隐藏
4.DLL劫持  。其中进程伪装相对容易。即获取相应进程的名字或路径，通过修改其PEB中的路径和命令行信息实现伪装。
