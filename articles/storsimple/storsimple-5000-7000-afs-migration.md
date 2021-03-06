---
title: Перенос данных с устройств StorSimple серий 5000–7000 в службу "Синхронизация файлов Azure" | Документация Майкрософт
description: В этой статье описывается, как переносить данные с устройств StorSimple серий 5000/7000 в службу "Синхронизация файлов Azure" (AFS).
services: storsimple
documentationcenter: NA
author: alkohli
manager: twooley
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/23/2018
ms.author: alkohli
ms.openlocfilehash: aad5b2d2e451f33a3f4f365fa0702eef5d698e4e
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/17/2018
ms.locfileid: "45729169"
---
# <a name="migrate-data-from-storsimple-5000-7000-series-to-azure-file-sync"></a>Перенос данных с устройств StorSimple серий 5000–7000 в службу "Синхронизация файлов Azure"

> [!IMPORTANT]
> 31 июля 2019 года поддержка устройств StorSimple серий 5000/7000 будет прекращена. Мы рекомендуем клиентам устройств StorSimple серий 5000 и 7000 перейти на один из альтернативных вариантов, описанных в этом документе.

Перенос данных — это перемещение данных из одного места хранения в другое. В результате создается точная копия текущих данных организации и перемещается с одного устройства на другое, желательно без нарушения работы или отключения активных приложений, а затем все операции ввода-вывода перенаправляются на новое устройство. 

Срок действия запоминающихся устройств StorSimple серий 5000 и 7000 истекает в июле 2019 года. Это означает, что после истечения срока действия корпорация Майкрософт больше не сможет поддерживать оборудование и программное обеспечение для устройств StorSimple серий 5000/7000. Клиенты, которые используют эти устройства, должны перенести данные StorSimple на другие гибридные решения хранения данных в Azure. В этой статье описывается перенос данных с устройства StorSimple серий 5000/7000 в службу "Синхронизация файлов Azure" (AFS).

## <a name="intended-audience"></a>Целевая аудитория

Эта статья предназначена для специалистов в области информационных технологий (ИТ) и информационных работников, ответственных за развертывание устройств StorSimple серий 5000/7000 и управление ими в центре обработки данных. Этот путь миграции особенно подходит для клиентов, использующих устройства StorSimple для рабочих нагрузок файлового сервера (с Windows Server). Если вы считаете, что функции службы "Синхронизация файлов Azure" подходят для вашей организации, данная статья поможет вам понять, как перейти на эти решения со StorSimple.

## <a name="migration-considerations"></a>Рекомендации по переносу

Этот процесс подходит для клиентов, которые настроили общий файловый ресурс Windows с использованием тома StorSimple для хранилища. Перенос данных с устройства StorSimple серий 5000/7000 в службу "Синхронизация файлов Azure" включает преобразование этого общего файлового ресурса в [конечную точку сервера](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning), а затем использование локально подключенных дисков в качестве другой конечной точки, которая позже станет вашим новым расположением. 

При перемещении в AFS следует учитывать следующие аспекты:

