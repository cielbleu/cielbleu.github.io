---
title: "소소한 Windows Server 자동화 스크립트"
description: "WIndow Server 관리 자동화를 위한 소소한 스크립트 정리"
category: Windows Server
tags: [Windows Server, Command, 스크립트]
toc: true
toc_sticky: true
---

Windows Server를 관리하면서 반드시 필요하지만, 매번 신경 쓰기에는 귀찮은 몇 가지 소소한 사항을 자동으로 처리하기 위해 만든 스크립트 모음입니다.  

![Windows Server Logo](/assets/images/windows_server_logo.svg)



# Windows Server 자동화 스크립트  

Windows Server를 관리하다 보면 반드시 필요하지만, 매번 신경 쓰기 귀찮은 몇 가지가 있습니다.  
예를 들면 다음과 같은 것들이 있습니다.  
- 백업한 파일을 압축하여 로컬 서버에 일정 기간 보관  
- 일정 기간이 지난 백업 또는 로그 삭제  
- 로컬 서버의 파일을 원격지 서버에 전송  
- 작업 결과를 관리자에게 메일로 발송  

대규모로 서버를 운영하는 기업에서는 전문 모니터링 도구 및 관리 소프트웨어를 사용할 수밖에 없지만, 관리할 서버가 많지 않을 때는 스크립트를 사용하는 것도 하나의 방법입니다.  
이 포스트에서는 소소한 관리업무를 자동으로 처리하기 위한 스크립트 몇 가지를 간략히 정리해 보았습니다.  

실무에서는 되도록 전문 모니터링 도구 및 관리 소프트웨어를 사용하고 부족한 부분을 보완하는 용도로만 사용하는 것이 좋습니다.  



## 1. 로컬 서버에 DB 백업 파일을 압축하여 보관하는 스크립트  

이 스크립트는 데이터베이스에서 백업한 전체백업 및 차등백업 파일을 압축하여 로컬 서버의 별도 폴더에 보관하는 스크립트입니다.  
데이터베이스에서 전체백업은 일주일에 1회, 차등백업은 매일 1회 백업하도록 설정했으며, 각 백업 파일은 1개만 보관되도록 설정했습니다.  
그리고 이 스크립트를 사용해 매일 새벽에 로컬 서버의 별도 폴더에 압축하여 보관하도록 설정했습니다.  
-전체백업
```cmd
@echo off
setlocal

echo Backup2Archive batch job has started.
echo Please do not close the Command Line Window until the operation is complete.

:: Split the date string(year/month/day)
set YEAR=%date:~10,4%
set MONTH=%date:~4,2%
set DAY=%date:~7,2%

:: Split the time string(hours/minutes/seconds)
set H=%time:~0,2%
set M=%time:~3,2%
set S=%time:~6,2%

set /a H=%H%+100
set H=%H:~1,2%

echo Start time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"

"C:\Program Files\7-Zip\7z.exe" a -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup.zip" "D:\SQL_Backup\Full_Backup\*.bak"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"
"C:\Program Files\7-Zip\7z.exe" t -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup.zip"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"

:: Split the time string(hours/minutes/seconds)
set H=%time:~0,2%
set M=%time:~3,2%
set S=%time:~6,2%

set /a H=%H%+100
set H=%H:~1,2%

echo End time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"

exit
```

-차등백업
```cmd
@echo off
setlocal

echo Backup2Archive batch job has started.
echo Please do not close the Command Line Window until the operation is complete.

:: Split the date string(year/month/day)
set YEAR=%date:~10,4%
set MONTH=%date:~4,2%
set DAY=%date:~7,2%

:: Split the time string(hours/minutes/seconds)
set H=%time:~0,2%
set M=%time:~3,2%
set S=%time:~6,2%

set /a H=%H%+100
set H=%H:~1,2%

echo Start time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Diff_Backup_Log.txt"

"C:\Program Files\7-Zip\7z.exe" a -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Diff_Backup.zip" "D:\SQL_Backup\Diff_Backup\*.bak"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Diff_Backup_Log.txt"
"C:\Program Files\7-Zip\7z.exe" t -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Diff_Backup.zip"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Diff_Backup_Log.txt"

:: Split the time string(hours/minutes/seconds)
set H=%time:~0,2%
set M=%time:~3,2%
set S=%time:~6,2%

set /a H=%H%+100
set H=%H:~1,2%

echo End time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Diff_Backup_Log.txt"

exit
```

