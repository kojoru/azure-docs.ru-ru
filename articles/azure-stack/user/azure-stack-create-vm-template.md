---
title: Руководство по созданию виртуальной машины Azure Stack с помощью шаблона | Документация Майкрософт
description: В статье описано, как с помощью пакета SDK создать виртуальную машину с использованием предопределенного шаблона и настраиваемого шаблона GitHub.
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/12/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 0b3ba5c3a091cf673d8b3dbc413d36cb5fb75de5
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/12/2018
ms.locfileid: "44713414"
---
# <a name="tutorial-create-a-vm-using-a-community-template"></a>Руководство по созданию виртуальной машины с использованием шаблона сообщества
Оператор или пользователь Azure Stack может создать виртуальную машину с помощью [настраиваемых шаблонов GitHub для быстрого запуска](https://github.com/Azure/AzureStack-QuickStart-Templates), а не развертывать их вручную из Azure Stack Marketplace.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Сведения о шаблонах быстрого запуска Azure Stack 
> * Создать виртуальную машину с помощью настраиваемого шаблона GitHub.
> * Запустить minikube и установить приложение.

## <a name="learn-about-azure-stack-quickstart-templates"></a>Сведения о шаблонах быстрого запуска Azure Stack
Шаблоны быстрого запуска Azure Stack хранятся в [общедоступном хранилище шаблонов быстрого запуска Azure Stack](https://github.com/Azure/AzureStack-QuickStart-Templates) на GitHub. Этот репозиторий содержит шаблоны развертывания Azure Resource Manager, которые были протестированы с помощью Пакета средств разработки Microsoft Azure Stack (ASDK). Вы можете использовать их для упрощения оценки Azure Stack и использования среды ASDK. 

С течением времени многие пользователи GitHub внесли свой вклад в репозиторий, в результате чего появилась огромная коллекция из более чем 400 шаблонов развертывания. Этот репозиторий — отличная отправная точка, позволяющая лучше понять, как развернуть различные среды в Azure Stack. 

>[!IMPORTANT]
> Некоторые из этих шаблонов создаются членами сообщества, а не корпорацией Майкрософт. Каждый шаблон лицензирован для вас в соответствии с условиями лицензионного соглашения с владельцем, а не с корпорацией Майкрософт. Корпорация Майкрософт не несет ответственности за шаблоны и не отслеживает их безопасность, совместимость или производительность. Шаблоны, созданные сообществом, не поддерживаются ни в соответствии с какой-либо программой поддержки Майкрософт, ни какими-либо службами поддержки Майкрософт и предоставляются КАК ЕСТЬ без каких-либо гарантий.

Если вы хотите внести свои шаблоны Azure Resource Manager на GitHub, воспользуйтесь [репозиторием azure-quickstart-templates](https://github.com/Azure/AzureStack-QuickStart-Templates).

Чтобы узнать больше о репозитории GitHub и о том, как внести в него вклад, ознакомьтесь с [файлом сведений](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/README.md). 


## <a name="create-a-vm-using-a-custom-github-template"></a>Создание виртуальной машины с помощью настраиваемого шаблона GitHub
В этом примере руководства шаблон быстрого запуска [101-vm-linux-minikube](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/101-vm-linux-minikube) Azure Stack используется для развертывания виртуальной машины Ubuntu 16.04 в AzureStack, работающей с Minikube, для управления кластером Kubernetes.

Minikube — это средство, которое позволяет легко запускать Kubernetes локально. Minikube запускает одноузловой кластер Kubernetes внутри виртуальной машины, это позволяет опробовать Kubernetes или использовать его для ежедневной разработки. Он поддерживает простой кластер Kubernetes с одним узлом, работающий на виртуальной машине Linux. Minikube — это самый быстрый и прямой способ получить полностью функциональный кластер Kubernetes. Он позволяет разрабатывать и тестировать развертывания приложений на основе Kubernetes на своих локальных машинах. С точки зрения архитектуры на виртуальной машине Minikube локально выполняются компоненты главного узла и узла агента:

- Компоненты главного узла, такие как сервер API, планировщик и [сервер etcd](https://coreos.com/etcd/), запускаются в одном Linux-процессе под названием LocalKube.
- Компоненты агента узла запускаются внутри контейнеров Docker так же, как и на обычном узле агента. С точки зрения развертывания приложения нет разницы, когда приложение развертывается в Minikube или в обычном кластере Kubernetes.

Этот шаблон устанавливает следующие компоненты:

- виртуальная машина Ubuntu 16.04 LTS;
- [Docker-CE](https://download.docker.com/linux/ubuntu); 
- [Kubectl](https://storage.googleapis.com/kubernetes-release/release/v1.8.0/bin/linux/amd64/kubectl)
- [Minikube](https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64);
- xFCE4;
- xRDP.

> [!IMPORTANT]
> Прежде чем приступить к выполнению этих шагов, убедитесь, что образ виртуальной машины Ubuntu (Ubuntu Server 16.04 LTS в этом примере) уже добавлен в Azure Stack Marketplace.

1.  Нажмите кнопку **+ Create a resource** (+ Cоздать ресурс)  > **Custom** (Пользовательский)  > **Развертывание шаблона**.

    ![](media/azure-stack-create-vm-template/1.PNG) 

2. Щелкните **Изменить шаблон**.

    ![](media/azure-stack-create-vm-template/2.PNG) 

3.  Щелкните **Шаблон быстрого запуска**.

    ![](media/azure-stack-create-vm-template/3.PNG)

4. Выберите **101-vm-linux minikube** из доступных шаблонов, используя раскрывающийся список **Select a template** (Выберите шаблон), а затем нажмите кнопку **ОК**.  

    ![](media/azure-stack-create-vm-template/4.PNG)

5. Если требуется, внесите изменения в шаблон JSON, если нет, нажмите кнопку **Сохранить**, чтобы закрыть диалоговое окно редактирования.

    ![](media/azure-stack-create-vm-template/5.PNG) 

6.  Щелкните **Параметры**, заполните или измените доступные поля и нажмите кнопку **ОК**. Выберите подписку для использования, создания или выбора имеющегося имени группы ресурсов, а затем нажмите кнопку **Создать**, чтобы начать развертывание шаблона.

    ![](media/azure-stack-create-vm-template/6.PNG)

7. Выберите подписку для использования, создания или выбора имеющегося имени группы ресурсов, а затем нажмите кнопку **Создать**, чтобы начать развертывание шаблона.

    ![](media/azure-stack-create-vm-template/7.PNG)

8. Развертыванию группы ресурсов требуется несколько минут для создания настраиваемой виртуальной машины на основе шаблона. Вы можете контролировать состояние установки с помощью уведомлений и в свойствах группы ресурсов. 

    ![](media/azure-stack-create-vm-template/8.PNG)

    >[!NOTE]
    > После развертывания будет запущена виртуальная машина. 

## <a name="start-minikube-and-install-an-application"></a>Запуск minikube и установка приложения
Теперь, когда виртуальная машина Linux была успешно создана, вы можете войти в систему, чтобы запустить minikube и установить приложение. 

1. После развертывания нажмите **Подключить**, чтобы просмотреть общедоступный IP-адрес, который будет использоваться для подключения к виртуальной машине Linux. 

    ![](media/azure-stack-create-vm-template/9.PNG)

2. В командной строке с повышенными привилегиями запустите файл **mstsc.exe**, чтобы открыть подключение к удаленному рабочему столу и подключиться к общедоступному IP-адресу виртуальной машины Linux, обнаруженному на предыдущем шаге. При появлении запроса на вход в xRDP используйте учетные данные, указанные при создании виртуальной машины.

    ![](media/azure-stack-create-vm-template/10.PNG)

3. Откройте эмулятор терминала и введите следующие команды для запуска minikube:

    ```shell
    sudo minikube start --vm-driver=none
    sudo minikube addons enable dashboard
    sudo minikube dashboard --url
    ```

    ![](media/azure-stack-create-vm-template/11.PNG)

4. Откройте веб-браузер и перейдите на страницу панели мониторинга Kubernetes. Поздравляем! Теперь у вас есть полностью работающая установка Kubernetes с использованием minikube.

    ![](media/azure-stack-create-vm-template/12.PNG)

5. Если вы хотите развернуть пример приложения, зайдите на официальную страницу документации по Kubernetes, пропустите раздел "Create Minikube Cluster" (Создание кластера Minikube), так как вы уже создали один из них. Сразу перейдите в раздел "Create your Node.js application" (Создание приложения Node.js) тут: https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/.

## <a name="next-steps"></a>Дополнительная информация

Из этого руководства вы узнали, как выполнить следующие задачи:

> [!div class="checklist"]
> * Сведения о шаблонах быстрого запуска Azure Stack 
> * Создать виртуальную машину с помощью настраиваемого шаблона GitHub.
> * Запуск minikube и установка приложения

