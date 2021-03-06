---
title: включение файла
description: включение файла
services: virtual-machines
author: shants123
ms.service: virtual-machines
ms.topic: include
ms.date: 07/02/2018
ms.author: shants
ms.custom: include file
ms.openlocfilehash: efedb2f48748264fb936fe82a1dbb3cf4403cc5e
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/16/2018
ms.locfileid: "40165925"
---
## <a name="view-vms-scheduled-for-maintenance-in-the-portal"></a>Просмотр виртуальных машин, для которых запланировано обслуживание, на портале

Как только обслуживание будет запланировано, вы сможете просмотреть список виртуальных машин, которые охватит предстоящее обслуживание. 

На портале Azure можно увидеть, на каких виртуальных машинах запланировано обслуживание.

1. Войдите на [портале Azure](https://portal.azure.com).

2. В левой панели навигации щелкните **Виртуальные машины**.

3. На панели виртуальных машин нажмите кнопку **Столбцы**, чтобы открыть список доступных столбцов.

4. Выберите и добавьте следующие столбцы:

   **Обслуживание**. Отображает состояние обслуживания для виртуальной машины. Ниже приведены возможные значения.
      
      | Значение | ОПИСАНИЕ |
      |-------|-------------|
      | Start now (Запустить сейчас) | Виртуальная машина находится в периоде самообслуживания, в течение которого вы можете инициировать обслуживание самостоятельно. Ниже приведены сведения о том, как запустить обслуживание виртуальной машины. | 
      | По расписанию | Для виртуальной машины запланировано обслуживание без возможности инициировать его самостоятельно. Сведения о периоде обслуживания можно узнать, выбрав элемент "Обслуживание" — "Запланированный период" в этом представлении или щелкнув виртуальную машину. | 
      | Уже обновлено | Виртуальная машина уже обновлена и дальнейшие действия пока не требуются. | 
      | Retry later (Повторите попытку позже) | Вы инициировали обслуживание, но его не удалось выполнить. Вы сможете попытаться запустить самостоятельное обслуживание позже. | 
      | Повторите попытку сейчас | Вы можете попытаться заново инициировать обслуживание, которое завершилось неудачей ранее. | 
      | - | Ваша виртуальная машина не включена в предстоящую волну планового обслуживания. |
      

   **Обслуживание — Период самообслуживания**. Отображает период времени, в течение которого вы можете самостоятельно инициировать обслуживание виртуальных машин.
   
   **Обслуживание — Запланированное**. Отображает окно времени, в течение которого Azure автоматически выполнит обслуживание виртуальной машины. 



## <a name="notification-and-alerts-in-the-portal"></a>Уведомление и оповещения на портале

Azure извещает расписание планового обслуживания, отправив сообщение владельцу подписки или группе совладельцев. Можно добавить дополнительных получателей и каналы для получения этой информации путем создания оповещений журнала действий Azure. Дополнительные сведения см. в статье [Мониторинг действий подписки с помощью журнала действий Azure] (../articles/monitoring-and-diagnostics/monitoring-overview-activity-logs.md)

1. Войдите на [портале Azure](https://portal.azure.com).
2. В меню слева выберите **Монитор**. 
3. На панели **Монитор — Оповещения (классические)** щелкните **Добавить оповещение журнала действий**.
5. Введите данные на странице **Добавить оповещение журнала действий**, обязательно указав следующие **критерии**:
   - **Категория событий** — работоспособность службы.
   - **Службы** — масштабируемые наборы виртуальных машин и виртуальные машины.
   - **Тип**. Плановое обслуживание 
    
Дополнительные сведения о том, как настроить оповещения журнала действий, см. в статье [Создание оповещений журнала действий](../articles/monitoring-and-diagnostics/monitoring-activity-log-alerts.md).
    
    
## <a name="start-maintenance-on-your-vm-from-the-portal"></a>Запуск обслуживания вашей виртуальной машины с портала

В информации о виртуальной машине содержатся дополнительные сведения, связанные с обслуживанием.  
Если ваша виртуальная машина включена в плановое обслуживание, в верхней части представления сведения о виртуальной машине будет добавлена новая лента уведомлений. Кроме того, добавлен параметр для запуска обслуживания по возможности. 


Щелкните уведомление об обслуживании, чтобы просмотреть на странице дополнительные сведения о плановом обслуживании. Здесь вы сможете **запустить обслуживание** вашей виртуальной машины.

Как только обслуживание начнется, на виртуальной машине будут выполнены необходимые действия, а через несколько минут состояние обслуживания отразит результат этих действий.

Если вы пропустите период самообслуживания, вы сможете увидеть такое же окно в тот момент, когда Azure выполнит автоматическое обслуживание этой виртуальной машины. 
