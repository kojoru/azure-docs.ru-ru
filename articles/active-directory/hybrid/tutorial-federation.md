---
title: Учебник. Федерация среды одного леса AD с Azure | Документы Майкрософт
description: Демонстрируется настройка гибридной среды идентификации с помощью федерации.
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/16/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 2a869e8d504e4f3557caa7925b89e8c184718af9
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306083"
---
# <a name="tutorial-federate-a-single-ad-forest-environment-to-the-cloud"></a>Учебник. Федерация среды одного леса AD с облаком

![Создание](media/tutorial-federation/diagram.png)

В этом руководстве последовательно рассматривается создание гибридной среды идентификации с помощью синхронизации.  Затем эту среду можно использовать для тестирования или дальнейшего знакомства с принципами работы гибридной идентификации.

## <a name="prerequisites"></a>Предварительные требования
Для работы с этим учебником требуется следующее:
- Компьютер с установленным [Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/hyper-v-technology-overview).  Рекомендуется использовать компьютер с ОС [Windows 10](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/supported-guest-os) или [Windows Server 2016](https://docs.microsoft.com/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).
- [Подписка Azure](https://azure.microsoft.com/free)
- - [Внешний сетевой адаптер](https://docs.microsoft.com/virtualization/hyper-v-on-windows/quick-start/connect-to-network) для связи виртуальной машины с Интернетом.
- Копия Windows Server 2016.
- [Личный домен](../../active-directory/fundamentals/add-custom-domain.md), который можно проверить.

> [!NOTE]
> В этом учебнике используются скрипты PowerShell, что позволяет создать учебную среду в минимальные сроки.  В каждом скрипте применяются переменные, которые объявляются в начале скрипта.  Эти переменные можно и нужно изменить в соответствии с особенностями вашей среды.
>
>Скрипты, используемые для создания общей среды Active Directory перед установкой Azure AD Connect.  Они актуальны для всех руководств.
>
> Копии скриптов PowerShell, используемых в этом учебнике, можно найти в Github [здесь](https://github.com/billmath/tutorial-phs).

## <a name="create-a-virtual-machine"></a>Создание виртуальной машины
Первое, что нужно сделать, чтобы подготовить среду гибридной идентификации к работе, — создать виртуальную машину, которая будет служить локальным сервером Active Directory.  

>[!NOTE]
>Если вы никогда не запускали скрипт в PowerShell на компьютере узла, необходимо выполнить команду `Set-ExecutionPolicy remotesigned` и ответить "да" в PowerShell перед запуском скриптов.

Выполните следующее:

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

```powershell
#Declare variables
$VMName = 'DC1'
$Switch = 'External'
$InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
$Path = 'D:\VM'
$VHDPath = 'D:\VM\DC1\DC1.vhdx'
$VHDSize = '64424509440'

#Create New Virtual Machine
New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

#Set the memory to be non-dynamic
Set-VMMemory $VMName -DynamicMemoryEnabled $false

#Add DVD Drive to Virtual Machine
Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

#Mount Installation Media
$DVDDrive = Get-VMDvdDrive -VMName $VMName

#Configure Virtual Machine to Boot from DVD
Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive 
```

## <a name="complete-the-operating-system-deployment"></a>Завершение развертывания операционной системы
Чтобы завершить создание виртуальной машины, необходимо завершить установку операционной системы.

1. В диспетчере Hyper-V дважды щелкните виртуальную машину.
2. Нажмите кнопку "Запустить".
3.  Вам будет предложено нажать любую клавишу для загрузки с компакт-диска или DVD-диска. Сделайте это.
4. На начальном экране Windows Server выберите язык и нажмите кнопку **Далее**.
5. Нажмите **Установить**.
6. Введите ключ лицензии и нажмите кнопку **Далее**.
7. Установите флажок "Я принимаю условия лицензии" и нажмите кнопку **Далее**.
8. Выберите вариант **Пользовательская: установить только Windows (расширенная)**
9. Щелкните **Далее**
10. Когда установка завершится, перезапустите виртуальную машину, выполните вход и запустите обновление Windows, чтобы обеспечить актуальность виртуальной машины.  Установите последние обновления.

## <a name="install-active-directory-pre-requisites"></a>Установка необходимых компонентов для Active Directory
Создав виртуальную машину, необходимо выполнить ряд действий перед установкой Active Directory.  В частности, нужно переименовать виртуальную машину, задать статический IP-адрес и сведения DNS, а также установить средства удаленного администрирования сервера.   Выполните следующее:

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните команду `Set-ExecutionPolicy remotesigned` и ответьте "да для всех" [A].  Нажмите клавишу ВВОД.
3. Выполните следующий сценарий.

```powershell
#Declare variables
$ipaddress = "10.0.1.117" 
$ipprefix = "24" 
$ipgw = "10.0.1.1" 
$ipdns = "10.0.1.117"
$ipdns2 = "8.8.8.8" 
$ipif = (Get-NetAdapter).ifIndex 
$featureLogPath = "c:\poshlog\featurelog.txt" 
$newname = "DC1"
$addsTools = "RSAT-AD-Tools" 

#Set static IP address
New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

# Set the DNS servers
Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

#Rename the computer 
Rename-Computer -NewName $newname -force 

#Install features 
New-Item $featureLogPath -ItemType file -Force 
Add-WindowsFeature $addsTools 
Get-WindowsFeature | Where installed >>$featureLogPath 

#Restart the computer 
Restart-Computer
```

## <a name="create-a-windows-server-ad-environment"></a>Создание среды AD для Windows Server
Создав виртуальную машину, переименовав ее и назначив ей статический IP-адрес, мы можем установить и настроить доменные службы Active Directory.  Выполните следующее:

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

```powershell 
#Declare variables
$DatabasePath = "c:\windows\NTDS"
$DomainMode = "WinThreshold"
$DomainName = "contoso.com"
$DomaninNetBIOSName = "CONTOSO"
$ForestMode = "WinThreshold"
$LogPath = "c:\windows\NTDS"
$SysVolPath = "c:\windows\SYSVOL"
$featureLogPath = "c:\poshlog\featurelog.txt" 
$Password = "Pass1w0rd"

#Install AD DS, DNS and GPMC 
start-job -Name addFeature -ScriptBlock { 
Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
Wait-Job -Name addFeature 
Get-WindowsFeature | Where installed >>$featureLogPath

#Create New AD Forest
Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $Password -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
```

## <a name="create-a-windows-server-ad-user"></a>Создание пользователя AD для Windows Server
Теперь, когда среда Active Directory создана, нам требуется тестовая учетная запись.  Она будет создана в локальной среде AD, а затем синхронизирована с Azure AD.  Выполните следующее:

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

```powershell 
#Declare variables
$Givenname = "Allie"
$Surname = "McCray"
$Displayname = "Allie McCray"
$Name = "amccray"
$Password = "Pass1w0rd"
$Identity = "CN=ammccray,CN=Users,DC=contoso,DC=com"
$SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


#Create the user
New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

#Set the password to never expire
Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
```

## <a name="create-a-certificate-for-ad-fs"></a>Создание сертификата для AD FS
Теперь мы создадим SSL-сертификат, который будет использоваться службами AD FS.  Это будет самозаверяющий сертификат только для тестирования.  Корпорация Майкрософт не рекомендует использовать самозаверяющий сертификат в рабочей среде. Выполните следующее:

1. Откройте интегрированную среду сценариев PowerShell от имени администратора.
2. Выполните следующий сценарий.

```powershell 
#Declare variables
$DNSname = "adfs.contoso.com"
$Location = "cert:\LocalMachine\My"

#Create certificate
New-SelfSignedCertificate -DnsName $DNSname -CertStoreLocation $Location
```

## <a name="create-an-azure-ad-tenant"></a>Создание клиента Azure AD
Теперь нужно создать клиент Azure AD, чтобы синхронизировать пользователей с облаком.  Чтобы создать клиент Azure AD, выполните указанные ниже действия.

1. Перейдите на [портал Azure](https://portal.azure.com) и выполните вход с учетной записью, имеющей подписку Azure.
2. Щелкните **значок "плюс" (+)** и выполните поиск по запросу **Azure Active Directory**.
3. В результатах поиска выберите **Azure Active Directory**.
4. Нажмите кнопку **Создать**.</br>
![Создание](media/tutorial-password-hash-sync/create1.png)</br>
5. Укажите **имя организации** с **первоначальным доменным именем**. Щелкните **Создать**. В результате будет создан каталог.
6. По завершении щелкните ссылку **здесь**, чтобы перейти к управлению каталогом.

## <a name="create-a-global-administrator-in-azure-ad"></a>Создание глобального администратора в Azure AD
Теперь, когда у нас есть клиент Azure AD, мы создадим учетную запись глобального администратора.  Она используется для создания учетной записи соединителя Azure AD во время установки Azure AD Connect.  Учетная запись соединителя Azure AD служит для записи сведений в Azure AD.   Чтобы создать учетную запись глобального администратора, выполните указанные ниже действия.

1.  В разделе **Управление** выберите **Пользователи**.</br>
![Создание](media/tutorial-password-hash-sync/gadmin1.png)</br>
2.  Выберите пункт **Все пользователи**, а затем выберите **+ Новый пользователь**.
3.  Укажите имя и имя пользователя. Этот пользователь будет глобальным администратором клиента. Кроме того, может потребоваться изменить **роль каталога** на **Глобальный администратор**. Можно также отобразить временный пароль. Когда все будет готово, нажмите кнопку **Создать**.</br>
![Создание](media/tutorial-password-hash-sync/gadmin2.png)</br>
4. По завершении откройте новую вкладку в браузере и выполните вход на странице myapps.microsoft.com, используя новую учетную запись глобального администратора и временный пароль.
5. Смените пароль глобального администратора и запомните новый пароль.

## <a name="add-the-custom-domain-name-to-your-directory"></a>Добавьте имя личного домена в каталог.
Теперь, когда у нас есть клиент и глобальный администратор, необходимо добавить личный домен, чтобы платформа Azure могла проверить его.  Выполните следующее:

1. Вернувшись на [портал Azure](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview), закройте колонку **Все пользователи**.
2. В левой части экрана щелкните **Имена пользовательских доменов**.
3. Нажмите кнопку **Добавить личный домен**.</br>
![Федерация](media/tutorial-federation/custom1.png)</br>
4. В колонке **Имена личных доменов** введите имя личного домена в поле, а затем щелкните **Добавить домен**.
5. На экране "Имя пользовательского домена" будут предоставлены сведения о записях TXT или MX.  Эти сведения следует добавить в информацию DNS на сайте регистратора вашего домена.  То есть вам нужно перейти на сайт своего регистратора доменных имен и ввести сведения о записях TXT или MX в параметрах DNS вашего домена.  Это позволит Azure проверить ваш домен.  На проверку может потребоваться до 24 часов.  Дополнительные сведения см. в документе о [добавлении личного домена](../../active-directory/fundamentals/add-custom-domain.md).</br>
![Федерация](media/tutorial-federation/custom2.png)</br>
6. Чтобы узнать, проверен ли домен, нажмите кнопку "Проверить".</br>
![Федерация](media/tutorial-federation/custom3.png)</br>

## <a name="download-and-install-azure-ad-connect"></a>Скачивание и установка Azure AD Connect
Теперь пора скачать и установить Azure AD Connect.  После скачивания мы произведем экспресс-установку.  Выполните следующее:

1. Скачайте [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).
2. Перейдите к файлу **AzureADConnect.msi**и дважды щелкните его.
3. На экране приветствия установите флажок, подтверждающий ваше согласие с условиями лицензионного соглашения, и нажмите кнопку **Продолжить**.  
4. На экране "Стандартные параметры" щелкните **Настроить**.  
5. На экране "Установка обязательных компонентов" выполните указанные ниже действия. Щелкните **Install**(Установить).  
6. На экране входа пользователя выберите **Федерация с AD FS** и нажмите кнопку **Далее**.
![Федерация](media/tutorial-federation/fed1.png)

1. На экране "Подключение к Azure AD" введите имя пользователя и пароль глобального администратора, созданного ранее, и нажмите кнопку **Далее**.
2. На экране "Подключить каталоги" щелкните **Добавить каталог**.  Затем выберите **Создать учетную запись AD**, введите имя пользователя и пароль в формате "contoso\Администратор" и нажмите кнопку **ОК**.
3. Щелкните **Далее**.
4. На экране "Настройка входа в Azure AD" выберите **Продолжить без сопоставления всех суффиксов имени субъекта-пользователя с проверенными доменами** и нажмите кнопку **Далее**.
5. На экране "Фильтрация доменов и подразделений" нажмите кнопку **Далее**.
6. На экране "Уникальная идентификация пользователей" нажмите кнопку **Далее**.
7. На экране "Фильтрация пользователей и устройств" нажмите кнопку **Далее**.
8. На экране "Дополнительные возможности" нажмите кнопку **Далее**.
9. На странице "Учетные данные администратора домена" введите имя пользователя и пароль в формате "contoso\Администратор" и нажмите кнопку **Далее**.
10. На экране "Ферма AD FS" должен быть выбран параметр **Настроить новую ферму AD FS**.
11. Выберите параметр **Использовать сертификат с серверов федерации** и нажмите кнопку **Обзор**.
12. В поле поиска введите "DC1" и выберите соответствующий результат.  Нажмите кнопку **ОК**.
13. В раскрывающемся списке **Файл сертификата** выберите сертификат **adfs.contoso.com**, созданный ранее.  Щелкните **Далее**.
![Федерация](media/tutorial-federation/fed2.png)

1. На экране "Сервер AD FS" нажмите кнопку **Обзор**, в поле поиска введите "DC1" и выберите соответствующий результат.  Нажмите кнопку **ОК**.  Щелкните **Далее**.
![Федерация](media/tutorial-federation/fed3.png)

1. На экране "Серверы прокси-службы веб-приложения" нажмите кнопку **Далее**.
2. На экране "Учетная запись службы AD FS" введите имя пользователя и пароль в формате "contoso\Администратор" и нажмите кнопку **Далее**.
3. На экране "Домен Azure AD" в раскрывающемся списке выберите проверенный личный домен и нажмите кнопку **Далее**.
4. На экране "Все готово к настройке" нажмите кнопку **Установить**.
5. По завершении установки щелкните **Выход**.
6. После завершения установки выполните выход и снова войдите, прежде чем начинать использовать Synchronization Service Manager или редактор правил синхронизации.


## <a name="verify-users-are-created-and-synchronization-is-occurring"></a>Проверка создания пользователей и выполнения синхронизации
Теперь мы проверим, были ли пользователи, имевшиеся в локальном каталоге, синхронизированы и добавлены в клиент Azure AD.  Имейте в виду, что эта операция может занять несколько часов.  Чтобы проверить синхронизацию пользователей, выполните указанные ниже действия.


1. Перейдите на [портал Azure](https://portal.azure.com) и выполните вход с учетной записью, имеющей подписку Azure.
2. Слева выберите **Azure Active Directory**.
3. В разделе **Управление** выберите **Пользователи**.
4. Убедитесь в том, что новый пользователь отображается на экране ![Синхронизация](media/tutorial-password-hash-sync/synch1.png) в клиенте.

## <a name="test-signing-in-with-one-of-our-users"></a>Проверка входа с помощью одной из учетных записей

1.  Перейдите на сайт [http://myapps.microsoft.com](http://myapps.microsoft.com).
2. Выполните вход с помощью учетной записи пользователя, которая была создана в новом клиенте.  Для этого следует использовать следующий формат: (user@domain.onmicrosoft.com). Используйте тот же пароль, что и для входа в локальную среду.
![Проверка](media/tutorial-password-hash-sync/verify1.png)

Вы успешно настроили среду гибридной идентификации, которую можно использовать для тестирования и ознакомления с возможностями Azure.

## <a name="next-steps"></a>Дальнейшие действия

- [Оборудование и предварительные требования](how-to-connect-install-prerequisites.md) 
- [Настраиваемые параметры](how-to-connect-install-custom.md)
- [Azure AD Connect и федерация](how-to-connect-fed-whatis.md)

