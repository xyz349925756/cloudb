windows变量

通过环境变量%USERPROFILE%就可以获取了

在cmd下输入echo %USERPROFILE%可以查看当前系统用户目录

这边列举一些其他常用的获取系统路径的：

%WINDIR%                 {系统目录 - C:\WINDOWS}

%SYSTEMROOT%             {系统目录 - C:\WINDOWS}

%SYSTEMDRIVE%            {系统根目录 - C:}

%HOMEDRIVE%              {当前用户根目录 - C:}

%USERPROFILE%            {当前用户目录 - C:\Documents and Settings\wy}

%HOMEPATH%               {当前用户路径 - \Documents and Settings\wy}

%TMP%                    {当前用户临时文件夹 - C:\DOCUME\~1\wy\LOCALS~1\Temp}

%TEMP%                   {当前用户临时文件夹 - C:\DOCUME\~1\wy\LOCALS~1\Temp}

%APPDATA%                {当前用户数据文件夹 - C:\Documents and Settings\wy\Application Data}

%PROGRAMFILES%           {程序默认安装目录 - C:\Program Files}

%COMMONPROGRAMFILES%     {文件通用目录 - C:\Program Files\Common Files}

%USERNAME%               {当前用户名 - wy}

%ALLUSERSPROFILE%        {所有用户文件目录 - C:\Documents and Settings\All Users}

%OS%                     {操作系统名 - Windows_NT}

%COMPUTERNAME%           {计算机名 - IBM-B63851E95C9}

%NUMBER_OF_PROCESSORS%   {处理器个数 - 1}

%PROCESSOR_ARCHITECTURE% {处理器芯片架构 - x86}

%PROCESSOR_LEVEL%        {处理器型号 - 6}

%PROCESSOR_REVISION%     {处理器修订号 - 0905}

%USERDOMAIN%             {包含用户帐号的域 - IBM-B63851E95C9}

%PATH%    {搜索路径

检测程序是否启动，并重启
（1）goto命令执行循环

（2）tasklist|findstr -i "mysqld.exe"

检测进程是否存在

（3）start "" "D:/a_wnmps/webserver/mysql/bin/mysqld.exe"

执行命令,后面的第一个"",用来允许第二个"..."中有空格 

（4）执行不同目录下的bat，两步

1.先 cd /d "D:\a_wnmps\webserver\nginx"    // 跳转目录
2.再 Call start_nginx.bat                                // 执行bat文件

最后，cd /d "D:\a_file"  // 返回原来的目录，不然命令行上没东西

（5）choice /t 300 /d y /n > nul

延时，/t后面的300指300秒，也可以用：

ping -n 300 127.0.0.1 >nul   延时，-n后面的300指300秒
