---
title: Удостоверение службы фабрики данных Azure | Документация Майкрософт
description: Сведения об удостоверении службы фабрики данных в фабрике данных Azure.
services: data-factory
author: linda33wj
manager: craigg
editor: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/17/2018
ms.author: jingwang
ms.openlocfilehash: db0bc0cb64c0b6d7df9319c8d2c5850a27e767a1
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/03/2018
ms.locfileid: "48249219"
---
# <a name="azure-data-factory-service-identity"></a>Удостоверение службы фабрики данных Azure

Эта статья поможет понять, что такое удостоверение службы фабрики данных и как оно работает.

## <a name="overview"></a>Обзор

При создании фабрики данных создается удостоверение службы. Это управляемое приложение, зарегистрированное в Azure Activity Directory и представляющее эту определенную фабрику данных.

Удостоверение службы фабрики данных использует преимущества следующих возможностей:

- [Хранение учетных данных в хранилище ключей Azure](store-credentials-in-key-vault.md). В этом случае удостоверение службы фабрики данных используется для проверки подлинности в Azure Key Vault;
- Соединители, включая [хранилище BLOB-объектов Azure](connector-azure-blob-storage.md), [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md), [Базу данных SQL Azure](connector-azure-sql-database.md) и [Хранилище данных SQL Azure](connector-azure-sql-data-warehouse.md).

## <a name="generate-service-identity"></a>Создание удостоверения службы

Удостоверение службы фабрики данных формируется следующим образом.

