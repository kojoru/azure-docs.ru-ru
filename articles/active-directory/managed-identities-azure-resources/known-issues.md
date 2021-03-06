---
title: Часто задаваемые вопросы об управляемых удостоверениях для ресурсов Azure, а также известные проблемы в их работе
description: Известные проблемы с управляемыми удостоверениями для ресурсов Azure.
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.assetid: 2097381a-a7ec-4e3b-b4ff-5d2fb17403b6
ms.service: active-directory
ms.component: msi
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 12/12/2017
ms.author: daveba
ms.openlocfilehash: 2a759aea4288af2e90335b47244408d6a537e24b
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/10/2018
ms.locfileid: "44295587"
---
# <a name="faqs-and-known-issues-with-managed-identities-for-azure-resources"></a>Часто задаваемые вопросы об управляемых удостоверениях для ресурсов Azure, а также известные проблемы в их работе

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

## <a name="frequently-asked-questions-faqs"></a>Часто задаваемые вопросы (FAQ)

> [!NOTE]
> Управляемые удостоверения для ресурсов Azure — это новое название службы "Управляемое удостоверение службы" (MSI).

### <a name="does-managed-identities-for-azure-resources-work-with-azure-cloud-services"></a>Можно ли использовать управляемые удостоверения для ресурсов Azure с облачными службами Azure?

Нет, мы не планируем реализовать поддержку управляемых удостоверений для ресурсов Azure в облачных службах Azure.

### <a name="does-managed-identities-for-azure-resources-work-with-the-active-directory-authentication-library-adal-or-the-microsoft-authentication-library-msal"></a>Можно ли использовать управляемые удостоверения для ресурсов Azure с библиотекой проверки подлинности Active Directory (ADAL) или библиотекой проверки подлинности Майкрософт (MSAL)?

Нет, управляемые удостоверения для ресурсов Azure еще не интегрированы с библиотеками ADAL и MSAL. Дополнительные сведения о получении маркера для управляемых удостоверений для ресурсов Azure с помощью конечной точки REST см. в статье [Как использовать управляемые удостоверения для ресурсов Azure на виртуальной машине Azure для получения маркера доступа](how-to-use-vm-token.md).

### <a name="what-is-the-security-boundary-of-managed-identities-for-azure-resources"></a>Что такое периметр безопасности для управляемых удостоверений для ресурсов Azure?

Периметром безопасности удостоверения является ресурс, к которому оно подключено. Например, периметром безопасности для виртуальной машины с включенными управляемыми удостоверениями для ресурсов Azure является виртуальная машина. В любом коде, выполняемом на этой виртуальной машине, можно вызвать управляемые удостоверения для конечной точки ресурсов Azure и маркеров запроса. Этот принцип применяется и к другим ресурсам, которые поддерживают управляемые удостоверения для ресурсов Azure.

### <a name="should-i-use-the-managed-identities-for-azure-resources-vm-imds-endpoint-or-the-vm-extension-endpoint"></a>Следует ли использовать управляемые удостоверения для конечной точки IMDS ресурсов Azure или конечной точки расширения виртуальной машины?

При использовании управляемых удостоверений для ресурсов Azure на виртуальных машинах мы рекомендуем использовать управляемые удостоверения для конечной точки IMDS ресурсов Azure. Служба метаданных экземпляров Azure является конечной точкой REST, доступной всем виртуальным машинам IaaS, созданным с помощью Azure Resource Manager. Ниже приведены некоторые из преимуществ использования управляемых удостоверений для ресурсов Azure посредством IMDS.

1. Все операционные системы с поддержкой Azure IaaS могут использовать управляемые удостоверения для ресурсов Azure посредством IMDS. 
2. Для включения управляемых удостоверений для ресурсов Azure больше не нужно устанавливать расширение на виртуальную машину. 
3. Сертификаты, используемые управляемыми удостоверениями для ресурсов Azure, больше не хранятся на виртуальной машине. 
4. Конечная точка IMDS представляет собой известный IP-адрес без поддержки маршрутизации, к которому можно получить доступ только в пределах виртуальной машины. 

Расширение виртуальной машины для управляемых удостоверений для ресурсов Azure по-прежнему можно использовать, однако в дальнейшем в качестве конечной точки по умолчанию будет использоваться IMDS. Расширение виртуальной машины для управляемых удостоверений для ресурсов Azure будет выведено из эксплуатации в январе 2019 г. 

