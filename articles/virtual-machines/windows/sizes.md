---
title: Размеры виртуальных машин Windows в Azure | Документация Майкрософт
description: Список различных размеров виртуальных машин Windows в Azure.
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager,azure-service-management
ms.assetid: aabf0d30-04eb-4d34-b44a-69f8bfb84f22
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 10/01/2018
ms.author: jonbeck
ms.openlocfilehash: cefcad9c427770c083e6451c088ce5cfc10a68f2
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394477"
---
# <a name="sizes-for-windows-virtual-machines-in-azure"></a>Размеры виртуальных машин Windows в Azure

В этой статье описаны доступные размеры и разновидности виртуальных машин Azure, которые можно использовать для запуска приложений для Windows и рабочих нагрузок Windows. Здесь также предоставлены рекомендации по развертыванию, которые нужно учитывать при планировании использования этих ресурсов.  Также доступна версия этой статьи для [виртуальных машин Linux](../linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).


| type                     | Размеры           |    ОПИСАНИЕ       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Универсальные](sizes-general.md)          | B, Dsv3, Dv3, DSv2, Dv2, Av2, DC | Сбалансированное соотношение ресурсов ЦП и памяти. Идеальное решение для тестирования и разработки, небольших и средних баз данных, а также веб-серверов с небольшим или средним объемом трафика. |
| [Оптимизированные для вычислений](sizes-compute.md)        | Fsv2 Fs, F             | Высокое соотношение ресурсов ЦП и памяти. Подходят для веб-серверов со средним объемом трафика, сетевых устройств, пакетных процессов и серверов приложений.        |
| [Оптимизированные для памяти](../virtual-machines-windows-sizes-memory.md)         | Esv3, Ev3, M, GS, G, DSv2, Dv2  | Высокое соотношение ресурсов памяти и ядра. Отлично подходят для серверов реляционной базы данных, кэша среднего и большого объема, а также выполняющейся в памяти аналитики.                 |
| [Оптимизированные для хранилища](../virtual-machines-windows-sizes-storage.md)        | Ls                | Высокая пропускная способность дисков и количество операций ввода-вывода. Идеальный вариант для работы с большими данными, а также с базами данных SQL и NoSQL.                                                         |
| [GPU](sizes-gpu.md)            | NV, NVv2, NC, NCv2, NCv3, ND            | Специализированные виртуальные машины, предназначенные для ресурсоемкой отрисовки изображений и редактирования видео, а также для обучения моделей и формирования выводов с помощью глубокого обучения. Доступны виртуальные машины одним или несколькими GPU.       |
| [Для высокопроизводительных вычислений](sizes-hpc.md) | H       | Виртуальные машины с самыми быстрыми и мощными ЦП, для которых можно настроить сетевые интерфейсы с высокой пропускной способностью (RDMA). |


<br> 

- Подробнее о ценах на различные размеры см. в разделе [Цены на виртуальные машины](https://azure.microsoft.com/pricing/details/virtual-machines/#Windows). 
- Сведения об общих ограничениях виртуальных машин Azure см. в статье [Подписка Azure, границы, квоты и ограничения службы](../../azure-subscription-service-limits.md).
- Стоимость хранилища рассчитывается отдельно в зависимости от количества страниц, используемых в учетной записи хранения. Дополнительные сведения см. на странице [Цены на хранилища Azure](https://azure.microsoft.com/pricing/details/storage/).
- Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.


## <a name="rest-api"></a>REST API

Сведения об использовании Rest API, чтобы получить сведения о размерах виртуальных машин, см. в следующих статьях:

- [List available virtual machine sizes for resizing](https://docs.microsoft.com/rest/api/compute/virtualmachines/virtualmachines_listavailablesizes) (Вывод доступных размеров виртуальных машин для изменения размеров)
- [List available virtual machine sizes for a subscription](https://docs.microsoft.com/rest/api/compute/virtualmachines/virtualmachines_listall) (Вывод доступных размеров виртуальных машин для подписки)
- [List available virtual machine sizes in an availability set](https://docs.microsoft.com/rest/api/compute/availabilitysets/availabilitysets_listavailablesizes) (Вывод доступных размеров виртуальных машин в группе доступности)

## <a name="acu"></a>ACU

Узнайте больше о том, как с помощью [единиц вычислений Azure (ACU)](acu.md) сравнить производительность вычислений для различных номеров SKU Azure.

## <a name="benchmark-scores"></a>Результаты теста производительности

Узнайте больше о вычислительной мощности виртуальных машин Windows с помощью [результатов теста производительности CoreMark](compute-benchmark-scores.md).

## <a name="next-steps"></a>Дополнительная информация

Узнайте больше о различных доступных размерах виртуальных машин.
- [Универсальные](sizes-general.md)
- [Оптимизированные для вычислений](sizes-compute.md)
- [Оптимизированные для памяти](../virtual-machines-windows-sizes-memory.md)
- [Оптимизированные для хранилища](../virtual-machines-windows-sizes-storage.md)
- [Оптимизированные для GPU](sizes-gpu.md)
- [Для высокопроизводительных вычислений](sizes-hpc.md)
- Описание серий A Standard, Dv1 (D1–4 и D11–14 v1) и A8–A11 приведено на странице [Предыдущее поколение](sizes-previous-gen.md).




