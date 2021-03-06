---
title: Создание решения для географически распределенного приложения с помощью Azure и Azure Stack | Документация Майкрософт
description: Узнайте, как создать решение для географически распределенного приложения с помощью Azure и Azure Stack.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: mabrigg
ms.reviewer: Anjay.Ajodha
ms.openlocfilehash: 19e7506dac82e4d12d5aecbdb5ae1c14fb944c29
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/24/2018
ms.locfileid: "46961541"
---
# <a name="tutorial-create-a-geo-distributed-app-solution-with-azure-and-azure-stack"></a>Руководство по созданию решения для географически распределенного приложения с помощью Azure и Azure Stack

*Область применения: интегрированные системы Azure Stack и Пакет средств разработки Azure Stack*

Узнайте, как в шаблоне географически распределенного приложения направлять трафик к определенным конечным точкам на основе различных метрик. Создав профиль диспетчера трафика с маршрутизацией по географическим данным и конфигурацией конечных точек, можно направлять данные к определенным конечным точкам с учетом региональных требований, корпоративных и международных стандартов и ваших потребностей по обработке данных.

В рамках этого руководства вы создадите пример среды и выполните в ней следующие действия:

> [!div class="checklist"]
> - создание географически распределенного приложения;
> - использование диспетчера трафика для приложения.

## <a name="use-the-geo-distributed-apps-pattern"></a>Использование шаблона географически распределенных приложений

Шаблон географически распределенного приложения позволяет распространить приложение на несколько регионов. Вы можете по умолчанию использовать общедоступное облако, а для некоторых пользователей сохранить данные в домашнем регионе. Также можно направлять пользователей в подходящее облако на основании их требований.

### <a name="issues-and-considerations"></a>Проблемы и рекомендации

#### <a name="scalability-considerations"></a>Вопросы масштабируемости

Создаваемое в рамках этого руководства решение не включает механизмов масштабирования. Но его можно сочетать с другими локальными или облачными технологиями и решениями, чтобы учесть требования к масштабируемости. Сведения о создании гибридного решения для автоматического масштабирования в нескольких облаках в Azure с помощью диспетчера трафика вы найдете в [этом руководстве](azure-stack-solution-cloud-burst.md).

#### <a name="availability-considerations"></a>Вопросы доступности

Как и в отношении масштабируемости, это решение не ориентировано непосредственно на обеспечение доступности. Но здесь также можно применить локальные и облачные технологии и решения, чтобы обеспечить высокий уровень доступности всех используемых компонентов.

### <a name="when-to-use-this-pattern"></a>Когда следует использовать этот шаблон

- У вашей организации есть международные филиалы, для которых применяются региональные политики безопасности и распространения.

- Каждый из офисов организации собирает данные о сотрудниках, бизнесе и предприятии, для чего нужно создавать отчеты с соблюдением местных требований и часовых поясов.

- Высокого уровня масштабируемости можно добиться за счет горизонтального масштабирования, при котором несколько приложений развертываются в одном или нескольких регионах.

### <a name="planning-the-topology"></a>Планирование топологии

Прежде чем создавать распределенную топологию, следует собрать некоторые сведения.

-   **Именной домен приложения.** Какое имя домена клиенты будут использовать для доступа к приложению? В нашем примере используется имя личного домена *www.scalableasedemo.com*

-   **Домен диспетчера трафика.** При создании [профиля диспетчера трафика Azure](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles) необходимо выбрать доменное имя. При регистрации записи домена, используемой диспетчером трафика, к этому имени будет добавлен суффикс *trafficmanager.net*. В нашем примере приложения используется имя *scalable-ase-demo*, поэтому полное имя домена, управляемое диспетчером трафика, — *scalable-ase-demo.trafficmanager.net*.

-   **Стратегия масштабирования приложения.** Как будет распределяться нагрузка между несколькими средами службы приложений? Они будут размещаться в одном регионе, в нескольких регионах, или эти подходы будут комбинироваться? Решение должно основываться на предполагаемых источниках клиентского трафика, а также на характеристиках масштабируемости серверной инфраструктуры приложения. Например, приложение, работающее без отслеживания состояния, можно масштабировать, развернув конфигурацию из нескольких сред Службы приложений в одном регионе, а также в нескольких регионах Azure. Сейчас доступно более 15 глобальных регионов Azure. Поэтому клиенты могут создать по-настоящему глобальное гипермасштабируемое приложение. Для приложения, рассматриваемого в этой статье, создано три среды службы приложений в одном регионе Azure (центрально-южная часть США).

