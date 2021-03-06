---
title: Что такое служба API распознавания лиц?
titleSuffix: Azure Cognitive Services
description: В этой статье описана служба API распознавания лиц и объяснены связанные термины.
author: SteveMSFT
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: overview
ms.date: 10/11/2018
ms.author: sbowles
ms.openlocfilehash: 6c1e0d0a51bc01c581c05e7ce3215f5501b4be76
ms.sourcegitcommit: 3a02e0e8759ab3835d7c58479a05d7907a719d9c
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/13/2018
ms.locfileid: "49310376"
---
# <a name="what-is-the-face-api-service"></a>Что такое служба API распознавания лиц?

API распознавания лиц — это облачная служба, которая предоставляет алгоритмы для анализа лиц людей на фотографиях и видео. У API распознавания лиц две основные функции — определение лиц с атрибутами и распознавание лиц.

## <a name="face-detection"></a>Обнаружение лиц

API распознавания лиц обнаруживает до 64 лиц с высокой точностью нахождения на изображении. Изображение можно передать в виде файла (потока байтов) или с использованием допустимого URL-адреса.

![Обзор обнаружения лиц](./Images/Face.detection.jpg)

Для каждого обнаруженного лица возвращаются данные, определяющие прямоугольник (горизонтальная и вертикальная координаты, ширина и высота), который обозначает расположение лица на изображении. При необходимости функция обнаружения лиц извлекает ряд атрибутов, связанных с лицом, например поза, пол, возраст, положение головы, волосяной покров лица и очки. Дополнительные сведения см. в разделе о [функции обнаружения API распознавания лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

## <a name="face-recognition"></a>Распознавание лиц

Возможность распознавания лиц имеет важное значение во многих сценариях, включая обеспечение безопасности, анализ содержимого изображений и управление им, а также использование естественных пользовательских интерфейсов, мобильных приложений и робототехники. Служба API распознавания лиц предоставляет четыре функции распознания лиц — проверка лиц, поиск похожих лиц, группировка лиц и идентификация личности.

### <a name="face-verification"></a>Проверка лиц

Функция проверки лиц выполняет проверку идентичности двух обнаруженных лиц или одного обнаруженного лица по отношению к одному человеку. Дополнительные сведения см. в разделе о [функции проверки API распознавания лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a).

### <a name="finding-similar-faces"></a>Поиск похожих лиц

Получив целевое обнаруженное лицо и набор потенциальных лиц для поиска, служба находит небольшое количество лиц, очень похожих на целевое лицо. Поддерживаются два режима работы — **matchFace** и **matchPerson**. Режим **matchPerson** возвращает похожие лица после применения порогового значения для идентификации одного и того же человека, которое получено с помощью [Face — Verify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a). Режим **matchFace** игнорирует это пороговое значение и возвращает кандидатов с наиболее похожими лицами. В следующем примере показаны лица кандидатов.
![Обзор — поиск похожих лиц](./Images/FaceFindSimilar.Candidates.jpg) Это лицо использовано в качестве запроса.
![Обзор — поиск похожих лиц](./Images/FaceFindSimilar.QueryFace.jpg)

При поиске похожих лиц режим **matchPerson** вернет фотографии (а) и (b), на которых изображен тот же человек, что и на фотографии, использованной в качестве запроса. Режим **matchFace** вернет фотографии (a), (b), (c) и (d), т. е. четыре кандидата, хотя некоторые из них не очень похожи. Дополнительные сведения см. в разделе о [функции поиска похожих лиц API распознавания лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237).

### <a name="face-grouping"></a>Группировка лиц

Получив один набор неизвестных лиц, API группирования лиц автоматически разделяет их на несколько групп на основе сходства. Каждая группа — это отдельный ряд исходного набора неизвестных лиц, содержащий похожие лица. Все лица в одной группе могут считаться принадлежащими одному и тому же человеку. Дополнительные сведения см. в разделе о [функции группировки лиц API распознавания лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238).

### <a name="person-identification"></a>Идентификация личности

API распознавания лиц можно использовать для идентификации людей на основе обнаруженного лица и базы данных людей. Эту базу данных можно создать заранее и редактировать по мере использования.

На рисунке ниже показан пример базы данных с именем myfriends. Каждая группа может содержать до 1 000 000/10 000 объектов, соответствующих разным людям. В свою очередь, для каждого объекта, соответствующего одному человеку, можно зарегистрировать до 248 лиц.

![Обзор — большая группа людей или группа людей](./Images/person.group.clare.jpg)

Создав и обучив базу данных, вы можете идентифицировать новое обнаруженное лицо путем сравнения с группой. Если лицо определяется как принадлежащее человеку в группе, то возвращается объект, соответствующий этому человеку.

Дополнительные сведения об идентификации людей см. в следующих руководствах по API:

[Face — Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239)  
[PersonGroup — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244)  
[PersonGroup Person — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c)  
[PersonGroup — Train](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249)  
[LargePersonGroup — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d)  
[LargePersonGroup Person — Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adcba3a7b9412a4d53f40)  
[LargePersonGroup — Train](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4)  

#### <a name="face-storage-and-pricing"></a>Хранилище изображений лиц и цены

Хранилище изображений лиц в стандартной подписке позволяет хранить дополнительные сохраненные лица объектов "Человек" в большой или обычной группе людей ([добавление лица для объекта "Человек" в группе людей](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b)/[добавление лица для объекта "Человек" в большой группе людей](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42)) или обычных и больших списках лиц ([добавление лица в список лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250)/[добавление лица в большой список лиц](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3)) для идентификации и сопоставления подобия с помощью API распознавания лиц. За хранение изображений взимается плата в размере 0,50 долл. США за 1000 изображений лиц. Плата начисляется за каждый день. Подписка уровня "Бесплатный" ограничена 1 000 изображений человек.

Например, если в первой половине месяца в учетной записи ежедневно сохранялись 10 000 изображений лиц, а во второй такие изображения не сохранялись, плата будет взиматься за 10 000 изображений лиц согласно количеству дней хранения. Кроме того, если на протяжении месяца вы каждый день в течение нескольких часов храните 1000 изображений лиц, а затем каждую ночь удаляете эти изображения, плата все равно будет взиматься за 1000 изображений в день.

## <a name="sample-apps"></a>Примеры приложений

Взгляните на эти примеры приложений, которые используют API распознавания лиц.

- [API распознавания лиц Майкрософт: клиентская библиотека Windows с примерами](https://github.com/Microsoft/Cognitive-Face-Windows)
  - Пример приложения WPF, демонстрирующего несколько сценариев обнаружения, анализа и идентификации лиц.
- [Приложение UWP FamilyNotes](https://github.com/Microsoft/Windows-appsample-familynotes)
  - Пример приложения универсальной платформы Windows (UWP), показывающий использование речи, рукописного ввода, Cortana и камеры в сценарии совместного доступа к семейным заметкам.
- [Пример анализа видеокадров](https://github.com/microsoft/cognitive-samples-videoframeanalysis)
  - Пример приложения для архитектуры Win32, которое демонстрирует анализ потокового видео практически в реальном времени с помощью API распознавания лиц, API компьютерного зрения и API распознавания эмоций.

## <a name="tutorials"></a>Учебники
В следующих руководствах описаны основные возможности API распознавания лиц и процессы создания подписок:
- [Руководство по началу работы с API распознавания лиц в CSharp](Tutorials/FaceAPIinCSharpTutorial.md)
- [Руководство по началу работы с API распознавания лиц в Java для Android](Tutorials/FaceAPIinJavaForAndroidTutorial.md)
- [Руководство по началу работы с API распознавания лиц в Python](Tutorials/FaceAPIinPythonTutorial.md)

## <a name="next-steps"></a>Дополнительная информация

Попробуйте краткое руководство по реализации простого сценария с использованием API распознавания лиц.
- [Краткое руководство. Обнаружение лиц на изображении с помощью C#](quickstarts/csharp.md) (доступны руководства для других языков)
