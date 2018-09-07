---
title: "소소한 Windows Server 자동화 스크립트 Vol.1"
description: "Window Server 관리 자동화를 위한 소소한 스크립트 정리 Vol.1"
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
이 포스트에서는 소소한 관리업무를 자동으로 처리하기 위한 스크립트 중 첫 번째로 DB 백업 파일을 압축해 로컬 서버에 보관하는 스크립트를 정리해 보았습니다.  
실무에서는 되도록 전문 모니터링 도구 및 관리 소프트웨어를 사용하고 부족한 부분을 보완하는 용도로만 사용하는 것이 좋습니다.  



## 로컬 서버에 DB 백업 파일을 압축하여 보관하는 스크립트  

이 스크립트는 데이터베이스에서 백업한 전체 백업 및 차등 백업 파일을 압축하여 로컬 서버의 별도 폴더에 보관하는 스크립트입니다.  
데이터베이스에서 전체 백업은 일주일에 1회, 차등 백업은 매일 1회 백업하도록 설정했으며, 각 백업 파일은 1개만 보관되도록 설정했습니다.  
그리고 이 스크립트를 사용해 매일 새벽에 로컬 서버의 별도 폴더에 압축하여 보관하도록 설정했습니다.  

**전체 백업**  
```bash
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

**차등 백업**  
```bash
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

### 구문 분석  

**@echo off**  
스크립트가 실행될 때 스크립트 파일 내의 명령들을 쓸데없이 '복창'하지 않도록 하는 명령어입니다.  
`@echo off` 명령어가 없으면 스크립트 내에서 명령어들이 실행될 때마다 cmd 화면에 출력되기에 화면이 지저분해집니다.  
`echo off` 자체도 하나의 명령이기 때문에 실행하면 화면에 출력됩니다.  
이를 방지하기 위해 앞에 `@`를 붙여줍니다.  
*.bat 또는 *.cmd에서는 명령어 앞에 `@`를 붙이면 그 명령어에 대해서는 '복창'하지 말라는 의미입니다.  

**setlocal**  
이 명령어는 스크립트 내의 변수가 스크립트 내에서만 영향을 주도록 제한하고 스크립트가 종료되면 변수를 지우는 명령어입니다.  
`setlocal`이 없다면 스크립트가 종료된 이후에도 환경변수로 남아 있게 되며, 만약 기존 환경변수와 같은 변수명을 사용했을 경우 기존 환경변수를 덮어쓰게 됩니다.  
예를 들면 `setlocal` 명령 없이 `set PATH=D:\Downloads`라고 사용할 경우 기존 `PATH` 환경변수를 덮어쓰게 되어 시스템 운용에 문제가 될 수 있습니다.  

**set YEAR=%date:~10,4%**  
**set MONTH=%date:~4,2%**  
**set DAY=%date:~7,2%**  
`%DATE%` 환경변수의 값을 나눠 각각 년/월/일 변수로 설정하는 명령어입니다.  
`%DATE%` 환경변수는 한글판 OS와 영문판 OS에서 출력되는 형식이 서로 다릅니다.  

**한글판 OS**  
```bash
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\MyAcccount>echo %date%
2018-09-06
```

**영문판 OS**  
```bash
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\Users\MyAcccount>echo %date%
Tue 09/06/2018
```

위의 샘플처럼 영문판 OS에서 출력되는 형식은 스크립트에서 사용하고자 하는 형식과 다르기 때문에 이를 각각의 변수로 나눠줄 필요가 있습니다.  
`set YEAR=%date:~10,4%`는 `%DATE%` 환경변수의 값 중 ‘10’번째 위치부터 4개 문자에 해당하는 값을 YEAR 환경변수에 설정하라는 의미입니다.  
참고로 첫 번째 위치는 '0'부터 시작합니다.  
이 스크립트는 영문판 OS 환경으로 작성한 것이므로 한글판 OS에서 사용하기 위해서는 이 부분을 아래와 같이 변경하여야 합니다.  

```bash
set YEAR=%date:~0,4%
set MONTH=%date:~5,2%
set DAY=%date:~8,2%
```
하지만 한글판 OS에서는 년/월/일을 변수로 나눌 필요 없이 그냥 `%DATE%` 환경변수를 사용해도 됩니다.  

**set H=%time:~0,2%**  
**set M=%time:~3,2%**  
**set S=%time:~6,2%**  
`%TIME%` 환경변수의 값을 나눠 각각 시/분/초 변수로 설정하는 명령어입니다.  
`%DATE%` 환경변수와 다르게 영문판 OS와 한글판 OS에서 출력되는 내용이 같습니다.  

