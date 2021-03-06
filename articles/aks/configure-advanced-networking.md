---
title: Настройка расширенных параметров сети в Службе Azure Kubernetes (AKS)
description: Узнайте, как настроить расширенные параметры сети в Службе Azure Kubernetes (AKS), включая развертывание кластера AKS в имеющейся виртуальной сети и подсети.
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 10/11/2018
ms.author: iainfou
ms.openlocfilehash: 87c3ab9624116e9c1c61041531fdf5d3b26117e1
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/17/2018
ms.locfileid: "49380762"
---
# <a name="configure-advanced-networking-in-azure-kubernetes-service-aks"></a>Настройка расширенных параметров сети в Службе Azure Kubernetes (AKS)

По умолчанию кластеры AKS используют *базовые параметры* сети, что включает в себя создание и настройку виртуальной сети и подсети для кластера. Чтобы обеспечить дополнительный контроль над такими параметрами сети, как диапазоны IP-адресов, можно вместо этого использовать *расширенные параметры* сети. С помощью расширенных параметров сети можно также создать кластер AKS в существующей виртуальной сети и подсети. Часто существующая виртуальная сеть обеспечивает подключение к локальной сети с помощью Azure ExpressRoute или VPN-подключения типа "сеть – сеть".

В этой статье показано, как использовать расширенные параметры сети для создания и использования виртуальной сети для кластера AKS. Дополнительные сведения о сетях см. в разделе [Основные понятия сети для Kubernetes и AKS][aks-network-concepts].

## <a name="prerequisites"></a>Предварительные требования

