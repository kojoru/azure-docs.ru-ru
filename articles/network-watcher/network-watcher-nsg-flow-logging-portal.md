---
title: Руководство по регистрации потока сетевого трафика на виртуальную машину и с нее с помощью портала Azure | Документация Майкрософт
description: Дополнительные сведения о регистрации потока сетевого трафика на виртуальную машину и обратно с помощью функции журналов потока NSG службы "Наблюдатель за сетями".
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
Customer intent: I need to log the network traffic to and from a VM so I can analyze it for anomalies.
ms.assetid: 01606cbf-d70b-40ad-bc1d-f03bb642e0af
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/30/2018
ms.author: jdial
ms.custom: mvc
ms.openlocfilehash: f010bebcf1130b3061c60987ffbd4e706a030773
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/20/2018
ms.locfileid: "41920621"
---
# <a name="tutorial-log-network-traffic-to-and-from-a-virtual-machine-using-the-azure-portal"></a>Руководство по регистрации потока сетевого трафика на виртуальную машину и с нее с помощью портала Azure

Группа безопасности сети (NSG) позволяет выполнять фильтрацию входящего и исходящего трафика на виртуальную машину и с нее. Вы можете регистрировать сетевой трафик, проходящий через NSG, с помощью функции журналов потока NSG службы "Наблюдатель за сетями". Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание виртуальной машины с помощью группы безопасности сети.
> * Включение службы "Наблюдатель за сетями" и регистрация поставщика Microsoft.Insights.
> * Включение регистрации потока трафика для NSG, используя функцию журнала потока NSG службы "Наблюдатель за сетями".
> * Скачивание зарегистрированных в журнале данных.
> * Просмотр зарегистрированных в журнале данных.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="create-a-vm"></a>Создание виртуальной машины