**set /a H=%H%+100**  
**set H=%H:~1,2%**  
`set H=%time:~0,2%`로 시간 값을 구하면 오전 2시와 오후 2시는 각각 2과 14로 표시됩니다.  
사용하는 데 지장은 없으나 자릿수가 맞지 않기 때문에 미관상 좋지 않습니다. 시간 값을 모두 두 자리로 맞춰 주는 것이 좋겠습니다.  
시간 값을 모두 두 자리로 표현하기 위해 `set H=%time:~0,2%`로 구한 시간 값에 100을 더한 후 ‘1’번째 위치부터 2개의 문자를 가져와서 시간 값으로 다시 설정했습니다.  
`set /a` 명령어를 사용해 간단한 계산을 할 수 있으나 결과가 그다지 정밀하지는 않으므로 간단한 계산에만 사용하는 것이 좋습니다.  

**echo Start time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"**  
스크립트 시작 시각을 파일에 입력하는 명령어입니다.  
`%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt` 파일은 스크립트의 실행 결과를 저장하는 파일이며, 이후 관리자에게 메일로 전송하기 위해 사용합니다.  

**"C:\Program Files\7-Zip\7z.exe" a -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup.zip" "D:\SQL_Backup\Full_Backup\*.bak"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"**  
스크립트의 실제 목적인 DB 백업 파일을 압축하는 명령어입니다.  
[7zip](https://www.7-zip.org)을 이용하여 `"D:\SQL_Backup\Full_Backup\*.bak"` 위치에 있는 bak 확장자를 갖는 모든 파일을 `"D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup.zip"`라는 파일로 압축하여 저장하라는 의미입니다.  
그리고 그 결과를 `"D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"` 파일에 저장하라는 의미입니다.  

첫 번째 Argument `a`는 압축파일에 파일을 추가하는 명령어입니다.  
사용법은 아래와 같습니다.  
>Example  
>7z a archive1.zip subdir\  
>[출처](https://sevenzip.osdn.jp/chm/cmdline/commands/index.htm)  

`-tzip` 은 압축 유형을 지정하는 스위치입니다.  
문법은 아래와 같습니다. 본 포스트에서는 유형만 zip으로 지정하였습니다.  
>-t{archive_type}[:s{Size}][:r][:e][:a]  
>[출처](https://sevenzip.osdn.jp/chm/cmdline/switches/index.htm)  

**"C:\Program Files\7-Zip\7z.exe" t -tzip "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup.zip"  >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"**  
압축한 DB 백업 파일에 이상이 있는지 테스트하고 그 결과를 `"D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"` 파일에 저장하라는 명령어입니다.  

첫 번째 Argument `t`는 압축파일을 테스트하는 명령어입니다.  
사용법은 아래와 같습니다.  
>Example  
>7z t archive.zip  
>[출처](https://sevenzip.osdn.jp/chm/cmdline/commands/index.htm)  

**echo End time : %YEAR%-%MONTH%-%DAY% %H%:%M%:%S% >> "D:\Archive\%YEAR%-%MONTH%-%DAY%_Full_Backup_Log.txt"**  
스크립트 종료 시각을 파일에 입력하는 명령어입니다.  

**exit**  
스크립트 종료 후 Command Window를 종료하는 명령어입니다.  

### 작업 스케줄러 등록  

스크립트를 생성하였으면 서버가 한가한 시간에 자동으로 실행되도록 작업 스케줄러에 등록할 필요가 있습니다.  

1. `Server Manager > Tools > Task Scheduler`를 실행하여 `작업 만들기`를 클릭합니다.  
![Task Scheduler #1](/assets/images/task_scheduler_1.png)

2. 아래 이미지와 같이 설정합니다.  
샘플 이미지에서는 계정 정보를 지웠습니다. `사용자 또는 그룹 변경` 버튼을 클릭하여 자신의 계정을 선택하세요.  
보통은 기본적으로 자신의 계정이 선택되어 있습니다.  
![Task Scheduler #2](/assets/images/task_scheduler_2.png)

3. `트리커` 탭을 선택하고 `새로 만들기` 버튼을 클릭합니다.  
그리고 아래 이미지와 같이 설정합니다.  
아래 예시 이미지는 전체 백업용 작업 스케줄러이며 매주 일요일 새벽 1시 30분에 실행되도록 설정하였습니다.  
차등 백업용으로 작업 스케줄러는 매일 새벽 2시 30분에 실행되도록 설정하였습니다.  
![Task Scheduler #3](/assets/images/task_scheduler_3.png)

4. `동작` 탭을 선택하고 `새로 만들기` 버튼을 클릭합니다.  
그리고 아래 이미지와 같이 설정합니다.  
`트리커` 탭에서 설정한 시각이 되면 등록한 스크립트를 실행하라는 의미입니다.  
![Task Scheduler #4](/assets/images/task_scheduler_4.png)

5. `설정` 탭을 선택하고 아래 이미지와 같이 설정합니다.  
![Task Scheduler #5](/assets/images/task_scheduler_5.png)
