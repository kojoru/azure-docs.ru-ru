---
title: включение файла
description: IoT Edge
author: kgremban
manager: timlt
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 09/18/2018
ms.author: kgremban
ms.openlocfilehash: a8160e677fa99d8cb691db39d7f29ba6eddbd261
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/24/2018
ms.locfileid: "47004685"
---
## <a name="enabling-extended-offline-operation-preview"></a>Включение расширенного автономного режима (предварительная версия)
Начиная с [выпуска 1.0.2](https://aka.ms/edge102) среды выполнения Edge устройство Edge и нисходящие устройства, подключающиеся к нему, можно настроить для использования расширенного автономного режима. 

Благодаря этой возможности локальные модули или нисходящие устройства при необходимости могут повторно пройти аутентификацию в устройстве Edge и взаимодействовать между собой, используя сообщения и методы, даже в случае отключения от Центра Интернета вещей. Прочитайте эту [запись блога](https://aka.ms/iot-edge-offline) и [концептуальную статью](../articles/iot-edge/offline-capabilities.md), чтобы изучить дополнительные сведения и область применения данной возможности.

Чтобы включить расширенный автономный режим в сценарии со шлюзом, установите отношения "родитель — дочерний элемент" между устройством Edge и нисходящими устройствами, которые будут к нему подключаться.

1. В колонке сведений об устройстве Edge на портале Центра Интернета вещей нажмите кнопку **Manage Child Devices (preview)** (Управление дочерними устройствами (предварительная версия)), расположенную на панели команд вверху.

1. Щелкните **+ Добавить**.

1. Из списка устройств с помощью стрелки вправо выберите те из них, которые нужно добавить в качестве дочерних.

1. Нажмите кнопку **ОК** для подтверждения.

1. На странице сведений об устройстве Edge **Задание модулей** щелкните **Настройка дополнительных параметров среды выполнения Edge**. В разделе переменных среды **Центр Edge** добавьте запись  **UpstreamProtocol** со значением **MQTT**. Добавьте такую же переменную среды и значение для **агента Edge**. 

1. Нажмите кнопку **Сохранить** и обязательно **отправьте** изменения после двух нажатий кнопки **Далее**.

Теперь устройство Edge и его дочерние устройства могут работать в расширенном автономном режиме.  