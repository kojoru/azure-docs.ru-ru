---
title: Настройка аварийного восстановления виртуальных машин Azure в дополнительный регион Azure с помощью службы Azure Site Recovery
description: Сведения о настройке аварийного восстановления виртуальных машин Azure в другой регион Azure с помощью службы Azure Site Recovery.
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: tutorial
ms.date: 10/19/2018
ms.author: raynew
ms.custom: mvc
ms.openlocfilehash: ab79201442fe9c7334525972c34eba86daf27e99
ms.sourcegitcommit: 668b486f3d07562b614de91451e50296be3c2e1f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/19/2018
ms.locfileid: "49457431"
---
# <a name="set-up-disaster-recovery-for-azure-vms-to-a-secondary-azure-region"></a>Настройка аварийного восстановления виртуальных машин Azure в дополнительный регион Azure

Служба [Azure Site Recovery](site-recovery-overview.md) помогает реализовать стратегию аварийного восстановления за счет управления процессами репликации, отработки отказа и восстановления локальных компьютеров и виртуальных машин Azure.

В этом учебнике содержатся сведения о настройке аварийного восстановления виртуальных машин Azure в дополнительном регионе Azure. Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание хранилища служб восстановления
> * Проверка параметров целевых ресурсов
> * Настройка исходящего доступа для виртуальных машин
> * Включение репликации для виртуальных машин.

> [!NOTE]
> Это руководство содержит пошаговую инструкцию для пользователя по активации репликации с минимальной настройкой. Если вам нужны дополнительные сведения о различных аспектах, связанных с аварийным восстановлением, включая требования к сети, автоматизацию или устранение неполадок, см. статьи с руководствами для виртуальных машин Azure.


## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством:

- Вам должны быть понятны [архитектура и компоненты сценария](concepts-azure-to-azure-architecture.md).
- [Ознакомьтесь](site-recovery-support-matrix-azure-to-azure.md) с требованиями поддержки для всех компонентов.

## <a name="create-a-vault"></a>Создание хранилища

Создайте хранилище в любом регионе, за исключением исходного.