* Виртуальная сеть для кластера AKS должна разрешать исходящее подключение к Интернету.
* Не создавайте больше одного кластера AKS в одной подсети.
* Кластер AKS не может использовать `169.254.0.0/16`, `172.30.0.0/16` или `172.31.0.0/16` для диапазона адресов службы Kubernetes.
* Субъект-служба, используемый кластером AKS, должен иметь по крайней мере разрешения [Участник сетей](../role-based-access-control/built-in-roles.md#network-contributor) в подсети в виртуальной сети. Если вы хотите определить [пользовательскую роль](../role-based-access-control/custom-roles.md) вместо того, чтобы использовать встроенную роль участника сети, требуются следующие разрешения:
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

## <a name="plan-ip-addressing-for-your-cluster"></a>Планирование назначения IP-адресов для кластера

Для кластеров с настроенным расширенным сетевым взаимодействием требуется дополнительное планирование. Размер виртуальной сети и подсети должен быть достаточным для количества контейнеров pod, которые будут одновременно запускаться, а также соответствовать количеству узлов в кластере.

IP-адреса для контейнеров pod и узлов кластера назначаются из определенной подсети в виртуальной сети. Для каждого узла настраивается основной IP-адрес, который является IP-адресом узла, и 30 дополнительных IP-адресов, предустановленных Azure CNI и назначенных контейнерам pod, запланированным на узле. При масштабировании кластера каждый узел настраивается аналогичным образом с использованием IP-адресов из подсети.

План IP-адреса для кластера AKS содержит виртуальную сеть, по крайней мере одну подсеть для узлов и контейнеров pod, а также диапазон адресов службы Kubernetes.

| Диапазон адресов / ресурс Azure | Установления размера и ограничения |
| --------- | ------------- |
| Виртуальная сеть | Виртуальная сеть Azure может достигать размера /8, но ограничиваться 65 536 настроенными IP-адресами. |
| Подсеть | Подсеть должна быть достаточно большой, чтобы разместить узлы, контейнеры pod и все ресурсы Kubernetes и Azure, которые могут быть выделены в кластере. Например, если развертывается внутренний Azure Load Balancer, его внешние IP-адреса выделяются из подсети кластера, а не из публичных IP-адресов. <p/>Вычисление *минимального* размера подсети: `(number of nodes) + (number of nodes * pods per node)` <p/>Пример для кластера из 50 узлов: `(50) + (50 * 30) = 1,550` (/21 или больше) |
| Диапазон адресов службы Kubernetes | Этот диапазон не должен использоваться элементом виртуальной сети или элементом, подключенным к ней. Адрес службы CIDR должен иметь размер меньше /12. |
| IP-адрес службы доменных имен (DNS) Kubernetes | IP-адрес в пределах диапазона адресов службы Kubernetes, который будет использоваться службой обнаружения кластеров (kube-dns). |
| Адрес моста Docker | IP-адрес (в нотации CIDR), используемый в качестве IP-адреса моста Docker на узлах. По умолчанию — 172.17.0.1/16. |

## <a name="maximum-pods-per-node"></a>Максимальное число контейнеров pod на узле

Максимальное количество контейнеров pod по умолчанию для каждого узла в кластере AKS зависит от базовой и расширенной сети и метода развертывания кластера.

| Метод развертывания | базовая; | Расширенная | Настройка при развертывании |
| -- | :--: | :--: | -- |
| Инфраструктура CLI Azure | 110 | 30 | Да (до 110) |
| Шаблон Resource Manager | 110 | 30 | Да (до 110) |
| Microsoft Azure | 110 | 30 | Нет  |

### <a name="configure-maximum---new-clusters"></a>Настройка максимального числа. Новые кластеры

Вы можете настроить максимальное число элементов pod на узел *только во время развертывания кластера*. При развертывании с помощью Azure CLI или шаблона Resource Manager можно задать максимальное число pod на узел (но не больше 110).

* **Azure CLI**. Укажите аргумент `--max-pods` при развертывании кластера с помощью команды [az aks create][az-aks-create]. Максимальное значение — 110.
* **Шаблон Resource Manager**. Укажите свойство `maxPods` в объекте [ManagedClusterAgentPoolProfile] при развертывании кластера с шаблоном Resource Manager. Максимальное значение — 110.
* **Портал Azure**. Невозможно изменить максимальное количество элементов pod на узел при развертывании кластера с помощью портала Azure. Кластеры с расширенными сетевыми возможностями ограничены 30 элементами pod на узел при развертывании с помощью портала Azure.

### <a name="configure-maximum---existing-clusters"></a>Настройка максимального числа. Имеющиеся кластеры

Невозможно изменить максимальное количество контейнеров pod на узле в имеющемся кластере AKS. Вы можете изменить количество только во время изначального развертывания кластера.

## <a name="deployment-parameters"></a>Параметры развертывания

При создании кластера AKS для сетевого взаимодействия уровня "Расширенный" можно настроить следующие параметры.

**Виртуальная сеть**. Виртуальная сеть, в которую необходимо развернуть кластер Kubernetes. Если для кластера необходимо создать новую виртуальную сеть, выберите *Создать новый* и следуйте инструкциям раздела *Создание виртуальной сети*. Дополнительные сведения об ограничениях и квотах для виртуальной сети Azure см. в статье [Подписка Azure, границы, квоты и ограничения службы](../azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits).

**Подсеть**. Подсеть в виртуальной сети, в которую требуется развернуть кластер. Если для кластера необходимо создать подсеть в виртуальной сети, выберите *Создать новый* и следуйте инструкциям раздела *Создание подсети*.

**Диапазон адресов службы Kubernetes** — набор виртуальных IP-адресов, которые Kubernetes присваивает [службам][services] в кластере. Можно использовать любой диапазон частных адресов, который отвечает следующим требованиям:

* Должен быть за пределами диапазона IP-адресов виртуальной сети вашего кластера.
* Не должен пересекаться с диапазоном других виртуальных сетей, с которыми связан кластер виртуальной сети.
* не должен перекрываться с какими-либо локальными IP-адресами.
* Адрес не должен быть в диапазонах `169.254.0.0/16`, `172.30.0.0/16` или `172.31.0.0/16`.

Не рекомендуется указывать диапазон адресов службы в той же виртуальной сети, что и кластер, хотя технически это возможно. Перекрывающиеся диапазоны IP-адресов могут привести к непредсказуемому поведению. Дополнительные сведения см. в разделе этой статьи [с вопросами и ответами](#frequently-asked-questions). Дополнительные сведения о службах Kubernetes см. в [этой документации][services].

**IP-адрес службы DNS Kubernetes** — IP-адрес службы DNS кластера. Этот адрес должен быть в пределах *диапазона адресов службы Kubernetes*.

**Адрес моста Docker** — IP-адрес и маска сети, которую необходимо назначить мосту Docker. Этот IP-адрес должен быть за пределами диапазона IP-адресов виртуальной сети вашего кластера.

## <a name="configure-networking---cli"></a>Настройка сети с помощью интерфейса командной строки

При создании кластера AKS с помощью Azure CLI можно также настроить сетевое взаимодействие уровня "Расширенный". Используйте приведенные ниже команды для создания кластера AKS с включенными функциями сетевого взаимодействия уровня "Расширенный".

Сначала получите идентификатор ресурса существующей подсети, к которой будет присоединен кластер AKS.

```console
$ az network vnet subnet list --resource-group myVnet --vnet-name myVnet --query [].id --output tsv

/subscriptions/d5b9d4b7-6fc1-46c5-bafe-38effaed19b2/resourceGroups/myVnet/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/default
```

Используйте команду [az aks create][az-aks-create] с аргументом `--network-plugin azure` для создания кластера с функциями сетевого взаимодействия уровня "Расширенный". Измените значение `--vnet-subnet-id`, указав идентификатор подсети, полученный на предыдущем шаге.

```azurecli
az aks create --resource-group myAKSCluster --name myAKSCluster --network-plugin azure --vnet-subnet-id <subnet-id> --docker-bridge-address 172.17.0.1/16 --dns-service-ip 10.2.0.10 --service-cidr 10.2.0.0/24
```

## <a name="configure-networking---portal"></a>Настройка сети с помощью портала

На следующем снимке экрана на портале Azure показан пример настройки этих параметров во время создания кластера AKS.

![Конфигурация расширенного сетевого взаимодействия на портале Azure][portal-01-networking-advanced]

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

Следующие вопросы и ответы относятся к конфигурации сети **Расширенный**.

* *Можно ли развернуть виртуальные машины в подсети моего кластера?*

  Нет. Развертывание виртуальных машин в подсети, используемой кластером Kubernetes, не поддерживается. Виртуальные машины можно развернуть в одной и той же виртуальной сети, но в разных подсетях.

* *Можно ли настроить политики сети для каждого контейнера pod?*

  Нет. Политики сети для каждого контейнера pod сейчас не поддерживаются.

* *Можно ли настроить максимальное число контейнеров pod, развертываемых на узел?*

  Да, при развертывании кластера с помощью Azure CLI или шаблона Resource Manager. См. статью [Конфигурация сети в службе Azure Kubernetes (AKS)](#maximum-pods-per-node).

  Невозможно изменить максимальное количество контейнеров pod в узле в имеющемся кластере.

* *Как настроить дополнительные свойства для подсети, созданной во время создания кластера AKS? Например, конечные точки службы.*

  Полный список свойств для виртуальной сети и подсетей, создаваемых во время создания кластера AKS, можно настроить на странице стандартной конфигурации виртуальной сети на портале Azure.

* *Можно ли использовать другую подсеть в виртуальной сети кластера для* **диапазона адресов службы Kubernetes**?

  Не рекомендуется, но эта конфигурация возможна. Диапазон адресов службы Kubernetes — это набор виртуальных IP-адресов, которые Kubernetes присваивает службам в кластере. Сеть Azure не видит адреса в диапазоне IP-адресов службы кластера Kubernetes. Из-за этого позже возможно создать подсеть в виртуальной сети кластера, которая пересекается с диапазоном адресов службы. При возникновении такого пересекания Kubernetes может присвоить службе IP-адрес, который уже используется другим ресурсом в подсети, что приводит к непредсказуемому поведению или сбоям. Убедившись, что диапазон адресов используется за пределами виртуальной сети кластера, можно избежать такого риска пересечений.

## <a name="next-steps"></a>Дополнительная информация

### <a name="networking-in-aks"></a>Сетевое взаимодействие в AKS

Узнайте больше о сетевом взаимодействии в AKS из следующих статей:

- [Использование статического IP-адреса с подсистемой балансировки нагрузки Службы контейнеров Azure (AKS)](static-ip.md)
- [Использование внутренней подсистемы балансировки нагрузки со Службой контейнеров Azure (AKS)](internal-lb.md)

- [Создать базовый контроллер входящего трафика с внешним сетевым подключением.][aks-ingress-basic]
- [Включить надстройку маршрутизации приложений HTTP.][aks-http-app-routing]
- [Создать контроллер входящего трафика, который использует внутреннюю, частную сети и IP-адрес.][aks-ingress-internal]
- [Создать контроллер входящего трафика с динамическим общедоступным IP-адресом и настроить шифрование для автоматического создания TLS-сертификатов.][aks-ingress-tls]
- [Создание контроллера входящего трафика со статическим общедоступным IP-адресом и настройка шифрования для автоматического создания TLS-сертификатов][aks-ingress-static-tls]

### <a name="acs-engine"></a>Обработчик ACS

[Обработчик Службы контейнера Azure (обработчик ACS)][acs-engine] — это проект с открытым исходным кодом, создающий шаблоны Azure Resource Manager, которые можно использовать для развертывания кластеров с поддержкой Docker в Azure. С помощью обработчика ACS можно развернуть Kubernetes, DC/OS, режим Swarm и оркестраторы Swarm.

Кластеры Kubernetes, созданные с помощью обработчика ACS, поддерживают подключаемые модули [kubenet][kubenet] и [Azure CNI][cni-networking]. Таким образом, обработчик ACS поддерживает сценарии как базового, так и расширенного сетевого взаимодействия.

<!-- IMAGES -->
[advanced-networking-diagram-01]: ./media/networking-overview/advanced-networking-diagram-01.png
[portal-01-networking-advanced]: ./media/networking-overview/portal-01-networking-advanced.png

<!-- LINKS - External -->
[acs-engine]: https://github.com/Azure/acs-engine
[services]: https://kubernetes.io/docs/concepts/services-networking/service/
[portal]: https://portal.azure.com
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet

<!-- LINKS - Internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[aks-ssh]: ssh.md
[ManagedClusterAgentPoolProfile]: /azure/templates/microsoft.containerservice/managedclusters#managedclusteragentpoolprofile-object
[aks-network-concepts]: concepts-network.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-internal]: ingress-internal-ip.md
