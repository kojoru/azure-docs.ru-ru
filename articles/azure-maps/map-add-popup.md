---
title: Добавление всплывающего окна с помощью службы "Карты Azure" | Документация Майкрософт
description: Как добавить всплывающее окно на карту в JavaScript
author: jingjing-z
ms.author: jinzh
ms.date: 09/17/2018
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 76a7e230491d5e524a1d73437a56d12594cfebe2
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127443"
---
# <a name="add-a-popup-to-the-map"></a>Добавление всплывающего окна на карту

В этой статье объясняется, как добавить всплывающее окно на карту.  

## <a name="understand-the-code"></a>Изучение кода

<a id="addAPopup"></a>

<iframe height='500' scrolling='no' title='Добавление всплывающего окна на карту' src='//codepen.io/azuremaps/embed/zRyKxj/?height=545&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Просмотрите фрагмент кода <a href='https://codepen.io/azuremaps/pen/zRyKxj/'>Добавление всплывающего окна на карту</a> службы "Карты Azure" (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) в <a href='https://codepen.io'>CodePen</a>.
</iframe>

В первом блоке приведенного выше кода создается объект карты. См. инструкции по [созданию карты](./map-create.md).

Во втором блоке кода создается закрепление, добавляемое на карту. См. инструкции по [добавлению закреплений на карту](./map-add-pin.md).

В третьем блоке кода создается содержимое, которое будет отображаться во всплывающем окне. Содержимое всплывающего окна представляет собой HTML-элемент.

В четвертом блоке кода создается [объект всплывающего окна](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest) с помощью `new atlas.Popup()`. Такие свойства всплывающего окна, как содержимое и положение, входят в класс [PopupOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/models.popupoptions?view=azure-iot-typescript-latest). PopupOptions можно определить в конструкторе выплывающего окна или с помощью функции [setPopupOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest#setpopupoptions) класса Popup.

В последнем блоке кода используется функция [addEventListener](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#addeventlistener) класса Map для ожидания передачи данных о событии наведения указателя мыши на закрепление. В нем также используется функция [open](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest#open) класса Popup, чтобы открыть всплывающее окно, когда возникает это событие.

## <a name="next-steps"></a>Дополнительная информация

Дополнительные сведения о классах и методах, которые используются в этой статье:

> [!div class="nextstepaction"]
> класс [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest);

> [!div class="nextstepaction"]
> [Popup](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.popup?view=azure-iot-typescript-latest)

Полные примеры кода см. в следующих превосходных статьях:

> [!div class="nextstepaction"]
> [Добавление фигуры](./map-add-shape.md)

> [!div class="nextstepaction"]
> [Добавление пользовательского HTML-кода](./map-add-custom-html.md)
