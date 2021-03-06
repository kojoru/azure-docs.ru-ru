---
title: Как использовать альтернативные входные данные в Conversation Learner (Microsoft Cognitive Services) | Документация Майкрософт
titleSuffix: Azure
description: Сведения об использовании альтернативных входных данных в Conversation Learner.
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: 8d3b3f419ceacbb9a6fe2b19cf68ea6873de536f
ms.sourcegitcommit: 4e5ac8a7fc5c17af68372f4597573210867d05df
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/20/2018
ms.locfileid: "39171023"
---
# <a name="how-to-use-alternative-inputs"></a>Как использовать альтернативные входные данные

В этом руководстве показано, как использовать поле альтернативных входных данных для данных, вводимых пользователем в интерфейсе обучения.

## <a name="video"></a>Видео

[![Предварительная версия руководства 8](http://aka.ms/cl-tutorial-08-preview)](http://aka.ms/blis-tutorial-08)

## <a name="requirements"></a>Требования
Для работы с этим руководством требуется запущенный бот обучения общего назначения.

    npm run tutorial-general

## <a name="details"></a>Сведения
Альтернативные входные данные — это другие высказывания, которые пользователь может произносить в определенной точке диалога обучения. Такие данные позволяют более компактно описать отклонения от основного потока диалога, не создавая отдельный обучающий диалог для каждого варианта.

## <a name="steps"></a>Действия

### <a name="create-the-model"></a>Создание модели

1. В пользовательском веб-интерфейсе щелкните "Новая модель".
2. В поле имени введите AlternativeInputs. Затем щелкните "Create" (Создать).

### <a name="create-an-entity"></a>Создание сущности

1. Щелкните Entities (Сущности), затем New Entity (Новая сущность).
2. В поле "Entity Name" (Имя сущности) введите city.
3. Щелкните Создать.

### <a name="create-three-actions"></a>Создание трех действий

1. Щелкните "Actions" (Действия) и "New Action" (Создать действие).
2. В поле "Response" (Ответ) введите "Which city do you want?" (Какой город вам нужен?).
3. В поле "Disqualifying Entities" (Блокирующие сущности) введите $city.
3. Нажмите кнопку "Создать"

Теперь создайте второе действие.

1. Щелкните Actions (Действия), а затем New Action (Новое действие).
3. В поле Response (Ответ) введите The weather in $city is probably sunny (Погода в городе $city скорее всего солнечная).
4. В поле Required Entities (Обязательные сущности) введите $city.
4. Щелкните Создать.

Создайте третье действие.

1. Щелкните Actions (Действия) и New Action (Новое действие).
3. В поле Response (Ответ) введите "Try asking for the weather" (Попробуй спросить о погоде).
    - Это будет ответом на такие вопросы пользователя, как "What can the system do? (Что умеет эта система?).
4. В поле "Disqualifying Entities" (Блокирующие сущности) введите $city.
4. Нажмите кнопку "Создать"

Теперь у вас есть три действия.

### <a name="train-the-bot"></a>Обучение бота

1. Щелкните Train Dialogs (Обучение диалогам), а затем New Train Dialog (Обучение новому диалогу).
2. Введите "What's the weather" (Какая погода?).
3. Щелкните "Score Actions" (Оценить действия) и выберите "Which city do you want?" (Какой город вам нужен?)
2. Введите denver (Денвер).
3. Дважды щелкните denver (Денвер) и выберите сущность city.
    - Выбранный текст помечается как сущность city.
5. Щелкните "Score Actions" (Оценить действия).
    - Denver (Денвер) теперь отображается в сущности city. 
6. Выберите The weather in $city is probably sunny (Погода в городе $city скорее всего солнечная).
7. Щелкните Done Teaching (Завершить обучение).

Добавьте другой пример диалога.

1. Щелкните "New Action" (Новое действие), а затем — "New Train Dialog" (Обучение новому диалогу).
2. Введите "What can you do?" (Что ты умеешь делать?).
3. Щелкните "Score Actions" (Оценить действия) и выберите "Try asking for the weather" (Попробуй спросить о погоде).
2. Введите "What's the weather in Seattle?" (Какая погода в Сиэтле?).
3. Дважды щелкните Seattle (Сиэтл) и выберите сущность city.
    - Выбранный текст помечается как сущность city.
5. Щелкните "Score Actions" (Оценить действия).
    - Seattle (Сиэтл) теперь отображается в сущности city. 
6. Выберите The weather in $city is probably sunny (Погода в городе $city скорее всего солнечная).
7. Щелкните "Done Teaching" (Завершить обучение).

Давайте посмотрим, что произойдет, если пользователь произнесет другую фразу, семантически аналогичную введенной выше.

1. Щелкните "New Action" (Новое действие), а затем — "New Train Dialog" (Обучение новому диалогу).
2. Введите "help" (Справка).
3. Щелкните Score Actions (Оценить действия).
    - Для двух возможных ответов назначены очень близкие оценки. Это означает, что модель не уверена в выборе между этими двумя действиями.
6. Щелкните Abandon Teaching (Отменить обучение), а затем Confirm (Подтвердить).

![](../media/tutorial8_closescores.png)

В этом случае будет полезным добавить в диалоги альтернативные входные данные. Вы можете это сделать сразу после завершения обучения. Вы также можете добавить их в любое время позже.

2. Щелкните "What can you do?" (Что ты умеешь делать?) в разделе Train Dialogs (Обучение диалогам).
2. В существующем диалоге щелкните "What can you do?" (Что ты умеешь делать?), чтобы выбрать этот шаг.
    1. В панели справа "Entity Detection" (Обнаружение сущности) введите в поле "Add alternative input" (Добавьте альтернативные входные данные) несколько дополнительных вариантов.
    1. Введите "What are my choices?" (Что я могу здесь сделать?).
    2. Введите "Tell me my choices?" (Предоставь возможные варианты).
    3. Введите "help"(Справка).
    1. щелкните "Submit Changes" (Отправить изменения).


![](../media/tutorial8_helpalternates.png)

2. Теперь щелкните "What's the weather in Seattle?" (Какая погода в Сиэтле?).
    1. В поле Add alternative input (Добавьте альтернативные входные данные) введите "forecast for seattle" (прогноз для Сиэтла).
    2. Дважды щелкните Seattle (Сиэтл) и выберите сущность city. Для альтернативных входных данных нужно определить такие же сущности, как для основного варианта. Но содержимое этих сущностей может отличаться.
    3. В поле "Add alternative input" (Добавьте альтернативные входные данные) введите "will it rain today in denver" (будет ли сегодня дождь в Денвере).
    4. Щелкните denver (Денвер) и выберите сущность city.
    5. Щелкните "Submit Changes" (Отправить изменения), а затем "Done" (Готово).


Теперь давайте добавим альтернативные входные данные в первый диалог.

1. Щелкните "Train Dialogs" (Обучение диалогам).
2. Щелкните диалог, который начинается с шага "what's the weather" (какая погода).
2. Щелчком выберите "what's the weather" (какая погода) на панели слева.
    1. В поле Add alternative input (Добавьте альтернативные входные данные) введите "weather forecast" (прогноз погоды).
    2. Введите "will it rain?" (будет ли дождь?).
    3. Щелкните "Submit Changes" (Отправить изменения).
4. Щелчком выберите "denver" в области слева:
    1. В поле "Add alternative input" (Добавьте альтернативные входные данные) введите "for denver" (для Денвера).
    2. Введите "forecast for austin" (прогноз для Остина).
        - Выделена полная фраза. Щелкните эту фразу, а затем щелкните красный крестик. Теперь выберите "Austin" (Остин) и щелкните сущность city.
        - Щелкните "Submit Changes" (Отправить изменения).
    1. Щелкните "Done" (Готово), чтобы запустить повторное обучение модели.

![](../media/tutorial8_altcities.png)

Давайте попробуем новые варианты.

1. Щелкните "New Train Dialog" (Обучение новому диалогу).
2. Введите "what are you capabilities" (какие у тебя возможности?).
3. Щелкните Score Actions (Оценить действия).
    - Оценки следующего действия стали более высокими, что демонстрирует уверенность модели.
2. Выберите "Try asking for weather" (Попробуй спросить о погоде).
6. Щелкните "Done Teaching" (Завершить обучение).

Итак, вы узнали, как с помощью альтернативных входных данных сообщить системе дополнительные фразы, которые может сказать пользователь. Это позволит не создавать множество дополнительных почти идентичных диалогов, свернув их в одну версию диалога с перечислением разных фраз пользователя.

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Запись диалогов](./9-log-dialogs.md)