1. Выберите **+ Создать ресурс** в верхнем левом углу окна портала Azure.
2. Выберите **Вычисление**, а затем — **Windows Server 2016 Datacenter** или виртуальную машину **Ubuntu Server 17.10**.
3. Введите или выберите следующие значения, примите значения по умолчанию для остальных параметров и нажмите кнопку **ОК**:

    |Параметр|Значение|
    |---|---|
    |ИМЯ|myVm|
    |Имя пользователя| Введите выбранное имя пользователя.|
    |Пароль| Введите выбранный пароль. Пароль должен включать минимум 12 символов и соответствовать [определенным требованиям к сложности](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Подписка| Выберите свою подписку.|
    |Группа ресурсов| Выберите **Создать новую**, а затем введите **myResourceGroup**.|
    |Расположение| Выберите **Восточная часть США**.|

4. Выберите размер виртуальной машины и щелкните **Выбрать**.
5. На странице **Параметры** примите все значения по умолчанию и нажмите кнопку **ОК**.
6. В разделе **создания** на странице **Сводка** выберите **Создать**, чтобы начать развертывание виртуальной машины. Развертывание виртуальной машины занимает несколько минут. Дождитесь окончания развертывания виртуальной машины, прежде чем приступить к выполнению следующих шагов.

Создание виртуальной машины занимает несколько минут. Выполняйте последующие шаги только после создания виртуальной машины. Во время создания виртуальной машины на портале также создается группа безопасности сети с именем **myVm-nsg** и выполняется ее привязка к сетевому интерфейсу для виртуальной машины.

## <a name="enable-network-watcher"></a>Включение Наблюдателя за сетями

Если в регионе "Восточная часть США" уже включена служба "Наблюдатель за сетями", перейдите к разделу о [регистрации поставщика Microsoft Insights](#register-insights-provider).

1. На портале выберите **Все службы**. В текстовом поле **Фильтр** введите *Наблюдатель за сетями*. Когда в результатах появится **Наблюдатель за сетями**, выберите его.
2. Выберите раздел **Регионы**, чтобы развернуть его, а затем щелкните **...** справа от региона **Восточная часть США**, как показано на следующем изображении:

    ![Включение Наблюдателя за сетями](./media/network-watcher-nsg-flow-logging-portal/enable-network-watcher.png)

3. Выберите **Включить Наблюдатель за сетями**.

## <a name="register-insights-provider"></a>Регистрация поставщика Microsoft Insights

Для регистрации потока NSG требуется поставщик **Microsoft.Insights**. Чтобы зарегистрировать поставщик, выполните следующие шаги:

1. В верхнем левом углу портала выберите **Все службы**. В поле фильтра введите *Подписки*. Когда в результатах поиска появится пункт **Подписки**, выберите его.
2. Из списка подписок выберите ту, для которой необходимо включить поставщик.
3. В разделе **Settings** (Параметры) выберите **Поставщики ресурсов**.
4. Убедитесь, что для поставщика **microsoft.insights** задано **состояние** **Зарегистрировано**, как показано на следующем изображении. Если состояние — **Не зарегистрировано**, выберите **Зарегистрировать** справа от поставщика.

    ![Регистрация поставщика](./media/network-watcher-nsg-flow-logging-portal/register-provider.png)

## <a name="enable-nsg-flow-log"></a>Включение журнала потоков NSG

1. Данные журнала потоков NSG записываются в учетную запись хранения Azure. Чтобы создать учетную запись хранения Azure, выберите **+ Создать ресурс** в верхнем левом углу портала.
2. Выберите **Хранилище**, а затем **Учетная запись хранения — BLOB-объект, файл, таблица, очередь**.
3. Введите или выберите приведенные ниже сведения, примите значения по умолчанию для остальных параметров и щелкните **Создать**.

    | Параметр        | Значение                                                        |
    | ---            | ---   |
    | ИМЯ           | Должно состоять из 3–24 символов, может содержать только буквы нижнего регистра и цифры и должно быть уникальным среди всех учетных записей хранения Azure.                                                               |
    | Расположение       | Выберите **Восточная часть США**.                                           |
    | Группа ресурсов | Щелкните **Use existing** (Использовать существующую), а затем выберите **myResourceGroup**. |

    Создание учетной записи хранения может занять примерно минуту. Выполняйте последующие шаги только после создания учетной записи хранения. Если вы использовали имеющуюся учетную запись хранения, а не создали новую, убедитесь, что выбрали учетную запись, для которой установлен параметр **Все сети** (по умолчанию) в области **Брандмауэры и виртуальные сети** раздела **Settings** (Параметры).
4. В верхнем левом углу портала выберите **Все службы**. В текстовом поле **Фильтр** введите *Наблюдатель за сетями*. Когда в результатах поиска появится **Наблюдатель за сетями**, выберите его.
5. В разделе **Журналы** выберите **Журналы последовательностей NSG**, как показано на следующем изображении.

    ![Группы NSG](./media/network-watcher-nsg-flow-logging-portal/nsgs.png)

6. В списке NSG выберите NSG с именем **myVm-nsg**.
7. В разделе **Параметры журналов последовательностей** выберите **On** (Включить).
8. Выберите учетную запись хранения, созданную во время выполнения шага 3.
9. Задайте в разделе **Хранение (дни)** значение 5, а затем выберите **Save** (Сохранить).

## <a name="download-flow-log"></a>Скачивание журналов потоков

1. В службе "Наблюдатель за сетями" на портале выберите **Журналы последовательностей NSG** в разделе **Журналы**.
2. Выберите **You can download flow logs from configured storage accounts** (Журналы потоков можно скачать в настроенных учетных записях хранения), как показано на следующем изображении.

  ![Скачивание журналов потоков](./media/network-watcher-nsg-flow-logging-portal/download-flow-logs.png)

3. Выберите учетную запись хранения, настроенную на шаге 2 в разделе о [включении журнала потоков NSG](#enable-nsg-flow-log).
4. Выберите **Контейнеры**в разделе **Служба BLOB-объектов** и выберите контейнер **insights-logs-networksecuritygroupflowevent**, как показано на следующем изображении:

    ![Выбор контейнера](./media/network-watcher-nsg-flow-logging-portal/select-container.png)
5. Перейдите к иерархии папок и найдите файл PT1H.json, как показано на следующем изображении:

    ![Файл журнала](./media/network-watcher-nsg-flow-logging-portal/log-file.png)

    Файлы журналов записываются в иерархию папок, которая соответствует следующему соглашению об именовании: https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json

6. Выберите **...** справа от файла PT1H.json и щелкните **Download** (Скачать).

## <a name="view-flow-log"></a>Просмотр журнала потоков

В следующем файле Json представлен пример отображаемых данных в файле PT1H.json для каждого потока, для которого зарегистрированы данные:

```json
{
    "time": "2018-05-01T15:00:02.1713710Z",
    "systemId": "<Id>",
    "category": "NetworkSecurityGroupFlowEvent",
    "resourceId": "/SUBSCRIPTIONS/<Id>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/MYVM-NSG",
    "operationName": "NetworkSecurityGroupFlowEvents",
    "properties": {
        "Version": 1,
        "flows": [{
            "rule": "UserRule_default-allow-rdp",
            "flows": [{
                "mac": "000D3A170C69",
                "flowTuples": ["1525186745,192.168.1.4,10.0.0.4,55960,3389,T,I,A"]
            }]
        }]
    }
}
```

Значение, заданное для **mac** в предыдущих выходных данных — MAC-адрес сетевого интерфейса, созданного при генерировании виртуальной машины. Данные, разделенные запятыми в разделе **flowTuples**, выглядят следующим образом:

| Демонстрационные данные | Что представляют собой данные   | Пояснение                                                                              |
| ---          | ---                    | ---                                                                                      |
| 1525186745   | Метка времени             | Метка времени возникновения потока в формате UNIX EPOCH. В предыдущем примере дата преобразуется в 1 мая 2018 г., 2:59:05 (GMT).                                                                                    |
| 192.168.1.4  | Исходный IP-адрес      | Исходный IP-адрес, с которого поступил поток.
| 10.0.0.4     | Конечный IP-адрес | IP-адрес назначения, на который был отправлен поток. 10.0.0.4 — частный IP-адрес виртуальной машины, созданной в [этом разделе](#create-a-vm).                                                                                 |
| 55960        | Исходный порт            | Исходный порт, с которого был отправлен поток.                                           |
| 3389         | Конечный порт       | Порт назначения, на который был отправлен поток. Так как трафик был отправлен на порт 3389, файл был обработан с помощью правила с именем **UserRule_default-allow-rdp**.                                                |
| T            | Протокол               | Какой протокол использовался для передачи потока: TCP (T) или UDP (U).                                  |
| I            | Направление              | Тип трафика: входящий (I) или исходящий (O).                                     |
| A            | Действие                 | Разрешен (A) или запрещен (D) трафик.                                           |

## <a name="next-steps"></a>Дополнительная информация

В этом руководстве вы узнали, как включить ведение журнала NSG для потока NSG. Вы также узнали, как скачать и просмотреть данные, записанные в файл. Необработанные данные в JSON-файле могут быть сложными для понимания. Чтобы визуализировать данные, можно использовать [аналитику трафика](traffic-analytics.md) службы "Наблюдатель за сетями", Microsoft [PowerBI](network-watcher-visualize-nsg-flow-logs-power-bi.md) и другие средства.