-   **Соглашение об именовании для сред Службы приложений.** Для каждой такой среды требуется уникальное имя. Если используется больше двух сред службы приложений, для их идентификации желательно использовать определенное соглашение об именовании. В нашем примере используется простое соглашение об именовании: Среды службы приложений называются *fe1ase*, *fe2ase* и *fe3ase*.

-   **Соглашение об именовании приложений.** Так как предполагается развертывание нескольких экземпляров приложения, каждому из них необходимо присвоить уникальное имя. В разных средах Службы приложений можно использовать одни и те же имена. Так как у каждой среды Службы приложений есть уникальный доменный суффикс, разработчики могут использовать одно и то же имя приложения во всех средах. Например, разработчик может назвать приложения так: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* и т. п. В этом примере каждый экземпляр приложения имеет уникальное имя. *webfrontend1*, *webfrontend2* и *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/azure-stack-solution-cloud-burst/hybrid-pillars.png)  
> Microsoft Azure Stack — это расширение Azure.re. Azure Stack обеспечивает гибкость и высокую скорость внедрения инноваций облачной среды в локальной среде. Только это гибридное облако позволяет создавать и развертывать гибридные приложения в любой точке мира.  
> 
> В техническом документе [Design Considerations for Hybrid Applications](https://aka.ms/hybrid-cloud-applications-pillars) (Рекомендации по проектированию гибридных приложений) рассматриваются основные аспекты качества программного обеспечения (размещение, масштабируемость, доступность, устойчивость, управляемость и безопасность) для разработки, развертывания и эксплуатации гибридных приложений. Эти рекомендации помогут оптимизировать разработку гибридных приложений и свести к минимуму проблемы в рабочих средах.

## <a name="part-1-create-a-geo-distributed-app"></a>Часть 1. Создание географически распределенного приложения

В рамках этого раздела вы создадите веб-приложение.

> [!div class="checklist"]
> - Создание и публикация веб-приложения
> - Добавление кода в проект VSTS
> - Настройте сборку приложения на несколько целевых объектов в облаке.
> - Настройка процесса непрерывного развертывания и управление им

### <a name="prerequisites"></a>Предварительные требования

Требуются подписка Azure и установка Azure Stack.

### <a name="geo-distributed-app-steps"></a>Шаги по созданию географически распределенного приложения

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Получение личного домена и настройка DNS

Обновление файла зоны DNS. Создание веб-приложений и публикация домена. Azure AD сможет проверить принадлежность имени личного домена. Вы можете использовать [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) для записей Azure, Office 365 и внешних записей DNS в Azure или добавить запись DNS [в другой регистратор DNS](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Зарегистрируйте личный домен в общедоступном регистраторе.

2. Войдите в соответствующий регистратор доменных имен. Для обновления DNS могут потребоваться права администратора.

3. Обновите файл зоны DNS для соответствующего домена, добавив предоставленную службой Azure AD DNS-запись. Эта DNS-запись не влияет на поведение работающих систем, например на маршрутизацию почты или веб-хостинг.

### <a name="create-web-apps-and-publish"></a>Создание и публикация веб-приложения

Настройте гибридный конвейер непрерывной интеграции и непрерывной поставки, чтобы развернуть веб-приложение в Azure и Azure Stack, а также автоматически отправлять изменения в оба облака.

> [!Note]  
> Вам необходим Azure Stack с подходящими образами, объединенными для запуска (Windows Server и SQL), и развернутой службой приложений. Дополнительные сведения об операторе Azure Stack см. в документации по Службе приложений в разделе о [подготовке к работе в Azure Stack](/articles/azure-stack/azure-stack-app-service-before-you-get-started).

#### <a name="add-code-to-vsts-project"></a>Добавление кода в проект VSTS

1. Войдите в Visual Studio, используя **учетную запись, которая предоставляет права на создание проекта** в VSTS.

    Гибридное решение для непрерывной интеграции и непрерывной поставки (CI/CD) можно применить как для кода приложения, так и для кода инфраструктуры. Используйте [шаблоны Azure Resource Manager](https://azure.microsoft.com/resources/templates/) для частной и облачной разработки.

    ![Alt text](media\azure-stack-solution-geo-distributed\image1.JPG)

2. **Клонируйте репозиторий** путем создания и открытия веб-приложения по умолчанию.

    ![Alt text](media\azure-stack-solution-geo-distributed\image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Создание развертывания веб-приложений в обоих облаках

1.  Измените файл **WebApplication.csproj**: выберите **Runtimeidentifier** и добавьте **win10-x64**. (См. документацию по [автономному развертыванию](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd)).

    ![Alt text](media\azure-stack-solution-geo-distributed\image3.png)

1.  **Добавьте код в VSTS** с помощью Team Explorer.

2.  Убедитесь, что **код приложения** добавлен в Visual Studio Team Services.

### <a name="create-the-build-definition"></a>Создание определения сборки

1. **Войдите в VSTS**, чтобы подтвердить возможность создания определений сборки.

2. Добавьте код **-r win10-x64**. Это необходимо, чтобы активировать автономное развертывание с .Net Core.

    ![Alt text](media\azure-stack-solution-geo-distributed\image4.png)

3. **Запустите сборку**. Процесс [сборки автономного развертывания](https://docs.microsoft.com/dotnet/core/deploying/#self-contained-deployments-scd) будет публиковать артефакты, которые могут выполняться в Azure и Azure Stack.

**Использование агента, размещенного в Azure**

Использование размещенного агента в VSTS удобно для создания и развертывания веб-приложений. Обновление и обслуживание, включая постоянную, непрерывную разработку, тестирование и развертывание, автоматически выполняются Microsoft Azure.

### <a name="manage-and-configure-the-cd-process"></a>Настройка процесса непрерывного развертывания и управление им

Visual Studio Team Services и Team Foundation Server предоставляют конвейер с широкими возможностями настройки и управления для выпусков в различные среды, такие как среда развертывания, промежуточная среда, среда для контроля качества и рабочая среда, включая получение утверждений на определенных стадиях.

#### <a name="create-release-definition"></a>Создание определения выпуска


![Alt text](media\azure-stack-solution-geo-distributed\image5.png)

1.  Нажмите кнопку со знаком **плюс**, чтобы добавить новый выпуск на **вкладку выпусков** на странице сборки и выпуска Visual Studio Online (VSO).

    ![Alt text](media\azure-stack-solution-geo-distributed\image6.png)

2. Примените шаблон **развертывания службы приложений Azure**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image7.png)

3. В раскрывающемся меню добавления артефакта **выберите соответствующую команду** для приложения сборки облака Azure.

    ![Alt text](media\azure-stack-solution-geo-distributed\image8.png)

4. На вкладке "Конвейер" выберите ссылку **Phase, Task** (Фаза, Задача) для используемой среды и задайте значения облачной среды Azure.

    ![Alt text](media\azure-stack-solution-geo-distributed\image9.png)

5. Задайте **имя среды** и выберите **подписку** Azure для конечной точки облака Azure.

    ![Alt text](media\azure-stack-solution-geo-distributed\image10.png)

6. В разделе имени среды задайте требуемое **имя службы приложений Azure**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image11.png)

7. Введите **Размещенная среда VS2017** в очереди агента для среды, размещенной в облаке Azure.

    ![Alt text](media\azure-stack-solution-geo-distributed\image12.png)

8. В меню развертывания службы приложений Azure выберите допустимый для среды **пакет или папку**. Нажмите кнопку "ОК", чтобы выбрать **расположение папки**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image13.png)

    ![Alt text](media\azure-stack-solution-geo-distributed\image14.png)

9. Сохраните все изменения и вернитесь к **конвейеру выпуска**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image15.png)

10. Добавьте **новый артефакт**, выбрав сборку для приложения Azure Stack.

    ![Alt text](media\azure-stack-solution-geo-distributed\image16.png)

11. Добавьте еще одну среду, применив **развертывание службы приложений Azure**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image17.png)