1. Войдите на [портал Azure](https://portal.azure.com) > **Службы восстановления**.
2. Последовательно выберите **Создать ресурс** > **Мониторинг и управление** > **Backup and Site Recovery**.
3. В поле **Имя**укажите понятное имя для идентификации хранилища. Если у вас есть несколько подписок, выберите нужную.
4. Создайте группу ресурсов или выберите существующую. Укажите регион Azure. Сведения о поддерживаемых регионах см. в разделе "Географическая доступность" на странице [цен на службу Azure Site Recovery](https://azure.microsoft.com/pricing/details/site-recovery/).
5. Для быстрого доступа к хранилищу из панели мониторинга щелкните **Закрепить на панели мониторинга**, а затем — **Создать**.

   ![Новое хранилище](./media/azure-to-azure-tutorial-enable-replication/new-vault-settings.png)

   Новое хранилище появится в колонке **Панель мониторинга** в разделе **Все ресурсы** и на основной странице **Хранилища служб восстановления**.

## <a name="verify-target-resources"></a>Проверка целевых ресурсов

1. Убедитесь, что подписка Azure позволяет создавать виртуальные машины в целевом регионе, используемом для аварийного восстановления. Свяжитесь со службой поддержки, чтобы включить необходимые квоты.

2. Убедитесь, что подписка располагает достаточными ресурсами для поддержки виртуальных машин таких же размеров, которые установлены для исходных виртуальных машин. Служба Site Recovery выбирает для целевой виртуальной машины такой же размер или похожий.

## <a name="configure-outbound-network-connectivity"></a>Настройка исходящих сетевых подключений

Чтобы служба Site Recovery работала ожидаемым образом, необходимо внести некоторые изменения в исходящее сетевое подключение из виртуальных машин, которые требуется реплицировать.

- Site Recovery не поддерживает использование прокси-службы проверки подлинности для управления сетевым подключением.
- При использовании прокси-службы проверки подлинности репликацию невозможно включить.

### <a name="outbound-connectivity-for-urls"></a>Исходящие подключения для URL-адресов

При использовании прокси-сервера брандмауэра на основе URL-адресов для управления исходящими подключениями разрешите использование следующих URL-адресов в службе Site Recovery.

| **URL-адрес** | **Дополнительные сведения** |
| ------- | ----------- |
| *.blob.core.windows.net | Разрешает записывать данные из виртуальной машины в учетную запись хранения кэша в исходном регионе. |
| login.microsoftonline.com | Предоставляет авторизацию и проверку подлинности URL-адресов службы Site Recovery. |
| *.hypervrecoverymanager.windowsazure.com | Разрешает виртуальной машине взаимодействовать со службой Site Recovery. |
| *.servicebus.windows.net | Разрешает виртуальной машине записывать данные мониторинга и диагностики службы Site Recovery. |

### <a name="outbound-connectivity-for-ip-address-ranges"></a>Исходящие подключения для диапазонов IP-адресов

Чтобы управлять исходящим подключением с использованием IP-адресов, а не URL-адресов, внесите в список разрешений соответствующие диапазоны центра обработки данных, адреса Office 365, а также адреса конечных точек служб для брандмауэров на основе IP-адресов, прокси-серверов или правил NSG.

  - [Диапазоны IP-адресов центра обработки данных Microsoft Azure](http://www.microsoft.com/en-us/download/details.aspx?id=41653)
  - [Диапазоны IP-адресов центра обработки данных Microsoft Azure в Германии](http://www.microsoft.com/en-us/download/details.aspx?id=54770)
  - [Диапазоны IP-адресов центра обработки данных Microsoft Azure в Китае](http://www.microsoft.com/en-us/download/details.aspx?id=42064)
  - [URL-адреса и диапазоны IP-адресов Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity)
  - [IP-адреса конечных точек службы Site Recovery](https://aka.ms/site-recovery-public-ips)

Для создания необходимых правил NSG можно использовать [этот скрипт](https://gallery.technet.microsoft.com/Azure-Recovery-script-to-0c950702).

## <a name="verify-azure-vm-certificates"></a>Проверка сертификатов виртуальной машины Azure

Убедитесь, что на виртуальной машине Windows или Linux, которую вы будете реплицировать, установлены все свежие корневые сертификаты. Если новых корневых сертификатов нет, виртуальную машину нельзя регистрировать в Site Recovery из-за ограничений безопасности.

- При использовании виртуальных машин Windows установите на виртуальной машине все последние обновления Windows, чтобы гарантировать присутствие всех доверенных корневых сертификатов. При работе в отключенной среде следуйте стандартным процедурам обновления Windows и сертификатам в вашей организации.

- На виртуальных машинах Linux выполните инструкции распространителя Linux, чтобы установить на виртуальные машины все свежие доверенные корневые сертификаты и список отзыва сертификатов.

## <a name="set-permissions-on-the-account"></a>Установка разрешений для учетной записи

Служба Azure Site Recovery предоставляет три встроенные роли для контроля операций управления Site Recovery.

- **Участник Site Recovery**. Эта роль имеет все разрешения, необходимые для управления операциями Azure Site Recovery в хранилище служб восстановления. Пользователь с этой ролью, тем не менее, не может создать или удалить хранилище служб восстановления или назначить права доступа другим пользователям. Эта роль лучше всего подходит для администраторов аварийного восстановления, которые могут включить аварийное восстановление и управлять им для приложений или всей организации.

- **Оператор Site Recovery**. Эта роль имеет разрешения на выполнение операций отработки отказа и восстановления размещения и управление ими. Пользователь с этой ролью не может включить или отключить репликацию, создать или удалить хранилище, зарегистрировать новую инфраструктуру или назначить права доступа другим пользователям. Эта роль лучше всего подходит для оператора аварийного восстановления, который может выполнять отработку отказа виртуальных машин или приложений по указанию владельцев приложений и ИТ-администраторов. После устранения аварии оператор аварийного восстановления может повторно применить защиту и восстановить размещение виртуальных машин.

- **Читатель Site Recovery**. Эта роль имеет разрешения на просмотр всех операций управления Site Recovery. Она лучше всего подходит для специалиста по контролю ИТ, который может отслеживать текущее состояние защиты и отправлять запросы в службу поддержки.

Дополнительные сведения см. в статье о [встроенных ролях RBAC в Azure](../role-based-access-control/built-in-roles.md).

## <a name="enable-replication"></a>Включение репликации

### <a name="select-the-source"></a>Выбор источника

1. В хранилище служб восстановления щелкните имя хранилища и выберите **Реплицировать**.
2. В поле **Источник** выберите **Azure**.
3. В поле **Исходное расположение** выберите исходный регион Azure, в котором сейчас запущены виртуальные машины.
4. Для виртуальных машин **Resource Manager** или **классических** виртуальных машин выберите **модель развертывания виртуальной машины Azure**.
5. Выберите **исходную подписку**, в которой работают виртуальные машины. Это может быть любая подписка в одном клиенте Azure Active Directory, где существует хранилище служб восстановления.
6. Выберите **исходную группу ресурсов** для виртуальных машин Resource Manager или **облачную службу** для классических виртуальных машин.
7. Нажмите кнопку **ОК**, чтобы сохранить настройки.

### <a name="select-the-vms"></a>Выбор виртуальных машин

Site Recovery получает список виртуальных машин, связанных с подпиской и группой ресурсов или облачной службой.

1. В разделе **Виртуальные машины** выберите виртуальные машины, которые необходимо реплицировать.
2. Последовательно выберите **ОК**.

### <a name="configure-replication-settings"></a>Настройка параметров репликации

Site Recovery создает параметры по умолчанию и политику репликации для целевого региона. Эти параметры можно изменить в соответствии с требованиями.

1. Чтобы просмотреть целевой объект и параметры репликации, выберите **Параметры**.
2. Чтобы переопределить параметры по умолчанию для целевых объектов, щелкните ссылку **Customize** (Настроить) рядом с заголовком **Resource group, Network, Storage and Availability Sets** (Группа ресурсов, сеть, хранилище и группы доступности).

  ![Настройка параметров](./media/azure-to-azure-tutorial-enable-replication/settings.png)


- **Целевая подписка**. Целевая подписка, используемая для аварийного восстановления. По умолчанию целевая подписка будет совпадать с исходной подпиской. Щелкните "Настроить", чтобы выбрать другую целевую подписку в том же клиенте Azure Active Directory.

- **Целевое расположение**. Целевой регион, используемый для аварийного восстановления. Мы рекомендуем, чтобы целевое расположение соответствовало расположению хранилища Site Recovery.

- **Целевая группа ресурсов**. Группа ресурсов в целевом регионе, в которую входят виртуальные машины Azure после отработки отказа. По умолчанию Site Recovery создает группу ресурсов в целевом регионе с суффиксом asr. Целевая группа ресурсов может располагаться в любом регионе за исключением того, в котором размещены исходные виртуальные машины.

- **Целевая виртуальная сеть**. Сеть в целевом регионе, в которой размещаются виртуальные машины Azure после отработки отказа.
  По умолчанию Site Recovery создает виртуальную сеть (и подсети) в целевом регионе с суффиксом asr.

- **Cache storage accounts** (Учетные записи хранения кэша). Site Recovery использует учетную запись хранения в исходном регионе. Изменения в исходных виртуальных машинах отправляются в эту учетную запись перед репликацией в целевое расположение.

- **Целевые учетные записи хранения (если на исходной виртуальной машине не используются управляемые диски).** По умолчанию Site Recovery создает учетную запись хранения в целевом регионе, чтобы отобразить учетную запись хранения исходной виртуальной машины.

- **Управляемые диски реплики (если на исходной виртуальной машине используются управляемые диски).** По умолчанию Site Recovery создает управляемые диски реплики в целевом регионе, которые отражают управляемые диски исходной виртуальной машины с тем же типом хранилища (класса "Стандартный" или "Премиум"), что и на управляемом диске исходной виртуальной машины.

- **Целевые группы доступности**. По умолчанию Site Recovery создает группу доступности в целевом регионе с суффиксом asr. Группы доступности можно добавить, только если виртуальные машины входят в состав исходного региона.

Чтобы переопределить параметры политики репликации по умолчанию, щелкните **Customize** (Настроить) рядом с заголовком **Политика репликации**.  

- **Replication policy name** (Имя политики репликации). Имя политики.

- **Хранение точки восстановления**. По умолчанию Site Recovery хранит точки восстановления 24 часа. Вы можете задать значение от 1 до 72 часов.

- **Периодичность создания моментальных снимков с согласованием приложений**. По умолчанию Site Recovery выполняет моментальные снимки с согласованием приложений каждые 4 часа. Вы можете задать значение от 1 до 12 часов. Моментальный снимок с согласованием приложений представляет собой созданный на определенный момент времени снимок данных приложения в виртуальной машине. Служба теневого копирования томов (VSS) обеспечивает согласованное состояние приложения в виртуальной машине при создании моментального снимка.

- **Группа репликации**. Если приложению требуется согласованность между несколькими виртуальными машинами, вы можете создать для них группу репликации. По умолчанию выбранные виртуальные машины не входят ни в одну группу репликации.

  Щелкните **Customize** (Настроить) рядом с заголовком **Политика репликации**, а затем выберите **Yes** (Да) для обеспечения согласованности между несколькими виртуальными машинами. Это нужно, чтобы они стали частью группы репликации. Вы можете создать группу репликации или использовать имеющуюся. Выберите виртуальные машины, которые должны быть частью группы репликации, и нажмите кнопку **ОК**.

> [!IMPORTANT]
  На всех компьютерах в группе репликации будут общие отказоустойчивые и согласованные точки восстановления после отработки отказа. Включение согласованности нескольких виртуальных машин может повлиять на производительность рабочей нагрузки, поэтому ее необходимо использовать, только если компьютеры выполняют одну и ту же рабочую нагрузку и должны действовать согласованно.

> [!IMPORTANT]
  Если включить согласованность между виртуальными машинами, компьютеры в группе репликации будут обмениваться данными друг с другом через порт 20004. Убедитесь, что нет устройства брандмауэра, которое блокирует внутренний обмен данными между виртуальными машинами через порт 20004. Если вы хотите, чтобы виртуальные машины Linux входили в группу репликации, нужно вручную открыть исходящий трафик через порт 20004 согласно указаниям для конкретной версии Linux.

### <a name="configure-encryption-settings"></a>Настройка параметров шифрования

Если в исходной виртуальной машине используется шифрование дисков Azure (ADE), отобразится раздел с указанными ниже настройками шифрования.

- **Хранилища ключей для шифрования диска**. По умолчанию Azure Site Recovery создает новое хранилище ключей в целевом регионе с именем, содержащим суффикс asr, используя ключи шифрования дисков исходной виртуальной машины. Если хранилище ключей, создаваемое Azure Site Recovery, уже существует, оно будет использовано повторно.
- **Хранилища ключей шифрования ключей**. По умолчанию Azure Site Recovery создает новое хранилище ключей в целевом регионе с именем, содержащим суффикс asr, используя ключи шифрования ключей исходной виртуальной машины. Если хранилище ключей, создаваемое Azure Site Recovery, уже существует, оно будет использовано повторно.

Нажмите "Настроить" рядом с параметрами шифрования, чтобы изменить параметры по умолчанию и выбрать пользовательские хранилища ключей.

>[!NOTE]
>Azure Site Recovery в настоящее время поддерживает только виртуальные машины Azure под управлением ОС Windows, в которых [включено шифрование с помощью приложения Azure AD](https://aka.ms/ade-aad-app).
>

### <a name="track-replication-status"></a>Отслеживание состояния репликации

1. В разделе **Параметры** щелкните **Обновить**, чтобы вывести актуальное состояние.

2. Ход выполнения задания **включения защиты** можно отслеживать, выбрав **Параметры** > **Задания** > **Задания Site Recovery**.

3. Выберите **Параметры** > **Реплицированные элементы**, чтобы просмотреть состояние виртуальных машин и ход выполнения начальной репликации. Выберите виртуальную машину, чтобы просмотреть ее параметры.

## <a name="next-steps"></a>Дополнительная информация

В этом руководстве вы настроили аварийное восстановление для виртуальной машины Azure. Далее нужно протестировать конфигурацию, выполнив отработку аварийного восстановления.

> [!div class="nextstepaction"]
> [Run a disaster recovery drill to Azure](azure-to-azure-tutorial-dr-drill.md) (Выполнение отработки аварийного восстановления в Azure)
