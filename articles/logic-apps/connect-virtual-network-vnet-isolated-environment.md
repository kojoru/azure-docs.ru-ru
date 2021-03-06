---
title: Подключение к виртуальным сетям Azure из Azure Logic Apps
description: Для доступа к виртуальным сетям Azure из Azure Logic Apps можно создать частные, выделенные и изолированные среды службы интеграции, которые отделяют приложения логики и другие ресурсы от общедоступной и глобальной среды Azure.
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.topic: article
ms.date: 09/25/2018
ms.openlocfilehash: 354c31014448b914b33d2bef5483efc78092f726
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/27/2018
ms.locfileid: "47391927"
---
# <a name="create-isolated-environments-to-access-azure-virtual-networks-from-azure-logic-apps"></a>Создание изолированных сред для доступа к виртуальным сетям Azure из Azure Logic Apps

> [!NOTE]
> Эта возможность находится на этапе *закрытой предварительной версии*. Чтобы запросить доступ, [создайте запрос на присоединение](https://aka.ms/iseprivatepreview).

Для сценариев интеграции, в которых приложениям логики и учетным записям интеграции необходим доступ к [виртуальной сети Azure](../virtual-network/virtual-networks-overview.md), можно создать[ *среду службы интеграции* (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), которая связывается с вашей виртуальной сетью и развертывает в нее службу Logic Apps. При создании приложений логики и учетных записей интеграции выберите эту ISE в качестве их расположения. После этого приложения логики и учетные записи интеграции смогут напрямую получать доступ к ресурсам в виртуальной сети, таким как виртуальные машины, серверы, системы и службы. 

![Выбор среды службы интеграции](./media/connect-virtual-network-vnet-isolated-environment/select-logic-app-integration-service-environment.png)

Среда службы интеграции является частной и изолированной средой, которая использует выделенное хранилище и другие ресурсы, существующие отдельно от общедоступной и *глобальной* службы Logic Apps. Такое разделение помогает уменьшить любое влияние других клиентов Azure на производительность вашего приложения. 

В этой статье показано, как выполнять следующие задачи:

* настраивать разрешения в виртуальной сети Azure для того, чтобы частные экземпляры Logic Apps могли получать доступ к ней;

* создавать среду службы интеграции; 

* создавать приложение логики, которое может выполняться в среде службы интеграции; 

* создавать учетную запись интеграции для приложений логики в среде службы интеграции.

Дополнительные сведения о средах службы интеграции см. в статье [Доступ к ресурсам виртуальной сети Azure (VNET) из изолированной службы Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md).

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, <a href="https://azure.microsoft.com/free/" target="_blank">зарегистрируйтесь для получения бесплатной учетной записи Azure</a>. 

* Если у вас нет виртуальной сети Azure, узнайте, [как ее создать](../virtual-network/quick-create-portal.md). 

  > [!IMPORTANT]
  > Хотя для создания среды виртуальная сеть Azure не требуется, при создании этой среды в качестве однорангового узла среды можно выбрать *только* виртуальную сеть. 

* Чтобы предоставить приложениям логики прямой доступ к виртуальной сети Azure, [настройте разрешения управления доступом на основе ролей (RBAC)](#vnet-access), позволяющие службе Logic Apps получать доступ к виртуальной сети. 

* Базовые знания [создания приложений логики](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="vnet-access"></a>

## <a name="set-virtual-network-permissions"></a>Настройка разрешений виртуальной сети

При создании среды службы интеграции в Azure можно выбрать виртуальную сеть Azure как *одноранговый узел* для вашей среды. Однако при создании среды можно выполнить только этот шаг или создать *пиринг*. Эта связь позволяет службе Logic Apps напрямую подключаться к ресурсам в виртуальной сети и предоставляет среде доступ к этим ресурсам. 

Прежде чем вы сможете выбрать виртуальную сеть, в ней необходимо настроить разрешения управления доступом на основе ролей. Для этого нужно назначить конкретные роли службе Azure Logic Apps.

1. На [портале Azure](https://portal.azure.com) найдите и выберите виртуальную сеть. 

1. В меню виртуальной сети выберите **Управление доступом (IAM)**. 

1. В разделе **Управление доступом** выберите пункт **Назначение роли**, если он еще не выбран. На панели инструментов **Назначение роли** выберите **Добавить**. 

   ![Добавление назначений ролей](./media/connect-virtual-network-vnet-isolated-environment/set-up-role-based-access-control-vnet.png)

1. В области **Добавление разрешений** настройте каждую роль в этой таблице для службы Azure Logic Apps. После завершения работы с каждой ролью нужно обязательно нажимать кнопку **Сохранить**:

   | Роль | Назначение доступа для | Выберите пункт | 
   |------|------------------|--------|
   | **Участник сети** | **Пользователя, группы или приложения Azure AD** | Войдите в **Azure Logic Apps**. Когда появится список участников, выберите то же значение. <p>**Совет**. Если найти эту службу не удается, введите идентификатор приложения службы Logic Apps: `7cd684f4-8a78-49b0-91ec-6a35d38739ba` | 
   | **Участник классических сетей** | **Пользователя, группы или приложения Azure AD** | Войдите в **Azure Logic Apps**. Когда появится список участников, выберите то же значение. <p>**Совет**. Если найти эту службу не удается, введите идентификатор приложения службы Logic Apps: `7cd684f4-8a78-49b0-91ec-6a35d38739ba` | 
   |||| 

   Например: 

   ![Добавление разрешений](./media/connect-virtual-network-vnet-isolated-environment/add-contributor-roles.png)

   Дополнительные сведения о разрешениях ролей, необходимых для пиринга, см. в [разделе "Разрешения" статьи "Создание, изменение и удаление пиринга в виртуальной сети"](../virtual-network/virtual-network-manage-peering.md#permissions). 

Если виртуальная сеть подключена через Azure ExpressRoute, VPN-подключение Azure типа "точка — сеть" или "сеть — сеть", перейдите к следующему разделу, чтобы добавить необходимую подсеть шлюза. В противном случае перейдите к разделу о [создании среды](#create-environment).

<a name="add-gateway-subnet"></a>

## <a name="add-gateway-subnet-for-virtual-networks-with-expressroute-or-vpns"></a>Добавление подсети шлюза для виртуальных сетей с использованием ExpressRoute или VPN

После завершения предыдущих шагов в виртуальную сеть необходимо также добавить [*подсеть шлюза*](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsub), чтобы предоставить среде службы интеграции доступ к виртуальной сети Azure, подключенной через [Azure ExpressRoute](../expressroute/expressroute-introduction.md), [VPN типа "точка — сеть"](../vpn-gateway/point-to-site-about.md) или [VPN типа "сеть — сеть"](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) Azure:

1. На [портале Azure](https://portal.azure.com) найдите и выберите виртуальную сеть. В меню виртуальной сети выберите **Подсети**, а затем — **Подсеть шлюза** > **ОК**.

   ![Добавление подсети шлюза](./media/connect-virtual-network-vnet-isolated-environment/add-gateway-subnet.png)

1. Теперь создайте [*таблицу маршрутов*](../virtual-network/manage-route-table.md), которая будет связана с подсетью шлюза, которая была создана ранее.

   1. В главном меню Azure выберите команду **Создать ресурс** > 
   **Сети** > **Таблица маршрутов**.

      ![Создание таблицы маршрутов](./media/connect-virtual-network-vnet-isolated-environment/create-route-table.png)

   1. Укажите сведения о таблице маршрутов, такие как имя, используемая подписка Azure, группа ресурсов Azure и расположение. Убедитесь, что для свойства **распространения маршрутов BGP** установлено значение **Включено**, а затем выберите **Создать**.

      ![Указание сведений о таблице маршрутов](./media/connect-virtual-network-vnet-isolated-environment/enter-route-table-information.png)

   1. В меню таблицы маршрутов выберите **Подсети**, а затем — **Привязать**. 

      ![Подключение таблицы маршрутов к подсети](./media/connect-virtual-network-vnet-isolated-environment/associate-route-table.png)

   1. Выберите **Виртуальная сеть**, а затем — соответствующую виртуальную сеть.
   
   1. Выберите **Подсеть**, а затем — созданную ранее подсеть шлюза.

   1. Когда все будет готово, нажмите кнопку **ОК**.

1. Если вы используете VPN типа "точка — сеть", выполните следующие действия:

   1. В Azure найдите и выберите свой ресурс шлюза виртуальной сети.

   1. В меню шлюза выберите **Конфигурации "точка — сеть"**, 
   а затем выберите **Скачивание VPN-клиента**, чтобы у вас была последняя версия конфигурации VPN-клиента.

      ![Скачивание последней версии VPN-клиента](./media/connect-virtual-network-vnet-isolated-environment/download-vpn-client.png)

Теперь все готово к настройке подсети шлюза для виртуальных сетей, использующих подключение ExpressRoute, VPN типа "точка — сеть" или VPN типа "сеть — сеть". Чтобы продолжить среду службы интеграции, выполните следующие шаги.

<a name="create-environment"></a>

## <a name="create-your-ise"></a>Создание среды службы интеграции

Чтобы создать среду службы интеграции, выполните следующие действия:

1. На [портале Azure](https://portal.azure.com) в главном меню Azure выберите **Создать ресурс**.

   ![Создание ресурса](./media/connect-virtual-network-vnet-isolated-environment/find-integration-service-environment.png)

1. В поле поиска введите "среда службы интеграции" в качестве фильтра.
Выберите в списке результатов **Integration Service Environment (preview)** (Среда службы интеграции (предварительная версия)), а затем нажмите кнопку **Создать**.

   ![Выбор пункта "Среда службы интеграции"](./media/connect-virtual-network-vnet-isolated-environment/select-integration-service-environment.png)

   ![Нажатие кнопки "Создать"](./media/connect-virtual-network-vnet-isolated-environment/create-integration-service-environment.png)

1. Предоставьте такие сведения о среде:

   ![Предоставление сведений о среде](./media/connect-virtual-network-vnet-isolated-environment/integration-service-environment-details.png)

   | Свойство | Обязательно | Значение | ОПИСАНИЕ |
   |----------|----------|-------|-------------|
   | **Имя** | Yes | <*имя_среды*> | Имя, которое будет присвоено среде. | 
   | **Подписка** | Yes | <*Azure-subscription-name*> | Подписка Azure, используемая для среды. | 
   | **Группа ресурсов** | Yes | <*имя_группы_ресурсов_Azure*> | Группа ресурсов Azure, в которой необходимо создать среду. |
   | **Местоположение.** | Yes | <*регион_центра_обработки_данных_Azure*> | Регион центра обработки данных Azure, в котором будет храниться информация о среде. |
   | **Одноранговая виртуальная сеть** | Нет  | <*Имя_виртуальной_сети_Azure*> | Виртуальная сеть Azure, которая буде сопоставлена со средой в качестве *одноранговой*, чтобы приложения логики в этой среде могли получить доступ к виртуальной сети. Прежде чем создавать эту связь, убедитесь, что вы уже [настроили управление доступом на основе ролей в виртуальной сети для Azure Logic Apps](#vnet-access). <p>**Важно**. Несмотря на то что виртуальная сеть не является обязательной, вы можете выбрать ее *только* при создании среды. | 
   | **Имя пиринга** | Да, с выбранной виртуальной сетью | <*имя_пиринга*> | Имя, которое будет присвоено одноранговой связи. | 
   | **Диапазон IP-адресов виртуальной сети** | Да, с выбранной виртуальной сетью | <*диапазон_IP-адресов*> | Диапазон IP-адресов, который будет использоваться для создания ресурсов в среде. Этот диапазон должен использовать [формат бесклассовой междоменной маршрутизации CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing), например, 10.0.0.1/16, и требует пространства адресов класса В. Диапазон не должен пересекаться с адресным пространством для виртуальной сети, выбранной в свойстве **Одноранговая виртуальная сеть**, и любыми другими частными IP-адресами, к которым подключена одноранговая сеть (через пиринг или шлюзы). <p><p>**Важно**. После создания среды изменить этот диапазон адресов *нельзя*. |
   |||||
   
1. Когда все будет готово, выберите **Создать**. 

   Azure начнет развертывание среды, но выполнение этого процесса может занять *до двух часов*. 
   Чтобы проверить состояние развертывания, на панели инструментов Azure выберите значок уведомлений, который открывает область уведомлений.

   ![Проверка состояния развертывания](./media/connect-virtual-network-vnet-isolated-environment/environment-deployment-status.png)

   Когда развертывание будет успешно завершено, в Azure отобразится следующее уведомление:

   ![Развертывание выполнено](./media/connect-virtual-network-vnet-isolated-environment/deployment-success.png)

1. Если Azure не перейдет автоматически к вашей среде после завершения развертывания, выберите **Перейти к ресурсу**, чтобы просмотреть среду.  

<a name="create-logic-apps-environment"></a>

## <a name="create-logic-app---ise"></a>Создание приложения логики, использующего среду службы интеграции

Чтобы создать приложения логики, использующие среду службы интеграции (ISE), выполните обычные действия из статьи [Краткое руководство. Создание первого автоматизированного рабочего процесса с помощью Azure Logic Apps на портале Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md), но с такими отличиями и рекомендациями: 

* При создании приложения логики в свойстве **Расположение** в разделе **Среды службы интеграции** перечислены ваши среды ISE, а в соответствующем разделе указаны доступные регионы. Выберите свою среду ISE, а не регион, например:

  ![Выбор среды службы интеграции](./media/connect-virtual-network-vnet-isolated-environment/create-logic-app-with-integration-service-environment.png)

* Можно использовать те же встроенные элементы, например, HTTP-триггер или действие, которые выполняются в той же ISE, что и родительское приложение логики. Соединители с меткой **ISE** также выполняются в той же среде ISE, что и родительское приложение логики. Соединители без надписи **ISE** выполняются в глобальной службе Logic Apps.

  ![Выбор соединителей ISE](./media/connect-virtual-network-vnet-isolated-environment/select-ise-connectors.png)

* Если ранее вы настроили свою ISE с виртуальной сетью Azure в качестве однорангового узла, приложения логики в ISE могут напрямую получать доступ к ресурсам в этой виртуальной сети. Для локальных систем в виртуальной сети, которая связана со средой ISE, приложения логики могут напрямую обращаться к этим системам с помощью любого из указанных ниже элементов: 

  * Соединитель среды ISE для этой системы, например SQL Server

  * Действие HTTP 

  * Настраиваемый соединитель

  В локальных системах, которые не входят в виртуальную сеть или не имеют соединителей ISE, подключение возможно после [настройки и использования локального шлюза данных](../logic-apps/logic-apps-gateway-install.md).

<a name="create-integration-account-environment"></a>

## <a name="create-integration-account---ise"></a>Создание учетной записи интеграции для среды службы интеграции

Чтобы использовать учетную запись интеграции с приложениями логики в среде службы интеграции (ISE), эта учетная запись должна использовать *ту же среду*, что и приложения логики. Приложения логики в среде ISE могут ссылаться только на учетные записи интеграции, которые находятся в одной с ними среде ISE. 

Чтобы создать учетную запись интеграции, которая использует ISE, выполните обычные действия из статьи [Создание и администрирование учетных записей интеграции для решений B2B в приложениях логики](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md), пропустив действия для свойства **Расположение**, в котором теперь перечислены ваши ISE в разделе **Среды службы интеграции**, а также доступные регионы. Выберите свою среду ISE, а не регион, например:

![Выбор среды службы интеграции](./media/connect-virtual-network-vnet-isolated-environment/create-integration-account-with-integration-service-environment.png)

## <a name="get-support"></a>Получение поддержки

* Если у вас возникли вопросы, то посетите <a href="https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurelogicapps" target="_blank">форум Azure Logic Apps</a>.
* Отправить идею по поводу возможности или проголосовать за нее вы можете на <a href="http://aka.ms/logicapps-wish" target="_blank">сайте отзывов пользователей Logic Apps</a>.

## <a name="next-steps"></a>Дополнительная информация

* Узнайте больше о [виртуальной сети Azure](../virtual-network/virtual-networks-overview.md).
* Узнайте об [интеграции виртуальной сети для служб Azure](../virtual-network/virtual-network-for-azure-services.md).