12. Назовите новую среду **Azure Stack**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image18.png)

13. Найдите среду Azure Stack на вкладке **Задача**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image19.png)

14. Выберите **подписку** для конечной точки Azure Stack.

  ![Alt text](media\azure-stack-solution-geo-distributed\image20.png)

15. Задайте имя веб-приложения Azure Stack в качестве **имени службы приложений**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image21.png)

16. Выберите **агент Azure Stack**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image22.png)

17. В разделе развертывания службы приложений Azure выберите допустимый для среды **пакет или папку**. Нажмите кнопку "ОК", чтобы выбрать **расположение папки**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image23.png)

    ![Alt text](media\azure-stack-solution-geo-distributed\image24.png)

18. В разделе **Переменные** добавьте переменную с именем `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, задайте для нее значение `true` и область `Azure Stack`.

    ![Alt text](media\azure-stack-solution-geo-distributed\image25.png)

19. Выберите значок триггера **непрерывного** развертывания в обоих артефактах и включите триггер **непрерывного** развертывания.

    ![Alt text](media\azure-stack-solution-geo-distributed\image26.png)

20. Выберите значок условий **перед развертыванием** в среде Azure Stack и задайте триггеру значение **После выпуска**.

21. Сохраните все изменения.

> [!Note]  
>  Некоторые параметры для задач могли быть автоматически определены как [переменные среды](https://docs.microsoft.com/vsts/build-release/concepts/definitions/release/variables?view=vsts#custom-variables) при создании определения выпуска на основе шаблона. Эти параметры невозможно изменить в параметрах задачи. Чтобы изменить их, необходимо выбрать родительский элемент среды.

## <a name="part-2-update-web-app-options"></a>Часть 2. Методы обновления веб-приложения

[Веб-приложения Azure](https://docs.microsoft.com/azure/app-service/app-service-web-overview) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. 

![Alt text](media\azure-stack-solution-geo-distributed\image27.png)

> [!div class="checklist"]
> - Сопоставление существующего настраиваемого DNS-имени с веб-приложениями Azure
> - Чтобы сопоставить настраиваемое DNS-имя со службой приложений, используйте **запись CNAME или **запись A**.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Сопоставление существующего настраиваемого DNS-имени с веб-приложениями Azure

> [!Note]  
>  Мы рекомендуем использовать записи CNAME для всех настраиваемых DNS-имен, кроме корневого домена (например, northwind.com).

Сведения о том, как перенести активный веб-сайт и его DNS-имя домена в службу приложений, см. в статье [Перенос активного DNS-имени в службу приложений Azure](https://docs.microsoft.com/azure/app-service/app-service-custom-domain-name-migrate).

### <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством:

-   [Создайте приложение службы приложений](https://docs.microsoft.com/azure/app-service/) или используйте приложение, созданное для работы с другим руководством.

-   Приобретите доменное имя и предоставьте поставщику домена доступ к реестру DNS.

Обновите файл зоны DNS для домена. Azure AD проверит принадлежность имени личного домена. Вы можете использовать [Azure DNS](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) для записей Azure, Office 365 и внешних записей DNS в Azure или добавить запись DNS [в другой регистратор DNS](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

-   Зарегистрируйте личный домен в общедоступном регистраторе.

-   Войдите в соответствующий регистратор доменных имен. (Для обновления DNS могут потребоваться права администратора.)

-   Обновите файл зоны DNS для соответствующего домена, добавив предоставленную службой Azure AD DNS-запись.

Например, чтобы добавить записи DNS www.northwindcloud.com fornorthwindcloud.comand, настройте параметры DNS для корневого домена thenorthwindcloud.com.

> [!Note]  
>  Доменное имя можно приобрести через [портал Azure](https://docs.microsoft.com/azure/app-service/custom-dns-web-site-buydomains-web-app).  
> Чтобы сопоставить настраиваемое DNS-имя с веб-приложением, его уровень [плана службы приложений](https://azure.microsoft.com/pricing/details/app-service/) должен быть платным (**Общий**, **Базовый**, **Стандартный** или **Премиум**).



### <a name="create-and-map-cname-and-a-records"></a>Создание и сопоставление записей CNAME и A

#### <a name="access-dns-records-with-domain-provider"></a>Доступ к записям DNS с помощью поставщика домена

> [!Note]  
>  С помощью Azure DNS настройте пользовательское DNS-имя для веб-приложений Azure. Дополнительные сведения см. в статье [Использование Azure DNS для указания параметров личного домена для службы Azure](https://docs.microsoft.com/azure/dns/dns-custom-domain).

1.  Войдите на веб-сайт основного поставщика домена.

2.  Найдите страницу управления записями DNS. Каждый поставщик доменов имеет собственный интерфейс для записей DNS. Найдите области сайта, обозначенные как **Имя домена**, **DNS** или **Name Server Management** (Управление сервером доменных имен).

Страницу управления записями DNS можно открыть из раздела **Мои домены**. Воспользуйтесь ссылкой **Файл зоны**, **Записи DNS** или **Расширенная конфигурация**.

На снимке экрана ниже показан пример страницы с записями DNS:

![Пример страницы с записями DNS](media\azure-stack-solution-geo-distributed\image28.png)

1.  На странице регистратора доменных имен выберите команду **"Добавить" или "Создать"**, чтобы создать запись. Некоторые поставщики имеют разные ссылки для добавления записей различных типов. Обратитесь к документации поставщика.

2.  Добавьте запись CNAME, чтобы сопоставить поддомен с именем узла по умолчанию для приложения.

  Например, для домена www.northwindcloud.com добавьте запись CNAME, которая сопоставляет namewwwto<app\_name>.azurewebsites.net.

После добавления этой записи CNAME страница управления записями DNS выглядит так:

![Переход к приложению Azure на портале](media\azure-stack-solution-geo-distributed\image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Включение сопоставления записи CNAME в приложении Azure

1.  В новой вкладке войдите на портал Azure.

2.  Перейдите к веб-службам.

3.  Выберите веб-приложение.

4.  В левой области навигации страницы приложения на портале Azure выберите **Личные домены**.

5.  Щелкните значок **+** рядом с параметром **Добавить имя узла**.

1.  Введите полное доменное имя, например `www.northwindcloud.com`.

2.  Выберите **Проверка**.

3.  Если это указано отдельно, добавьте в DNS-записи регистратора доменных имен дополнительные записи других типов (обычно `A` или `TXT`). Azure предоставит значения и типы для этих записей:

    a.  Запись **A** для сопоставления с IP-адресом приложения.

    b.  Запись типа **TXT** для сопоставления с именем узла по умолчанию для приложения: hostname <app_name>.azurewebsites.net. Служба приложений использует эту запись только во время настройки, чтобы убедиться, что вы являетесь владельцем личного домена. После проверки удалите запись типа TXT.

4.  Выполните эту задачу на вкладке регистратора домена и повторите проверку, чтобы активировать кнопку **Добавить имя узла**.

5.  Убедитесь, что для **типа записи имени узла выбрано значение **CNAME (www.example.com или любой поддомен)**.

6.  Выберите **Добавить имя узла**.

7.  Введите полное доменное имя, например `northwindcloud.com`.

8.  Выберите **Проверка**.

9.  Теперь активируется **кнопка добавления**.

10. Убедитесь, что для **типа записи имени узла выбрано значение **Запись А (example.com)**.

11. **Добавление имени узла**.

  Возможно, потребуется некоторое время, чтобы новое имя узла отобразилось на странице **Личные домены** вашего приложения. Попробуйте обновить браузер, чтобы обновить данные.
  
  ![Alt text](media\azure-stack-solution-geo-distributed\image31.png) 
  
  В случае ошибки в нижней части страницы появится уведомление об ошибке проверки. ![Ошибка проверки](media\azure-stack-solution-geo-distributed\image32.png)

> [!Note]  
>  Описанные выше действия можно повторить для сопоставления доменного имени с подстановочным знаком (\*. northwindcloud.com). Это позволяет добавлять любые поддомены к этой службе приложений, не создавая для каждого из них отдельную запись CNAME. Следуйте инструкциям регистратора по настройке этого параметра.

#### <a name="test-in-a-browser"></a>Проверка в браузере

Откройте в браузере DNS-имя (или имена), которое вы настроили ранее (например, `northwindcloud.com` или www.northwindcloud.com).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Часть 3. Привязка настраиваемого SSL-сертификата

В этой части вы выполните следующие задачи:

> [!div class="checklist"]
> - привязывание пользовательского SSL-сертификата к службе приложений;
> - принудительное использование HTTPS для приложения;
> - Автоматизация привязки SSL-сертификата с помощью скриптов.

> [!Note]  
> Если потребуется, получите SSL-сертификат для клиента на портале Azure и привяжите его к веб-приложению. Следуйте указаниям в [руководстве по сертификатам службы приложений](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством:

-   [Создано приложение службы приложений](https://docs.microsoft.com/azure/app-service/).
-   [С вашим веб-приложением сопоставлено настраиваемое DNS-имя](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain).
-   Получите SSL-сертификат из доверенного центра сертификации и подпишите запрос этим ключом.

### <a name="requirements-for-your-ssl-certificate"></a>Требования к SSL-сертификату

Чтобы сертификат можно было использовать в службе приложений, он:

-   должен быть подписан доверенным центром сертификации;

-   должен быть экспортирован в PFX-файл, защищенный паролем;

-   должен содержать закрытый ключ длиной не менее 2048 битов;

-   должен содержать все промежуточные сертификаты из цепочки сертификатов.

> [!Note]  
>  **Сертификаты с шифрованием на основе эллиптических кривых (ECC)** совместимы со Службой приложений, но не рассматриваются в этой статье. Обратитесь в центр сертификации за помощью в создании сертификата ECC. 

#### <a name="prepare-the-web-app"></a>Подготовка веб-приложения

Чтобы привязать SSL-сертификат к веб-приложению, ваш [план службы приложений](https://azure.microsoft.com/pricing/details/app-service/) должен относиться к категории **Базовый**, **Стандартный** или **Премиум**.

#### <a name="sign-in-to-azure"></a>Вход в Azure

1.  Войдите на [портал Azure](https://portal.azure.com/) и перейдите к своему веб-приложению.

2.  В меню слева выберите **Службы приложений** и щелкните имя нужного приложения Azure.

![Выбор веб-приложения](media\azure-stack-solution-geo-distributed\image33.png)

#### <a name="check-the-pricing-tier"></a>Проверка ценовой категории

1.  В области навигации слева на странице веб-приложения перейдите к разделу **Параметры** и выберите **Увеличить масштаб (план службы приложений)**.

    ![Меню увеличения масштаба](media\azure-stack-solution-geo-distributed\image34.png)

1.  Убедитесь, что веб-приложение не относится к категории **Бесплатный** или **Общий**. Текущая категория веб-приложения выделена синей рамкой.

    ![Проверка ценовой категории](media\azure-stack-solution-geo-distributed\image35.png)

Настраиваемые SSL-сертификаты не поддерживаются в ценовых категориях **Бесплатный** или **Общий**. Чтобы повысить уровень, выполните инструкции из следующего раздела или на странице **Выбор ценовой категории** и перейдите к разделу [отправки и привязки SSL-сертификата](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Изменение уровня плана службы приложений

1.  Выберите одну из категорий: **Базовый**, **Стандартный** или **Премиум**.

2.  Щелкните **Выбрать**.

![Выбор ценовой категории](media\azure-stack-solution-geo-distributed\image36.png)

Операция будет считаться завершенной, когда появится уведомление.

![Уведомление об увеличении масштаба](media\azure-stack-solution-geo-distributed\image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Привязка сертификата SSL и объединение промежуточных сертификатов

Объедините нескольких сертификатов в цепочку.

1. **Откройте каждый полученный сертификат** в текстовом редакторе.

2. Создайте файл для объединенных сертификатов и присвойте ему имя *mergedcertificate.crt*. В текстовом редакторе скопируйте содержимое каждого сертификата в этот файл. Порядок сертификатов должен соответствовать порядку в цепочке сертификатов, начиная вашим сертификатом и заканчивая корневым сертификатом. Это должно выглядеть следующим образом:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Экспорт сертификата в PFX-файл

Экспортируйте объединенный SSL-сертификат с закрытым ключом, созданным с помощью сертификата.

Файл закрытого ключа создается с помощью OpenSSL. Чтобы экспортировать сертификат в PFX-файл, выполните приведенную ниже команду, заменив заполнители *<private-key-file>* и *<merged-certificate-file>* путями закрытого ключа и файлом объединения сертификатов соответственно.

```PowerShell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Когда появится приглашение, определите пароль для последующей передачи SSL-сертификата в Службу приложений.