- При создании фабрики данных с помощью **портала Azure или PowerShell** удостоверение службы всегда будет создаваться автоматически.
- При создании фабрики данных с помощью пакета **SDK** удостоверение службы будет создаваться, только если в создаваемом объекте фабрики указать "Identity = new FactoryIdentity()". Пример см. в инструкциях по [созданию фабрики данных из краткого руководства по .NET](quickstart-create-data-factory-dot-net.md#create-a-data-factory).
- При создании фабрики данных с помощью **REST API** удостоверение службы будет создано, только если в тексте запроса указан раздел identity. Пример см. в инструкциях по [созданию фабрики данных из краткого руководства по REST](quickstart-create-data-factory-rest-api.md#create-a-data-factory).

Если при выполнении инструкций по [получению удостоверения службы](#retrieve-service-identity) вы обнаружили, что для фабрики данных соответствующее удостоверение отсутствует, вы можете явным образом создать его, обновив фабрику данных программными средствами — с помощью инициатора удостоверений:

- [Создание удостоверения службы с помощью PowerShell](#generate-service-identity-using-powershell)
- [Создание удостоверения службы с помощью REST API](#generate-service-identity-using-rest-api)
- [Создание удостоверения службы с помощью пакета SDK](#generate-service-identity-using-sdk)

>[!NOTE]
>- Удостоверение службы невозможно изменить. Обновление фабрики данных, в которой уже есть удостоверение службы, ничего не изменит. Удостоверение службы останется таким же.
>- Если обновить фабрику данных, в которой уже есть удостоверение службы, не указав параметр identity в объекте фабрики либо не указав раздел identity в тексте запроса REST, произойдет ошибка.
>- При удалении фабрики данных соответствующее удостоверение службы будет удалено вместе с ней.

### <a name="generate-service-identity-using-powershell"></a>Создание удостоверения службы с помощью PowerShell

Вызовите команду **Set-AzureRmDataFactoryV2** еще раз. Будут созданы поля Identity:

```powershell
PS C:\WINDOWS\system32> Set-AzureRmDataFactoryV2 -ResourceGroupName <resourceGroupName> -Name <dataFactoryName> -Location <region>

DataFactoryName   : ADFV2DemoFactory
DataFactoryId     : /subscriptions/<subsID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/ADFV2DemoFactory
ResourceGroupName : <resourceGroupName>
Location          : East US
Tags              : {}
Identity          : Microsoft.Azure.Management.DataFactory.Models.FactoryIdentity
ProvisioningState : Succeeded
```

### <a name="generate-service-identity-using-rest-api"></a>Создание удостоверения службы с помощью REST API

Вызовите API с разделом identity в тексте запроса, как в примере ниже.

```
PATCH https://management.azure.com/subscriptions/<subsID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<data factory name>?api-version=2017-09-01-preview
```

**Текст запроса**: add "identity": { "type": "SystemAssigned" }.

```json
{
    "name": "<dataFactoryName>",
    "location": "<region>",
    "properties": {},
    "identity": {
        "type": "SystemAssigned"
    }
}
```

**Ответ**: удостоверение службы создается автоматически, и раздел identity заполняется соответствующим образом.

```json
{
    "name": "ADFV2DemoFactory",
    "tags": {},
    "properties": {
        "provisioningState": "Succeeded",
        "loggingStorageAccountKey": "**********",
        "createTime": "2017-09-26T04:10:01.1135678Z",
        "version": "2017-09-01-preview"
    },
    "identity": {
        "type": "SystemAssigned",
        "principalId": "765ad4ab-XXXX-XXXX-XXXX-51ed985819dc",
        "tenantId": "72f988bf-XXXX-XXXX-XXXX-2d7cd011db47"
    },
    "id": "/subscriptions/<subscriptionId>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/ADFV2DemoFactory",
    "type": "Microsoft.DataFactory/factories",
    "location": "EastUS"
}
```

### <a name="generate-service-identity-using-sdk"></a>Создание удостоверения службы с помощью пакета SDK

Вызовите функцию create_or_update фабрики данных с Identity=new FactoryIdentity(). Пример кода с использованием .NET:

```csharp
Factory dataFactory = new Factory
{
    Location = <region>,
    Identity = new FactoryIdentity()
};
client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
```

## <a name="retrieve-service-identity"></a>Получение удостоверения службы

Удостоверение службы можно получить с помощью портала Azure или программных средств. В разделах ниже приведено несколько примеров.

>[!TIP]
> Если удостоверение службы не отображается, [создайте его](#generate-service-identity), обновив фабрику.

### <a name="retrieve-service-identity-using-azure-portal"></a>Получение удостоверения службы с помощью портала Azure

Сведения об удостоверении службы можно найти на портале Azure. Выберите свою фабрику, данных, а затем щелкните "Параметры" -> "Свойства":

- ИДЕНТИФИКАТОР УДОСТОВЕРЕНИЯ СЛУЖБЫ
- КЛИЕНТ УДОСТОВЕРЕНИЯ СЛУЖБЫ
- **ИДЕНТИФИКАТОР ПРИЛОЖЕНИЯ УДОСТОВЕРЕНИЯ СЛУЖБЫ** > скопируйте это значение.

![Получение удостоверения службы](media/data-factory-service-identity/retrieve-service-identity-portal.png)

### <a name="retrieve-service-identity-using-powershell"></a>Получение удостоверения службы с помощью PowerShell

Идентификатор субъекта удостоверения службы и идентификатор клиента возвращаются при получении определенной фабрики данных:

```powershell
PS C:\WINDOWS\system32> (Get-AzureRmDataFactoryV2 -ResourceGroupName <resourceGroupName> -Name <dataFactoryName>).Identity

PrincipalId                          TenantId
-----------                          --------
765ad4ab-XXXX-XXXX-XXXX-51ed985819dc 72f988bf-XXXX-XXXX-XXXX-2d7cd011db47
```

Скопируйте идентификатор субъекта, а затем выполните указанную ниже команду Azure Active Directory с идентификатором субъекта в качестве параметра, чтобы получить **идентификатор приложения**, используемый для предоставления доступа:

```powershell
PS C:\WINDOWS\system32> Get-AzureRmADServicePrincipal -ObjectId 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc

ServicePrincipalNames : {76f668b3-XXXX-XXXX-XXXX-1b3348c75e02, https://identity.azure.net/P86P8g6nt1QxfPJx22om8MOooMf/Ag0Qf/nnREppHkU=}
ApplicationId         : 76f668b3-XXXX-XXXX-XXXX-1b3348c75e02
DisplayName           : ADFV2DemoFactory
Id                    : 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc
Type                  : ServicePrincipal
```

## <a name="next-steps"></a>Дополнительная информация
См. следующие вводные разделы по использованию удостоверения службы фабрики данных:

- [Хранение учетных данных в Azure Key Vault](store-credentials-in-key-vault.md)
- [Копирование данных в Azure Data Lake Storage Gen1 и из него с помощью фабрики данных Azure](connector-azure-data-lake-store.md)

Дополнительные сведения об управляемых удостоверениях для ресурсов Azure, на основе которых создается идентификация фабрики данных, см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](~/articles/active-directory/msi-overview.md). 