1. К файлам Azure в настоящее время применяется ограничение 5 ТБ на общий ресурс. Это ограничение можно преодолеть с помощью службы "Синхронизация файлов Azure", распределив данные по нескольким общим файловым ресурсам Azure. Дополнительные сведения см. в статье [Планирование развертывания службы файлов Azure](https://docs.microsoft.com/azure/storage/files/storage-files-planning).
2. Во время этого перемещения весь первичный набор данных загружается в локальное устройство, так как копия данных выполняется с локального устройства. Убедитесь, что у вас достаточно пропускной способности для этой операции перемещения.
3. Этот процесс не сохраняет уже созданные моментальные снимки. Переносятся только первичные данные. Этот процесс также не сохраняет связанные шаблоны пропускной способности или политики архивации. [С помощью службы Azure Backup](https://docs.microsoft.com/azure/backup/backup-azure-files) можно настроить политики архивации после переноса данных в общий файловый ресурс Azure.
4. Устройство StorSimple предоставляет основное оборудование. Однако при применении файлов Azure и службы "Синхронизация файлов Azure" вы используете свое локальное оборудование Windows Server как локальный кэш. Вы должны убедиться, что емкости вашего локального хранилища достаточно для хранения выбранного набора данных. Дополнительные сведения о распределении по уровням и установке целевого показателя требуемого свободного пространства см. в статье [Развертывание Синхронизации файлов Azure](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=portal). 
5. На странице [Цены на файлы Azure](https://azure.microsoft.com/pricing/details/storage/files/) просмотрите цену на службу "Синхронизация файлов Azure", так как она зависит от устройства StorSimple. AFS не поддерживает дедупликацию и сжатие, как StorSimple.

## <a name="migration-prerequisites"></a>Предварительные требования к миграции

Ниже перечислены требования к перемещению с устаревшего устройства серий 5000/7000 в службу "Синхронизация файлов Azure". Перед началом работы убедитесь, что у вас есть следующее:

- Доступ к устройству StorSimple серий 5000/7000 с версией программного обеспечения 2.1.1.518 или новее. Более ранние версии не поддерживаются. Текущая версия программного обеспечения должна отображаться в верхнем правом углу веб-интерфейса вашего устройства StorSimple.  
    Если версия отличается от 2.1.1.518, выполните обновление системы до минимальной требуемой версии. Подробные инструкции см. в статье [Software Patch Upgrade Guide v2.1.1.518](http://onlinehelp.storsimple.com/111_Appliance/6_System_Upgrade_Guides/Current_(v2.1.1)/000_Software_Patch_Upgrade_Guide_v2.1.1.518) (Руководство по обновлению программного обеспечения до версии 2.1.1.518).
- Проверьте наличие активных заданий резервного копирования, которые выполняются на исходном устройстве. Сюда входят задания на узле StorSimple Data Protection Console. Дождитесь завершения текущих заданий. 
- Получите доступ к узлу Windows Server, подключенному к вашему устройству StorSimple серий 5000–7000. Узел должен работать под управлением поддерживаемой версии Windows Server, как описано в статье [Планирование развертывания службы синхронизации файлов Azure](https://docs.microsoft.com/azure/storage/files/storage-sync-files-planning).
- Тома StorSimple подключаются на узле и содержат общие файловые ресурсы.
- В локальном хранилище узла достаточно места для хранения локально кэшированных данных.
- Права владельца для доступа к подписке Azure, которую вы будете использовать для развертывания службы "Синхронизация файлов Azure". При создании облачной конечной точки для группы синхронизации могут возникнуть проблемы, если у вас нет прав владельца или администратора.
- Получите доступ к [учетной записи хранения общего назначения версии 2](https://docs.microsoft.com/azure/storage/common/storage-account-overview) с общим файловым ресурсом Azure, с которым нужно выполнить синхронизацию. Дополнительные сведения см. в разделе [Создание учетной записи хранения](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account).
 - [Создание общей папки с помощью портала Azure](https://docs.microsoft.com/azure/storage/files/storage-how-to-create-file-share#create-file-share-through-the-azure-portal).

## <a name="migration-process"></a>Процесс миграции

Перемещение данных с устройства StorSimple серий 5000–7000 в AFS — это двухэтапный процесс:
1.  Реплицируйте данные с локального файлового сервера, в котором тома StorSimple подключены к общему файловому ресурсу Azure.  Репликация выполняется с помощью установленного вами агента AFS.
2.  Отключите устройство StorSimple. После этого локальные диски будут выступать в качестве локального кэша.

### <a name="migration-steps"></a>Этапы миграции

Выполните следующие шаги, чтобы перенести общий файловый Windows, настроенный в томах StorSimple, в общий ресурс службы "Синхронизация файлов Azure". 
1.  Выполните эти шаги в том же узле Windows Server, где подключены тома StorSimple, или используйте другую систему. 
    - [Подготовка сервера Windows к работе со службой "Синхронизация файлов Azure"](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#prepare-windows-server-to-use-with-azure-file-sync).
    - [Установка агента службы "Синхронизация файлов Azure"](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#install-the-azure-file-sync-agent).
    - [Развертывание службы синхронизации хранилища](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#deploy-the-storage-sync-service). 
    - [Регистрация Windows Server в службе синхронизации хранилища](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#register-windows-server-with-storage-sync-service). 
    - [Создание группы синхронизации и облачной конечной точки](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide#create-a-sync-group-and-a-cloud-endpoint). Вам нужно создать группы синхронизации для каждого общего файлового ресурса Windows, который необходимо перенести с узла.
    - [Создание конечной точки сервера](https://docs.microsoft.com/azure/storage/files/storage-sync-files-deployment-guide?tabs=portal#create-a-server-endpoint). Укажите путь в качестве пути к тому StorSimple, который содержит данные общего файлового ресурса. Например, если том StorSimple — это диск `J`, а ваши данные находятся в `J:/<myafsshare>`, добавьте этот путь в качестве конечной точки сервера. Для параметра **Tiering** (Распределение по уровням) оставьте значение **Отключено**.
2.  Подождите, пока не завершится синхронизация файлового сервера. Для каждого сервера в данной группе синхронизации проверьте следующее:
    - Метки времени последней попытки синхронизации для отправки и загрузки не устарели.
    - Состояние отправки и загрузки имеет зеленый цвет.
    - В поле **Действие синхронизации** нет файлов для синхронизации или их очень мало.
    - В полях **Несинхронизирующиеся файлы** для отправки и загрузки стоит значение 0.
    Дополнительные сведения о завершении синхронизации сервера см. в разделе [Как узнать, синхронизируются ли серверы между собой?](https://docs.microsoft.com/azure/storage/files/storage-sync-files-troubleshoot?tabs=portal1%2Cportal#how-do-i-know-if-my-servers-are-in-sync-with-each-other). Синхронизация может занять от нескольких часов до нескольких дней, в зависимости от размера данных и пропускной способности. Как только синхронизация будет завершена, все ваши данные будут безопасно размещены в общем файловом ресурсе Azure. 
3.  Перейдите к общим ресурсам в томах StorSimple. Щелкните правой кнопкой мыши общий ресурс и выберите **Свойства**. Обратите внимание на разрешения общего ресурса в разделе **Безопасность**. Позже эти разрешения необходимо будет вручную применить к новому ресурсу.
4.  Следующие шаги будут зависеть от того, используете ли вы один и тот же узел Windows Server или нет.

    Пропустите этот шаг и перейдите к следующему, если вы используете другой узел Windows Server. Если вы используете тот же файловый сервер Windows для AFS, возникнет небольшой простой. 
    - **Начало простоя**. Удалите конечную точку сервера, созданную во время *шага 1F*. 
    - Создайте конечную точку сервера с путем, в котором позже будут размещаться данные.
    - Как только состояние конечной точки сервера изменится на "Работает" (это может занять несколько минут), данные отобразятся в новом расположении. Теперь вы можете настроить узел Windows Server для обслуживания файлов из нового расположения. - **Завершение простоя**.
5.  Если вы используете другой файловый сервер Windows для службы "Синхронизация файлов Azure", простой не возникнет. 
    - Добавьте еще одну конечную точку сервера с путем локального хранилища, которое вы готовы использовать в качестве кэша вместо устройства StorSimple. 
    - Просмотреть файлы на новом сервере можно будет через несколько минут. В любой момент вы сможете переключиться с устройства StorSimple в новое расположение на узле.

    > [!TIP] 
    > Подумайте о настройке нового общего файлового ресурса с тем же именем и путем, что и ресурса, который он заменяет, чтобы свести к минимуму перебои. При использовании DFS-N может возникнуть необходимость в изменении конфигурации.
6.  Повторно настройте разрешения на общий доступ, как описано в *шаге 3*.

Если во время переноса данных возникают какие-либо проблемы, [обратитесь в службу поддержки Майкрософт](storsimple-8000-contact-microsoft-support.md). 



## <a name="next-steps"></a>Дополнительная информация

Если AFS вам не подходит, узнайте, как [перенести данные с устройства StorSimple серий 5000–7000 на устройство серии 8000](storsimple-8000-migrate-from-5000-7000.md).