Если вы создали запрос на сертификат с помощью IIS или **Certreq.exe**, установите сертификат на локальный компьютер и [экспортируйте его в PFX-файл](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>Отправка SSL-сертификата

1.  На панели навигации слева выберите **Параметры SSL**.

2.  Выберите **Отправить сертификат**.

3.  В разделе **PFX-файл сертификата** выберите PFX-файл.

4.  4. В поле **Пароль сертификата** введите пароль, созданный при экспорте PFX-файла.

5.  Щелкните **Отправить**.

![Передача сертификата](media\azure-stack-solution-geo-distributed\image38.png)

Когда Служба приложений завершит передачу сертификата, он появится на странице **Параметры SSL**.

![Alt text](media\azure-stack-solution-geo-distributed\image39.png)

#### <a name="bind-your-ssl-certificate"></a>Привязка SSL-сертификата

1.  В разделе **SSL-привязки** выберите **Добавить привязку**.

    > [!Note]  
    >  Если сертификат уже передан, но не появляется в списке доменных имен в раскрывающемся списке **Имя узла**, попробуйте обновить страницу браузера.

1.  На странице **добавления привязки SSL** в раскрывающихся списках выберите доменное имя для защиты и используемый сертификат.

2.  В разделе **Тип SSL** можно выбрать SSL на основе [**указания имени сервера (SNI)**](http://en.wikipedia.org/wiki/Server_Name_Indication) или IP-адреса.

-   **SSL на основе SNI** позволяет добавить несколько привязок SSL на основе SNI. Этот параметр позволяет использовать несколько SSL-сертификатов для защиты нескольких доменов с одним IP-адресом. Большинство современных браузеров (включая Internet Explorer, Chrome, Firefox и Opera) поддерживает SNI (более подробную информацию о поддержки браузеров можно найти в статье [Server Name Indication](http://wikipedia.org/wiki/Server_Name_Indication) (Указание имени сервера)).

-   **SSL на основе IP-адреса** позволяет добавить только одну привязку SSL на основе IP-адреса. Этот параметр позволяет использовать только один SSL-сертификат для защиты выделенного общедоступного IP-адреса. Чтобы защитить несколько доменов, используйте один и тот же SSL-сертификат. Это традиционный вариант привязки SSL.

    1.  Щелкните **Добавить привязку**.

    ![Alt text](media\azure-stack-solution-geo-distributed\image40.png)

Когда служба приложений завершит передачу сертификата, он появится в разделах **SSL-привязки**.

![Alt text](media\azure-stack-solution-geo-distributed\image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Переназначение записи A для SSL на основе IP-адреса

Если вы не используете в веб-приложении SSL на основе IP-адреса, перейдите к разделу о [тестировании протокола HTTPS для личного домена](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

По умолчанию веб-приложение использует общий общедоступный IP-адрес. После привязки сертификата с помощью SSL на основе IP-адреса Служба приложений создает выделенный IP-адрес для вашего веб-приложения.

Когда запись A будет сопоставлена с веб-приложением, следует внести в реестр домена новый выделенный IP-адрес.

Он появится на странице **Личный домен**. Скопируйте этот [IP-адрес](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), а затем измените его в [записи A](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain).

#### <a name="test-https"></a>Тестирование HTTPS

В любом браузере перейдите по адресу https://<ваш.личный.домен> и убедитесь, что открывается нужное веб-приложение.

![Alt text](media\azure-stack-solution-geo-distributed\image42.png)

> [!Note]  
> Если возникнет ошибка проверки сертификата, это может быть связано с использованием самозаверяющего сертификата или с пропуском промежуточного сертификата при экспорте в PFX-файл.

#### <a name="enforce-https"></a>Принудительное использование HTTPS

По умолчанию любой пользователь может получить доступ к вашему веб-приложению по протоколу HTTP. Вы можете перенаправить все HTTP-запросы на HTTPS-порт.

На странице веб-приложения выберите **Параметры SL**. Затем в окне **Только HTTPS**выберите **ВКЛ**.

![Принудительное использование HTTPS](media\azure-stack-solution-geo-distributed\image43.png)

По завершении операции перейдите по любому из URL-адресов HTTP, которые указывают на ваше приложение. Например: 

-   http://<имя_приложения>.azurewebsites.net.
-   http://northwindcloud.com
-   <http://www.northwindcloud.com>

#### <a name="enforce-tls-1112"></a>Принудительное применение TLS 1.1/1.2

Приложение по умолчанию разрешает применение [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) версии 1.0, которая больше не считается безопасной в соответствии с отраслевыми стандартами, такими как [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). Чтобы принудительно применить TLS более поздней версии, выполните следующие инструкции:

1.  На странице веб-приложения в области слева выберите **Параметры SSL**.

2.  Затем в разделе **Версия TLS** выберите минимальную требуемую версию TLS.

![Принудительное использование TLS 1.1 или 1.2](media\azure-stack-solution-geo-distributed\image44.png)

### <a name="create-a-traffic-manager-profile"></a>Создание профиля диспетчера трафика

1.  Последовательно выберите **Создать ресурс** > **Сети** > **Профиль диспетчера трафика** > **Создать**.

2.  В области **Создание профиля диспетчера трафика** сделайте следующее:

    1.  В поле **Имя** укажите имя профиля. Оно должно быть уникальным в пределах зоны trafficmanager.net. В результате будет создано DNS-имя trafficmanager.net для доступа к профилю диспетчера трафика.

    2.  В списке **Метод маршрутизации** выберите **метод географической маршрутизации**.

    3.  В списке **Подписка** выберите подписку, в которой нужно создать профиль.

    4.  В разделе **Группа ресурсов** создайте группу ресурсов, в которую следует поместить этот профиль.

    5.  Из списка **Расположение группы ресурсов** выберите расположение группы ресурсов. Этот параметр задает расположение группы ресурсов и не влияет на профиль диспетчера трафика, который будет развернут глобально.

    6.  Нажмите кнопку **Создать**.

    7.  Когда завершится глобальное развертывание профиля для диспетчера трафика, он появится в соответствующей группе ресурсов как один из ресурсов.

    ![Alt text](media\azure-stack-solution-geo-distributed\image45.png)

### <a name="add-traffic-manager-endpoints"></a>Добавление конечных точек диспетчера трафика

1.  На панели поиска портала выполните поиск по имени **профиля диспетчера трафика**, созданного в предыдущем разделе, и выберите профиль диспетчера трафика в отображаемых результатах.

2.  В колонке **Профиль диспетчера трафика** в разделе **Параметры** щелкните **Конечные точки**.

3.  Выберите **Добавить**.

4.  Добавление конечной точки Azure Stack.

5.  В поле **Тип** выберите **Внешняя конечная точка**.

6.  Укажите **имя** для этой конечной точки, желательно имя Azure Stack.

7.  В качестве полного доменного имени (**FQDN**) укажите внешний URL-адрес веб-приложения Azure Stack.

8.  В разделе географической привязки выберите регион или континент, где находится ресурс, например **Europe** (Европа).

9.  В раскрывающемся списке "Страна/Регион" выберите страну, для которой будет применяться эта конечная точка, например **Germany** (Германия).

10. Оставьте флажок **Добавить как отключенный** снятым.

11. Нажмите кнопку **ОК**.

12. Добавление конечной точки Azure.

    1.  В раскрывающемся списке **Тип** выберите **Конечная точка Azure**.

    2.  Укажите **Имя** для этой конечной точки.

    3.  В раскрывающемся списке **Тип целевого ресурса** выберите **Служба приложений**.

    4.  В разделе **Целевой ресурс** щелкните **Выберите службу приложений**, чтобы отобразился список веб-приложений, размещенных в этой подписке. В колонке **Ресурс** выберите службу приложений, которую вы намерены добавить в качестве первой конечной точки.

13. В разделе географической привязки выберите регион или континент, где находится ресурс, например **North America/Central America/Caribbean** (Северная Америка, Центральная Америка, Карибские острова).

14. В раскрывающемся списке "Страна/Регион" оставьте пустое значение, чтобы выбрать полностью весь регион, указанный выше.

15. Оставьте флажок **Добавить как отключенный** снятым.

16. Нажмите кнопку **ОК**.

  > [!Note]  
  >  Создайте по меньшей мере одну конечную точку с географической областью All (World) (Все (мир)), которая будет конечной точкой по умолчанию для этого ресурса.

1.  После добавления обе конечные точки отобразятся в колонке **Профиль диспетчера трафика** с состоянием **В сети**.

  ![Alt text](media\azure-stack-solution-geo-distributed\image46.png)

**Предприятия мирового уровня используют возможности географического распределения Azure**

Направляя трафик в диспетчер трафика Azure и конечные точки с привязкой к географическим регионам, предприятия мирового уровня обеспечивают соблюдение региональных регламентов, соответствие политикам и безопасность данных, которые так важны для успеха локальных и зарубежных предприятий и филиалов.

## <a name="next-steps"></a>Дополнительная информация

- Дополнительные сведения о шаблонах для облака Azure см. в статье [Конструктивные шаблоны облачных решений](https://docs.microsoft.com/azure/architecture/patterns).