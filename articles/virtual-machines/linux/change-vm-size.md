---
title: Как изменить размер виртуальной машины Linux с помощью интерфейса командной строки Azure | Документация Майкрософт
description: В статье рассматривается, как увеличить и уменьшить масштаб виртуальной машины Linux, изменяя ее размер.
services: virtual-machines-linux
documentationcenter: na
author: mikewasson
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: e163f878-b919-45c5-9f5a-75a64f3b14a0
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/10/2017
ms.author: mwasson
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0b8802c91ceb59d391dc27a71da905de9c15a1dc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993235"
---
# <a name="resize-a-linux-virtual-machine-using-azure-cli"></a>Изменение размера виртуальной машины Linux с помощью интерфейса командной строки Azure 

После того как вы подготовили виртуальную машину, ее можно увеличить или уменьшить, изменив [размер][vm-sizes]. В некоторых случаях сначала необходимо освободить виртуальную машину. Необходимо завершить общение с виртуальной машиной, если требуемый размер недоступен в кластере оборудования, в котором она размещена. В этой статье описано, как изменить размер виртуальной машины Linux с помощью интерфейса командной строки Azure. 

## <a name="resize-a-vm"></a>Изменение размера виртуальной машины
Чтобы изменить размер виртуальной машины, нужно установить последнюю версию [интерфейса командной строки Azure](/cli/azure/install-az-cli2) и войти в учетную запись Azure с помощью команды [az login](/cli/azure/reference-index#az_login).

1. Выведите список размеров виртуальной машины, доступных в кластере оборудования, в котором она размещена, с помощью команды [az vm list-vm-resize-options](/cli/azure/vm#az_vm_list_vm_resize_options). В следующем примере содержится список размеров виртуальных машин с именем `myVM` в группе ресурсов `myResourceGroup` региона:
   
    ```azurecli
    az vm list-vm-resize-options --resource-group myResourceGroup --name myVM --output table
    ```

2. Если в списке есть нужный размер, выполните команду [az vm resize](/cli/azure/vm#az_vm_resize), чтобы изменить размер виртуальной машины. В следующем примере размер виртуальной машины с именем `myVM` изменяется до размера `Standard_DS3_v2`:
   
    ```azurecli
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    ```
   
    Во время этого процесса виртуальная машина перезагружается. После перезагрузки существующие диски ОС и данных переназначаются. Данные на временном диске будут утеряны.

3. Если требуемый размер виртуальной машины отсутствует в списке, сначала необходимо завершить общение с виртуальной машиной, используя команду [az vm deallocate](/cli/azure/vm#az_vm_deallocate). Этот процесс позволяет изменить размер виртуальной машины до любого значения, которое поддерживает регион, а затем перезапустить ее. В следующих шагах завершается общение с виртуальной машиной `myVM` в группе ресурсов `myResourceGroup`, она масштабируется, а затем запускается.
   
    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    az vm start --resource-group myResourceGroup --name myVM
    ```
   
   > [!WARNING]
   > Освобождение виртуальной машины также освобождает все назначенные ей динамические IP-адреса. Это не влияет на диски ОС и данных.

## <a name="next-steps"></a>Дополнительная информация
Для дополнительной масштабируемости запустите несколько экземпляров виртуальных машин и разверните их. Дополнительные сведения см. в статье [Автоматическое масштабирование машин Linux в наборе масштабирования виртуальных машин][scale-set]. 

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[scale-set]: ../../virtual-machine-scale-sets/virtual-machine-scale-sets-linux-autoscale.md 
[vm-sizes]:sizes.md
