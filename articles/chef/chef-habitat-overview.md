---
title: Использование Habitat для развертывания приложения в Azure
description: Узнайте, как последовательно развернуть приложение в виртуальные машины и контейнеры Azure
keywords: azure, chef, devops, virtual machines, overview, automate, habitat
ms.service: virtual-machines-linux
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 05/15/2018
ms.topic: article
ms.openlocfilehash: 73c6834eb53c0496fa673dd25ab90abc18a6a139
ms.sourcegitcommit: 96089449d17548263691d40e4f1e8f9557561197
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/17/2018
ms.locfileid: "34267205"
---
# <a name="use-habitat-to-deploy-your-application-to-azure"></a>Использование Habitat для развертывания приложения в Azure
[Habitat](https://www.habitat.sh/) является первым в своем роде проектом с открытым исходным кодом, который предлагает полностью новый подход к управлению приложениями. Habitat делает приложение и его службу автоматизации единицей развертывания. Когда приложения упакованы в легковесное расширение, среда выполнения, будь то контейнер, исходное состояние системы или PaaS, больше не находятся в фокусе и не ограничивают приложения. 

В этой статье описываются преимущества использования Habitat.

## <a name="support-for-the-modern-application"></a>Поддержка для современных приложений
Пакеты Habitat включают все, что необходимо приложению на протяжении его жизненного цикла. Основными компонентами Habitat являются:
- Формат упаковки. Приложения в пакете Habitat являются атомарными, неизменяемыми и проверяемыми.
- Супервизор Habitat запускает пакеты приложений с учетом взаимных связей пакетов, стратегии обновления и политик безопасности. Супервизор Habitat настраивает приложение и управляет им для любой имеющейся среды.
- Кроме того, экосистема Habitat предоставляет службу сборки, которая принимает план сборки Habitat, создает пакет Habitat и публикует его в хранилище.

## <a name="run-any-application-anywhere"></a>Выполнение любого приложения в любой среде
С помощью Habitat приложения могут работать без изменений в любой среде выполнения. Это включает в себя все: от исходного состояния систем и виртуальных машин до контейнеров (например, Docker), систем управления кластерами (таких как Mesosphere или Kubernetes) и систем PaaS (таких как Pivotal Cloud Foundry).

## <a name="easily-port-legacy-applications"></a>Легкий перенос приложений прежних версий
Когда приложения прежних версий упакованы в пакет Habitat, они не зависят от среды, для которой были изначально разработаны. Пакеты могут быть быстро перенесены в более современные среды, такие как облако или контейнеры. Кроме того, так как пакеты Habitat имеют стандартный внешний интерфейс, управлять приложениями прежних версий становится намного проще.

## <a name="improve-the-container-experience"></a>Улучшения интерфейса контейнера
Habitat упрощает управление контейнерами в рабочих средах. Автоматизируя конфигурацию приложения в контейнере, Habitat устраняет сложности, с которыми сталкиваются разработчики при перемещении приложений на основе контейнеров из среды разработки в рабочую среду.

## <a name="integrate-into-the-chef-devops-workflow"></a>Интеграция в рабочий процесс DevOps Chef
Проект Habitat спонсируется Chef. Habitat использует обширный опыт Chef в области автоматизации инфраструктуры для обеспечения беспрецедентных возможностей автоматизации приложений. Chef предложит коммерческую поддержку Habitat и обеспечит простую интеграцию между Habitat и Chef Delivery, чтобы полностью автоматизировать цикл выпуска приложений — от разработки до развертывания.

## <a name="next-steps"></a>Дополнительная информация
* [Автоматизация развертывания виртуальной машины Azure с помощью Chef](/azure/virtual-machines/windows/chef-automation)