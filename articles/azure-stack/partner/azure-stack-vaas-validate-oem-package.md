---
title: Проверка пакетов поставщика вычислительной техники (OEM) с помощью проверки как услуги Azure Stack | Документация Майкрософт
description: Узнайте, как проверить пакеты поставщика вычислительной техники (OEM) с помощью проверки как услуги.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 10/19/2018
ms.author: mabrigg
ms.reviewer: johnhas
ms.openlocfilehash: bcfc4cb65c94e34e9f6056ada53726f88489fefb
ms.sourcegitcommit: ccdea744097d1ad196b605ffae2d09141d9c0bd9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/23/2018
ms.locfileid: "49646657"
---
# <a name="validate-oem-packages"></a>Проверка пакетов OEM

[!INCLUDE [Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

Вы можете протестировать новый пакет OEM при изменение встроенного ПО или драйверов для проверки завершенного решения. Пакет подписывается корпорацией Майкрософт после прохождения теста. В проверке должен содержаться обновленный пакет расширений OEM с драйверами и встроенным ПО, которые прошли проверки по логотипу и PCS Windows Server.

[!INCLUDE [azure-stack-vaas-workflow-validation-completion](includes/azure-stack-vaas-workflow-validation-completion.md)]

> [!IMPORTANT]
> Перед передачей или отправкой пакетов ознакомьтесь со статьей [Create an OEM package](azure-stack-vaas-create-oem-package.md) (Создание пакета OEM), чтобы получить сведения о предполагаемом формате и содержимом пакета.

## <a name="managing-packages-for-validation"></a>Управление пакетами для проверки

При использовании рабочего процесса **проверки пакета**, чтобы проверить пакет, необходимо будет указать URL-адрес **Azure Storage Blob**. Этот большой двоичный объект является пакетом OEM, установленным в решении во время развертывания. Для создания большого двоичного объекта с помощью учетной записи хранения Azure, созданной во время установки, обратитесь к статье [Руководство по настройке ресурсов для проверки как услуги](azure-stack-vaas-set-up-resources.md).

### <a name="prerequisite-provision-a-storage-container"></a>Предварительные требования: подготовка контейнера хранилища

Создайте контейнер в учетной записи хранения для больших двоичных объектов пакета. Этот контейнер может использоваться для всех ваших запусков проверки пакета.

1. На [портале Azure](https://portal.azure.com) перейдите к учетной записи хранения, созданной в рамках статьи [Руководство по настройке ресурсов для проверки как услуги](azure-stack-vaas-set-up-resources.md).
2. В левой колонке в разделе **Служба BLOB-объектов** выберите **Контейнеры**.
3. Выберите **+ Container** (+ Контейнер) в строке меню и укажите имя для контейнера, например `vaaspackages`.

### <a name="upload-package-to-storage-account"></a>Отправка пакета в учетную запись хранения

1. Подготовьте пакет, который требуется проверить. Если пакет содержит несколько файлов, сожмите их в файл `.zip`.
2. На [портале Azure](https://portal.azure.com) выберите контейнер пакета и отправьте его, выбрав в строке меню **Отправить**.
3. Выберите файл `.zip` пакета для отправки. Оставьте значения по умолчанию для **типа большого двоичного объекта** (т. е. **блочный BLOB-объект**) и **размера блока**.

> [!NOTE]
> Убедитесь, что содержимое `.zip` помещается в корне файла `.zip`. В пакете не должно быть вложенных папок.

### <a name="generate-package-blob-url-for-vaas"></a>Создание URL-адреса большого двоичного объекта пакета для VaaS

При создании рабочего процесса **проверки пакета** на портале VaaS необходимо будет указать URL-адрес Azure Storage Blob, содержащего пакет.

#### <a name="option-1-generating-an-account-sas-url"></a>Вариант 1. Создание URL-адреса SAS учетной записи

1. [!INCLUDE [azure-stack-vaas-sas-step_navigate](includes/azure-stack-vaas-sas-step_navigate.md)]

1. Выберите **Большой двоичный объект** в разделе **Разрешенные службы**. Снимите все остальные флажки.

1. В разделе **Разрешенные типы ресурсов** выберите **Контейнер** и **Объект**. Снимите все остальные флажки.

1. Выберите **Read** (Чтение) и **List** (Список) в разделе **Предоставленные разрешения**. Снимите все остальные флажки.

1. Задайте текущее время в поле **Время начала**, а в поле **Время окончания** — время, отстоящее на 1 час от текущего.

1. [!INCLUDE [azure-stack-vaas-sas-step_generate](includes/azure-stack-vaas-sas-step_generate.md)]
    Вот какой формат должен появиться: `https://storageaccountname.blob.core.windows.net/?sv=2016-05-31&ss=b&srt=co&sp=rl&se=2017-05-11T21:41:05Z&st=2017-05-11T13:41:05Z&spr=https`

1. Измените созданный URL-адрес SAS для включения контейнера пакета, `{containername}`, и имя большого двоичного объекта пакета, `{mypackage.zip}`, как показано ниже: `https://storageaccountname.blob.core.windows.net/{containername}/{mypackage.zip}?sv=2016-05-31&ss=b&srt=co&sp=rl&se=2017-05-11T21:41:05Z&st=2017-05-11T13:41:05Z&spr=https`

    Используйте это значение при запуске нового рабочего процесса **проверки пакета** на портале VaaS.

#### <a name="option-2-using-public-read-container"></a>Вариант 2. Использование контейнера с общим доступом на чтение

> [!CAUTION]
> Благодаря этому параметру контейнер открывается для анонимного доступа только для чтения.

1. Предоставьте **общий доступ на чтение только для больших двоичных объектов** к контейнеру пакета, следуя инструкциям в разделе [Предоставление анонимным пользователям разрешений для контейнеров и больших двоичных объектов](https://docs.microsoft.com/azure/storage/storage-manage-access-to-resources#grant-anonymous-users-permissions-to-containers-and-blobs).

2. В контейнере пакета выберите большой двоичный объект пакета, чтобы открыть панель свойств.

3. Скопируйте **URL-адрес**. Используйте это значение при запуске нового рабочего процесса **проверки пакета** на портале VaaS.

## <a name="apply-monthly-update"></a>Применение ежемесячного обновления

[!INCLUDE [azure-stack-vaas-workflow-section_update-azs](includes/azure-stack-vaas-workflow-section_update-azs.md)]

## <a name="create-a-package-validation-workflow"></a>Создание рабочего процесса проверки пакета

1. Войдите на [портал VaaS](https://azurestackvalidation.com).

2. [!INCLUDE [azure-stack-vaas-workflow-step_select-solution](includes/azure-stack-vaas-workflow-step_select-solution.md)]

3. Выберите **Start** (Начать) на плитке **Package Validation** (Проверка пакета).

    ![Плитка рабочего процесса проверок пакета](media/tile_validation-package.png)

4. [!INCLUDE [azure-stack-vaas-workflow-step_naming](includes/azure-stack-vaas-workflow-step_naming.md)]

5. Введите URL-адрес Azure Storage Blob для пакета OEM, установленного в решении во время развертывания. Инструкции см. в разделе [Создание URL-адреса большого двоичного объекта пакета для VaaS](#generate-package-blob-url-for-vaas).

6. [!INCLUDE [azure-stack-vaas-workflow-step_upload-stampinfo](includes/azure-stack-vaas-workflow-step_upload-stampinfo.md)]

7. [!INCLUDE [azure-stack-vaas-workflow-step_test-params](includes/azure-stack-vaas-workflow-step_test-params.md)]

    > [!NOTE]
    > Параметры среды невозможно изменить после создания рабочего процесса.

8. [!INCLUDE [azure-stack-vaas-workflow-step_tags](includes/azure-stack-vaas-workflow-step_tags.md)]

9. [!INCLUDE [azure-stack-vaas-workflow-step_submit](includes/azure-stack-vaas-workflow-step_submit.md)]
    Вы будете перенаправлены на страницу сводок по тестам.

## <a name="run-package-validation-tests"></a>Запуск тестов проверки пакета

На странице **Package validation tests summary** (Сводки по тестам проверки пакета) вы увидите список тестов, необходимых для завершения проверки. В этом рабочем процессе тесты выполняются примерно в течение 24 часов.

[!INCLUDE [azure-stack-vaas-workflow-validation-section_schedule](includes/azure-stack-vaas-workflow-validation-section_schedule.md)]

Когда все тесты будут успешно выполнены, отправьте имя своей проверки пакета и решения VaaS по адресу [vaashelp@microsoft.com](mailto:vaashelp@microsoft.com) для запроса на подписание пакета.

## <a name="next-steps"></a>Дополнительная информация

- [Мониторинг теста с помощью проверки как услуги Azure Stack](azure-stack-vaas-monitor-test.md)