Дополнительные сведения о службе метаданных экземпляров Azure см. в [документации по IMDS](https://docs.microsoft.com/azure/virtual-machines/windows/instance-metadata-service)

### <a name="what-are-the-supported-linux-distributions"></a>Какие дистрибутивы Linux поддерживаются?

Все дистрибутивы Linux, поддерживаемые IaaS Azure, могут использоваться с управляемыми удостоверениями для ресурсов Azure через конечную точку IMDS. 

Примечание. Расширение виртуальной машины для управляемых удостоверений для ресурсов Azure (которое будет выведено из эксплуатации в январе 2019 г.) поддерживает только следующие дистрибутивы Linux:
- CoreOS Stable
- CentOS 7.1;
- Red Hat 7.2
- Ubuntu 15.04.
- Ubuntu 16.04.

Другие дистрибутивы Linux в настоящее время не поддерживаются, поэтому при их использовании установка расширения может завершиться ошибкой.

Расширение работает для CentOS 6.9. Однако из-за отсутствия поддержки системы версии 6.9 при сбое или завершении работы расширения, автоматического перезапуска не произойдет. Оно повторно запустится после перезапуска виртуальной машины. Чтобы перезапустить расширение вручную, ознакомьтесь с разделом [Как перезапустить расширение управляемых удостоверений для ресурсов Azure?](#how-do-you-restart-the-managed-identities-for-Azure-resources-extension)

### <a name="how-do-you-restart-the-managed-identities-for-azure-resources-extension"></a>Как перезапустить расширение управляемых удостоверений для ресурсов Azure?
При завершении работы расширения в Windows и некоторых версиях Linux вы можете перезапустить его вручную с помощью следующего командлета:

```powershell
Set-AzureRmVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

Описание 
- Для Windows используется имя расширения и тип ManagedIdentityExtensionForWindows.
- Для Linux используется имя расширения и тип ManagedIdentityExtensionForLinux.

## <a name="known-issues"></a>Известные проблемы

### <a name="automation-script-fails-when-attempting-schema-export-for-managed-identities-for-azure-resources-extension"></a>Скрипт автоматизации завершается ошибкой при попытке экспорта схемы для расширения управляемых удостоверений для ресурсов Azure

Если на виртуальной машине включены управляемые удостоверения для ресурсов Azure, отображается следующая ошибка при попытке использовать функцию "Скрипт автоматизации" для виртуальной машины или ее группы ресурсов.

![Ошибка экспорта скрипта автоматизации управляемых удостоверений для ресурсов Azure](./media/msi-known-issues/automation-script-export-error.png)

Расширение виртуальной машины для управляемых удостоверений для ресурсов Azure (которое будет выведено из эксплуатации в январе 2019 г.) сейчас не поддерживает возможность экспорта его схемы в шаблон группы ресурсов. Поэтому созданный шаблон не содержит параметры конфигурации для включения управляемых удостоверений для ресурсов Azure в ресурсе. Эти разделы можно добавить вручную. Инструкции и примеры см. в статье [Настройка управляемых удостоверений для ресурсов Azure на виртуальной машине Azure с помощью шаблонов](qs-configure-template-windows-vm.md).

Когда функция экспорта схемы станет доступной для расширения виртуальной машины для управляемых удостоверений для ресурсов Azure (которое будет выведено из эксплуатации в январе 2019 г.), оно будет добавлено в раздел [Поддерживаемые расширения виртуальной машины](../../virtual-machines/extensions/export-templates.md#supported-virtual-machine-extensions).

### <a name="configuration-blade-does-not-appear-in-the-azure-portal"></a>На портале Azure не отображается колонка "Конфигурация"

Если колонка "Конфигурация виртуальной машины" для виртуальной машины не отображается, значит управляемые удостоверения для ресурсов Azure еще не включены на портале в вашем регионе.  Повторите попытку позже.  Управляемые удостоверения для ресурсов Azure можно также включить для виртуальной машины с помощью [PowerShell](qs-configure-powershell-windows-vm.md) или [Azure CLI](qs-configure-cli-windows-vm.md).

### <a name="cannot-assign-access-to-virtual-machines-in-the-access-control-iam-blade"></a>Не удается назначить доступ виртуальным машинам в колонке "Управление доступом (IAM)"

Если на портале Azure в списке **Назначение доступа к** на странице **Управление доступом (IAM)** > **Добавление разрешений** недоступен для выбора пункт **Виртуальная машина**, то управляемые удостоверения для ресурсов Azure еще не включены на портале в вашем регионе. Повторите попытку позже.  Вы все равно можете выбрать удостоверение для виртуальной машины, чтобы назначить роль, выполнив поиск субъекта-службы управляемых удостоверений для ресурсов Azure.  Введите имя виртуальной машины в поле **Выбор**, и субъект-служба отобразится в результатах поиска.

### <a name="vm-fails-to-start-after-being-moved-from-resource-group-or-subscription"></a>Виртуальная машина не запускается после перемещения из группы ресурсов или подписки

Если перемещается запущенная виртуальная машина, то она продолжает работать во время перемещения. Однако если после перемещения виртуальная машина останавливается и перезапускается, то ее запуск завершается сбоем. Эта проблема возникает из-за того, что виртуальная машина не обновляет ссылку на идентификатор управляемых удостоверений для ресурсов Azure и продолжает указывать на него в старой группе ресурсов.

**Возможное решение** 
 
Активируйте обновление на виртуальной машине, чтобы она могла получить правильные значения управляемых удостоверений для ресурсов Azure. Можно изменить свойства виртуальной машины, чтобы обновить ссылку на ресурс управляемых удостоверений для ресурсов Azure. Например, можно задать новое значение тега виртуальной машины с помощью следующей команды.

```azurecli-interactive
 az  vm update -n <VM Name> -g <Resource Group> --set tags.fixVM=1
```
 
Эта команда задает новый тег fixVM со значением 1 для виртуальной машины. 
 
После установки этого свойства виртуальная машина обновится и будет использовать правильный универсальный код ресурса (URI) управляемых удостоверений для ресурсов Azure, и вы сможете запустить ее. 
 
После запуска виртуальной машины этот тег можно будет удалить, выполнив следующую команду.

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --remove tags.fixVM
```

## <a name="known-issues-with-user-assigned-identities"></a>Известные проблемы с удостоверениями, назначаемыми пользователем

- Задание удостоверений, назначаемых пользователем, доступно только для виртуальных машин и масштабируемых наборов виртуальных машин. Важно. Задание удостоверений, назначаемых пользователем, будет изменено в ближайшие месяцы.
- Повторяющиеся удостоверения, назначаемые пользователем, на одной и той же виртуальной машине или в одном и том же масштабируемом наборе виртуальных машин приведут к сбою этой виртуальной машины или масштабируемого набора. Это относится и к удостоверениям, добавляемым в разном регистре. Например, MyUserAssignedIdentity и myuserassignedidentity. 
- Подготовка расширения для виртуальной машины (которое будет выведено из эксплуатации в январе 2019 г.) может завершиться сбоем из-за ошибок при поиске DNS. Перезапустите виртуальную машину и повторите попытку. 
- Добавление несуществующего удостоверения, назначаемого пользователем, вызовет сбой виртуальной машины. 
- При создании удостоверения, назначаемого пользователем, не поддерживается использование специальных символов (например, символа подчеркивания) в имени.
- В комплексном сценарии имена удостоверений, назначаемых пользователем, должны содержать не более 24 символов. Назначение пользователем удостоверений с именами, содержащими более 24 символов, завершится сбоем.
- Если используется расширение виртуальной машины для управляемых удостоверений (которое будет выведено из эксплуатации в январе 2019 г.), поддерживается не более 32 управляемых удостоверений, назначаемых пользователем. Если расширение виртуальной машины для управляемых удостоверений отсутствует, то лимит составляет 512.  
- При добавлении второго удостоверения, назначаемого пользователем, идентификатор clientID может оказаться недоступным для маркеров запроса для расширения виртуальной машины. Чтобы устранить эту проблему, перезапустите расширение виртуальной машины для управляемых удостоверений для ресурсов Azure с помощью двух приведенных ниже команд Bash.
 - `sudo bash -c "/var/lib/waagent/Microsoft.ManagedIdentity.ManagedIdentityExtensionForLinux-1.0.0.8/msi-extension-handler disable"`
 - `sudo bash -c "/var/lib/waagent/Microsoft.ManagedIdentity.ManagedIdentityExtensionForLinux-1.0.0.8/msi-extension-handler enable"`
- Если на виртуальной машине есть удостоверение, назначаемое пользователем, но отсутствует удостоверение, назначаемое системой, то в пользовательском интерфейсе портала будет отображаться информация о том, что управляемые удостоверения для ресурсов Azure отключены. Чтобы включить удостоверение, назначаемое системой, используйте шаблон Azure Resource Manager, Azure CLI или пакет SDK.
