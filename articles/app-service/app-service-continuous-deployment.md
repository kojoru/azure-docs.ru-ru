---
title: Непрерывное развертывание в службе приложений Azure | Документация Майкрософт
description: Узнайте, как включить непрерывное развертывание в службе приложений Azure.
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
ms.assetid: 6adb5c84-6cf3-424e-a336-c554f23b4000
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/05/2018
ms.author: cephalin;dariagrigoriu
ms.openlocfilehash: bd440e0ef017e2bf116e80ad049883e2338efddb
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298953"
---
# <a name="continuous-deployment-to-azure-app-service"></a>Непрерывное развертывание в службе приложений Azure
В этой статье показано, как настроить непрерывное развертывание для [службы приложений Azure](app-service-web-overview.md). Служба приложений обеспечивает непрерывное развертывание из BitBucket, GitHub и [Azure DevOps Services](https://www.visualstudio.com/team-services/), выбирая самые последние обновления из существующего репозитория в одной из этих служб.

Чтобы узнать, как вручную настроить непрерывное развертывание из облачного репозитория, которого нет списке на портале Azure (например, [GitLab](https://gitlab.com/)), см. раздел [Setting up continuous deployment using manual steps](https://github.com/projectkudu/kudu/wiki/Continuous-deployment#setting-up-continuous-deployment-using-manual-steps) (Настройка непрерывного развертывания вручную).

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

Опубликуйте готовый репозиторий для одной из поддерживаемых служб. Дополнительные сведения о публикации проекта в этих службах см. в статьях, посвященных [Create a repo (GitHub)] и [Create a repo (BitBucket)], а также [DevOps overview for Azure DevOps Services and TFS].

## <a name="deploy-continuously-from-github"></a>Непрерывное развертывание из GitHub

Чтобы включить непрерывное развертывание с помощью GitHub, перейдите на страницу приложения службы приложений на [портале Azure](https://portal.azure.com).

В меню слева щелкните **Центр развертывания** > **GitHub** > **Авторизовать**. Следуйте инструкциям авторизации. 

![](media/app-service-continuous-deployment/github-choose-source.png)

В GitHub необходимо авторизоваться только один раз. Если авторизация уже пройдена, просто щелкните **Продолжить**. Авторизованную учетную запись GitHub можно изменить, щелкнув **Изменить учетную запись**.

![](media/app-service-continuous-deployment/github-continue.png)

На странице **Поставщик сборки** выберите поставщика сборки и нажмите > **Продолжить**.

### <a name="option-1-use-app-service-kudu-build-server"></a>Вариант 1. Использование сервера сборки Kudu службы приложений

На странице **Настроить** выберите организацию, репозиторий и ветку, из которых будет происходить непрерывное развертывание. По завершении нажмите кнопку **Продолжить**.

### <a name="option-2-use-azure-devops-services-continuous-delivery"></a>Вариант 2. Использование непрерывной поставки Azure DevOps Services

> [!NOTE]
> Чтобы Служба приложений создавала необходимые конвейеры Azure Pipelines в организации Azure DevOps Services, ваша учетная запись Azure должна иметь роль **владельца** в подписке Azure.
>

На странице **Настроить** в разделе **Код** выберите организацию, репозиторий и ветку, из которых будет происходить непрерывное развертывание. По завершении нажмите кнопку **Продолжить**.

На странице **Настроить** в разделе **Сборка** настройте новую организацию Azure DevOps Services или укажите имеющуюся. По завершении нажмите кнопку **Продолжить**.

> [!NOTE]
> Чтобы использовать имеющуюся организацию Azure DevOps Services, не указанную в списке, [ее необходимо связать с подпиской Azure](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App).

На странице **Тест** выберите, следует ли включать тесты нагрузки, затем нажмите **Продолжить**.

В зависимости от [ценовой категории](https://azure.microsoft.com/pricing/details/app-service/plans/) вашего плана App Service также можно увидеть страницу **Разворачивание по этапам**. Выберите, нужно ли [включить слоты развертывания](web-sites-staged-publishing.md) или нет, затем нажмите **Продолжить**.

### <a name="finish-configuration"></a>Завершение настройки

На странице **Сводка** проверьте параметры и нажмите **Готово**.

По окончании настройки в выбранном репозитории в приложении службы приложений будут непрерывно развертываться новые фиксации.

![](media/app-service-continuous-deployment/github-finished.png)

## <a name="deploy-continuously-from-bitbucket"></a>Непрерывное развертывание из BitBucket

Чтобы включить непрерывное развертывание с помощью BitBucket, перейдите на страницу приложения службы приложений на [портале Azure](https://portal.azure.com).

В меню слева щелкните **Центр развертывания** > **BitBucket** > **Авторизовать**. Следуйте инструкциям авторизации. 

![](media/app-service-continuous-deployment/bitbucket-choose-source.png)

В BitBucket необходимо авторизоваться только один раз. Если авторизация уже пройдена, просто щелкните **Продолжить**. Авторизованную учетную запись BitBucket можно изменить, щелкнув **Изменить учетную запись**.

![](media/app-service-continuous-deployment/bitbucket-continue.png)

На странице **Настроить** выберите репозиторий и ветку, из которых будет происходить непрерывное развертывание. По завершении нажмите кнопку **Продолжить**.

На странице **Сводка** проверьте параметры и нажмите **Готово**.

По окончании настройки в выбранном репозитории в приложении службы приложений будут непрерывно развертываться новые фиксации.

## <a name="deploy-continuously-from-azure-devops-services"></a>Непрерывное развертывание из Azure DevOps Services

Чтобы включить непрерывное развертывание с помощью Azure DevOps Services, перейдите на страницу приложения службы приложений на [портале Azure](https://portal.azure.com).

В меню слева щелкните **Центр развертывания** > **Azure DevOps Services** > **Продолжить**. 

![](media/app-service-continuous-deployment/vsts-choose-source.png)

На странице **Поставщик сборки** выберите поставщика сборки и нажмите > **Продолжить**.

### <a name="option-1-use-app-service-kudu-build-server"></a>Вариант 1. Использование сервера сборки Kudu службы приложений

На странице **Настроить** выберите организацию Azure DevOps Services, проект, репозиторий и ветку, из которых будет происходить непрерывное развертывание. По завершении нажмите кнопку **Продолжить**.

### <a name="option-2-use-azure-devops-services-continuous-delivery"></a>Вариант 2. Использование непрерывной поставки Azure DevOps Services

> [!NOTE]
> Чтобы служба приложений создавала необходимые конвейеры Azure Pipelines в организации Azure DevOps Services, ваша учетная запись Azure должна иметь роль **владельца** в подписке Azure.
>

На странице **Настроить** в разделе **Код** выберите организацию Azure DevOps Services, проект, репозиторий и ветку, из которых будет происходить непрерывное развертывание. По завершении нажмите кнопку **Продолжить**.

> [!NOTE]
> Чтобы использовать имеющуюся организацию Azure DevOps Services, не указанную в списке, [ее необходимо связать с подпиской Azure](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App).

На странице **Настроить** в разделе **Сборка** укажите структуру языка, которую должен использовать Azure DevOps Services для запуска задач сборки для выбранного репозитория. По завершении нажмите кнопку **Продолжить**.

На странице **Тест** выберите, следует ли включать тесты нагрузки, затем нажмите **Продолжить**.

В зависимости от [ценовой категории](https://azure.microsoft.com/pricing/details/app-service/plans/) вашего плана App Service также можно увидеть страницу **Разворачивание по этапам**. Выберите, нужно ли [включить слоты развертывания](web-sites-staged-publishing.md) или нет, затем нажмите **Продолжить**. 

### <a name="finish-configuration"></a>Завершение настройки

На странице **Сводка** проверьте параметры и нажмите **Готово**.

По окончании настройки в выбранном репозитории в приложении службы приложений будут непрерывно развертываться новые фиксации.

## <a name="disable-continuous-deployment"></a>Отключение непрерывного развертывания

Чтобы выключить непрерывное развертывание, перейдите на страницу приложения службы приложений на [портале Azure](https://portal.azure.com).

В меню слева щелкните **Центр развертывания** > **GitHub** или **Azure DevOps Services**, или **BitBucket** > **Отключить**.

![](media/app-service-continuous-deployment/disable.png)

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Изучение распространенных проблем с непрерывным развертыванием](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)
* [Использование PowerShell для Azure]
* [Документация по Git]
* [Project Kudu](https://github.com/projectkudu/kudu/wiki)
* [Использование Azure для автоматического создания конвейера CI/CD для развертывания приложения ASP.NET 4](https://www.visualstudio.com/docs/build/get-started/aspnet-4-ci-cd-azure-automatic)

[Azure portal]: https://portal.azure.com
[VSTS Portal]: https://www.visualstudio.com/en-us/products/visual-studio-team-services-vs.aspx
[Installing Git]: http://git-scm.com/book/en/Getting-Started-Installing-Git
[Использование PowerShell для Azure]: /powershell/azureps-cmdlets-docs
[Документация по Git]: http://git-scm.com/documentation

[Create a repo (GitHub)]: https://help.github.com/articles/create-a-repo (Создание репозитория GitHub)
[Create a repo (BitBucket)]: https://confluence.atlassian.com/display/BITBUCKET/Create+an+Account+and+a+Git+Repo (Создание репозитория BitBucket)
[DevOps overview for Azure DevOps Services and TFS]: https://www.visualstudio.com/docs/vsts-tfs-overview (Общие сведения о DevOps для Azure DevOps Services и TFS)