**@echo off**  
스크립트 실행시 스크립트 파일 내의 명령들을 쓸데없이 '복창'하지 않도록 하는 명령어입니다.  
`@echo off` 명령어가 없으면 스크립트 내에서 명령어들이 실행될 때마다 cmd 화면에 출력되기에 화면이 지저분해집니다.  
`echo off` 자체도 하나의 명령이기 때문에 실행하면 화면에 출력됩니다.  
이를 방지하기 위해 앞에 `@`를 붙여줍니다.  
*.bat 또는 *.cmd에서는 명령어 앞에 `@`를 붙이면 그 명령어에 대해서는 '복창'하지 말라는 의미입니다.  

**setlocal**  
이 명령어는 스크립트 내의 변수가 스크립트 내에서만 영향을 주도록 제한하고 스크립트가 종료되면 변수를 지우는 명령어입니다.  
`setlocal`이 없다면 스크립트가 종료된 이후에도 환경변수로 남아 있게 되며, 만약 기존 환경변수와 동일한 변수명을 사용했을 경우 기존 환경변수를 덮어쓰게 됩니다.  
예를 들면 `setlocal` 명령 없이 set PATH=D:\Downloads라고 사용할 경우 기존 `PATH` 환경변수를 덮어쓰게 되어 시스템 운용에 문제가 될 수 있습니다.  

**set YEAR=%date:~10,4%**  
**set MONTH=%date:~4,2%**  
**set DAY=%date:~7,2%**  
`%DATE%` 환경변수의 값을 나눠 각각 년/월/일 변수로 설정하는 명령어입니다.  
한글판 OS를 사용하신다면 그냥 `%DATE%` 환경변수를 사용해도 됩니다.  
이 스크립트에서 년/월/일을 각각 변수로 나눈 이유는 영문판 OS에서 사용하기 위한 목적으로 작성했기 때문입니다.  
-한글판 OS
```cmd
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\MyAcccount>echo %date%
2018-09-06
```

-영문판 OS
```cmd
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\MyAcccount>echo %date%
Tue 09/06/2018
```
`set YEAR=%date:~10,4%`는 YEAR 환경변수에 `%DATE%` 환경변수의 값 중 ‘10’번째 위치부터 4개 문자에 해당하는 값을 설정하라는 의미입니다.  
참고로 첫 번째 위치는 '0'입니다.  
이 스크립트는 영문판 OS 환경으로 작성한 것이므로 한글판 OS에서 사용하기 위해서는 이 부분을 아래와 같이 변경하여야 합니다.  
```cmd
set YEAR=%date:~0,4%
set MONTH=%date:~5,2%
set DAY=%date:~8,2%
```

**set H=%time:~0,2%**  
**set M=%time:~3,2%**  
**set S=%time:~6,2%**  
`%TIME%` 환경변수의 값을 나눠 각각 시/분/초 변수로 설정하는 명령어입니다.  
`%DATE%` 환경변수와 다르게 영문판 OS와 한글판 OS에서 출력되는 내용이 동일합니다.  

**set /a H=%H%+100**  
**set H=%H:~1,2%**  
`set H=%time:~0,2%`로 시간 값을 구하면 오전 2시와 오후 2시는 각각 2과 14로 표시됩니다.  
자릿수가 맞지 않기 때문에 보기에 좋지 않습니다.  
시간 값을 모두 두자리로 표현하기 위해 `set H=%time:~0,2%`로 구한 시간 값에 100을 더한 후 ‘0’번째 위치부터 2개의 문자를 가져와서 시간 값으로 다시 설정했습니다.  
`set /a` 명령어를 사용해 간단한 계산을 할 수 있으나 결과가 그다지 정밀하지는 않으므로 간단한 계산에만 사용하는 것이 좋습니다.  

**echo Start time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"**  
스크립트 시작시간을 파일에 입력하는 명령어입니다.  
`%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt` 파일은 스크립트의 실행 결과를 저장하는 파일이며, 이후 관리자에게 메일로 전송하기 위해 사용합니다.  

**"C:\Program Files\7-Zip\7z.exe" a -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup.zip" "D:\SQL_Backup\Full_Backup\*.bak"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"**  
스크립트의 실제 목적을 실행하는 명령어입니.  
[7zip](https://www.7-zip.org)을 이용하여 '"D:\SQL_Backup\Full_Backup\*.bak"' 위치에 있는 bak 확장자를 갖는 모든 파일을 '"D:\Archive"' 위치에 '%YEAR%-%MONTH%-%DAY%_Full_Backup.zip'라는 파일로 압축하여 저장하라는 의미입니다.  