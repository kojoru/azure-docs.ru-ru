---
title: Потоковые данные в качестве входных данных Azure Stream Analytics
description: Узнайте больше о настройке подключения данных в Azure Stream Analytics. К входным данным относятся поток данных из событий, а также справочные данные.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/27/2018
ms.openlocfilehash: 556b71d8312c7661b58a3ccf56fd76f6753ef9cf
ms.sourcegitcommit: c2c279cb2cbc0bc268b38fbd900f1bac2fd0e88f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49987585"
---
# <a name="stream-data-as-input-into-stream-analytics"></a>Потоковые данные в качестве входных данных Stream Analytics

Stream Analytics полностью интегрируется с потоками данных Azure в качестве входных данных трех типов ресурсов.
- [Центры событий Azure](https://azure.microsoft.com/services/event-hubs/)
- [Центр Интернета вещей Azure](https://azure.microsoft.com/services/iot-hub/) 
- [хранилище BLOB-объектов Azure](https://azure.microsoft.com/services/storage/blobs/) 

Эти ресурсы входных данных могут существовать в той же подписке Azure, что и задание Stream Analytics, или другой подписке.

### <a name="compression"></a>Сжатие
Stream Analytics поддерживает функцию сжатия во всех источниках входных потоковых данных. Сейчас поддерживаются ссылочные типы None, GZip и Deflate. Для ссылочных данных сжатие не поддерживается. Если у входных данных формат AVRO, они сжимаются и выполняется их прозрачная обработка. Для сериализации Avro не требуется указывать тип сжатия. 

## <a name="create-edit-or-test-inputs"></a>Создание, изменение или проверка входных данных
С помощью [портала Azure](https://portal.azure.com) вы можете [создавать входные данные](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-quick-create-portal#configure-input-to-the-job) и просматривать или изменять их в задании потоковой передачи. Вы также можете проверять подключения к источникам входных данных и [запросы](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-manage-job#test-your-query) на основе примеров данных. При написании запроса укажите входные данные в предложении FROM. Список доступных входных данных можно получить на странице **Запрос** на портале. Чтобы использовать несколько источников входных данных, объедините их с помощью параметра `JOIN` или напишите несколько запросов `SELECT`.


## <a name="stream-data-from-event-hubs"></a>Потоковая передача данных из Центров событий

Центры событий Azure предоставляют высокомасштабируемую службу приема данных о событиях публикации и подписки. Концентратор событий может обрабатывать миллионы событий в секунду, позволяя вам обрабатывать и анализировать огромное количество данных, создаваемых подключенными устройствами и приложениями. Используя Центры событий со службой Stream Analytics, вы получаете комплексное решение для анализа данных в режиме реального времени. Центры событий позволяют передавать события в Azure в режиме реального времени, где задания Stream Analytics также обрабатывают эти события в режиме реального времени. Например, в Центры событий можно отправлять сведения о щелчках, показания датчиков или журналы сетевых событий. Затем можно создать задания Stream Analytics, которые используют Центры событий в качестве входных потоков данных для фильтрации в режиме реального времени, выполнения агрегации и корреляции.

`EventEnqueuedUtcTime` — это метка времени поступления события в концентратор, которая также является меткой времени по умолчанию для событий, поступающих из Центров событий в Stream Analytics. Для обработки данных как потока с помощью метки времени в полезных данных события необходимо использовать ключевое слово [TIMESTAMP BY](https://msdn.microsoft.com/library/azure/dn834998.aspx).

### <a name="consumer-groups"></a>Группы получателей
Каждый концентратор событий Stream Analytics нужно настроить таким образом, чтобы у него была собственная группа получателей. Если задание содержит самосоединение или несколько источников входных данных, некоторые входные данные могут последовательно считываться несколькими модулями чтения. Эта ситуация влияет на количество модулей чтения в группе получателей. Чтобы не превысить лимит на количество модулей чтения для Центров событий (5 на каждую группу получателей в разделе), рекомендуется назначить группу получателей для каждого задания Stream Analytics. У каждого концентратора событий должно быть не более 20 групп получателей. Дополнительные сведения см. в разделе об [устранении неполадок входных данных Azure Stream Analytics](stream-analytics-troubleshoot-input.md).

### <a name="stream-data-from-event-hubs"></a>Потоковая передача данных из Центров событий
В следующей таблице описываются все параметры на странице **Новые входные данные** на портале Azure для передачи потока входных данных из концентратора событий.

| Свойство | ОПИСАНИЕ |
| --- | --- |
| **Псевдоним входных данных** |Понятное имя, используемое в запросах задания для ссылки на эти входные данные. |
| **Подписка** | Выберите подписку, в которой существует ресурс концентратора событий. | 
| **Пространство имен концентратора событий** | Пространство имен концентратора событий — это контейнер для набора сущностей обмена сообщениями. При создании нового концентратора событий также создается пространство имен. |
| **Имя концентратора событий** | Имя концентратора событий для использования в качестве источника входных данных. |
| **Имя политики концентратора событий** | Политика общего доступа, которая предоставляет доступ к концентратору событий. Каждой политике общего доступа присваивается имя, а также для нее задаются разрешения и ключи доступа. Этот параметр автоматически заполняется, если только не указан параметр "Указать настройки концентратора событий вручную".|
| **Группа получателей концентратора событий** (рекомендуется) | Для каждого задания Stream Analytics настоятельно рекомендуется использовать отдельную группу получателей. Эта строка указывает группу получателей, принимающих данные из концентратора событий. Если группа получателей не указана, задание Stream Analytics использует группу получателей "$Default".  |
| **Формат сериализации событий** | Формат сериализации (JSON, CSV или Avro) входного потока данных.  Убедитесь, что формат JSON совпадает со спецификацией и не содержит ведущий "0" в десятичных числах. |
| **Кодирование** | Сейчас UTF-8 — единственный поддерживаемый формат кодировки. |
| **Тип сжатия событий** | Тип сжатия используется для чтения таких входящих потоков данных, как None (по умолчанию), GZip или Deflate. |

При поступлении входных данных из потока данных концентратора событий запрос Stream Analytics может получить доступ к следующим полям метаданных:

| Свойство | ОПИСАНИЕ |
| --- | --- |
| **EventProcessedUtcTime** |Дата и время обработки события службой Stream Analytics. |
| **EventEnqueuedUtcTime** |Дата и время получения события Центрами событий. |
| **PartitionId** |Идентификатор секции для входного адаптера (нумерация идет от нуля). |

Например, используя эти поля, можно писать запросы, как в следующем примере:

```sql
SELECT
    EventProcessedUtcTime,
    EventEnqueuedUtcTime,
    PartitionId
FROM Input
```

> [!NOTE]
> При использовании концентратора события в качестве конечной точки для маршрутов Центра Интернета вещей вы можете получить доступ к метаданным Центра Интернета вещей с помощью [функции GetMetadataPropertyValue](https://msdn.microsoft.com/library/azure/mt793845.aspx).
> 

## <a name="stream-data-from-iot-hub"></a>Потоковая передача данных из Центра Интернета вещей
Центр Azure IoT — это высокомасштабируемая служба приема данных о событиях публикации и подписки, оптимизированная под сценарии "Интернет вещей".

По умолчанию метка времени событий, поступающих из Центра Интернета вещей в Stream Analytics, — это метка времени поступления события в концентратор Центра Интернета вещей, то есть `EventEnqueuedUtcTime`. Для обработки данных как потока с помощью метки времени в полезных данных события необходимо использовать ключевое слово [TIMESTAMP BY](https://msdn.microsoft.com/library/azure/dn834998.aspx).

### <a name="consumer-groups"></a>Группы получателей
Каждый Центр Интернета вещей Stream Analytics нужно настроить таким образом, чтобы у него была собственная группа получателей. Если задание включает самосоединение или несколько источников входных данных, некоторые входные данные могут последовательно считываться несколькими модулями чтения. Эта ситуация влияет на количество модулей чтения в группе получателей. Чтобы не превысить лимит на количество модулей чтения для Центра Интернета вещей (5 на каждую группу получателей в разделе), рекомендуется назначить группу получателей для каждого задания Stream Analytics.

### <a name="configure-an-iot-hub-as-a-data-stream-input"></a>Настройка Центра Интернета вещей в качестве входного потока данных
В следующей таблице описываются все параметры на странице **Новые входные данные** на портале Azure при настройке Центра Интернета вещей в качестве потокового входа.

| Свойство | ОПИСАНИЕ |
| --- | --- |
| **Псевдоним входных данных** | Понятное имя, используемое в запросах задания для ссылки на эти входные данные.|
| **Подписка** | Выберите подписку, в которой существуют ресурсы Центра Интернета вещей. | 
| **Центр Интернета вещей** | Имя Центра Интернета вещей для использования в качестве источника входных данных. |
| **Конечная точка** | Конечная точка для Центра Интернета вещей.|
| **Имя политики общего доступа** | Политика общего доступа, которая предоставляет доступ к Центру Интернета вещей. Каждой политике общего доступа присваивается имя, а также для нее задаются разрешения и ключи доступа. |
| **Ключ политики общего доступа** | Ключ общего доступа, используемый для авторизации доступа к Центру Интернета вещей.  Этот параметр автоматически заполняется, если только не будет указан вариант ручной настройки параметров Центра Интернета вещей. |
| **Группа потребителей** | Для каждого задания Stream Analytics настоятельно рекомендуется использовать отдельную группу получателей. Группа получателей используется для принимающих данных из Центра Интернета вещей. Stream Analytics использует группу получателей "$Default", если не указано иное.  |
| **Формат сериализации событий** | Формат сериализации (JSON, CSV или Avro) входного потока данных.  Убедитесь, что формат JSON совпадает со спецификацией и не содержит ведущий "0" в десятичных числах. |
| **Кодирование** | Сейчас UTF-8 — единственный поддерживаемый формат кодировки. |
| **Тип сжатия событий** | Тип сжатия используется для чтения таких входящих потоков данных, как None (по умолчанию), GZip или Deflate. |


При использовании потоковой передачи данных из Центра Интернета вещей запрос Stream Analytics может получить доступ к следующим полям метаданных:

| Свойство | ОПИСАНИЕ |
| --- | --- |
| **EventProcessedUtcTime** | Дата и время обработки события. |
| **EventEnqueuedUtcTime** | Дата и время получения события Центром Интернета вещей. |
| **PartitionId** | Идентификатор секции для входного адаптера (нумерация идет от нуля). |
| **IoTHub.MessageId** | Идентификатор, используемый для корреляции двустороннего обмена данными в Центре Интернета вещей. |
| **IoTHub.CorrelationId** | Идентификатор, используемый в ответах на сообщение и отзывах в Центре Интернета вещей. |
| **IoTHub.ConnectionDeviceId** | Идентификатор проверки подлинности, используемый для отправки этого сообщения. Это значение помещается в сообщения, связанные со службой, Центром Интернета вещей. |
| **IoTHub.ConnectionDeviceGenerationId** | Идентификатор создания устройства, прошедшего проверку подлинности, используемый для отправки этого сообщения. Это значение помещается в сообщения, связанные со службой, Центром Интернета вещей. |
| **IoTHub.EnqueuedTime** | Время, когда Центр Интернета вещей получил сообщение. |
| **IoTHub.StreamId** | Настраиваемое свойство события, добавленное устройством отправителя. |


## <a name="stream-data-from-blob-storage"></a>Потоковая передача данных из хранилища BLOB-объектов
Хранилище больших двоичных объектов Azure служит экономичным и масштабируемым решением в сценариях, связанных с хранением больших объемов неструктурированных данных в облаке. Данные в хранилище BLOB-объектов обычно считаются неактивными, но служба Stream Analytics может обрабатывать эти данные как поток данных. 

Обработка журналов — это часто используемый сценарий для обработки входных данных хранилища больших двоичных объектов с помощью Stream Analytics. В этом сценарии файлы данных телеметрии, полученные из системы, необходимо проанализировать и обработать, чтобы извлечь значимые данные.

По умолчанию метка времени событий хранилища больших двоичных объектов в Stream Analytics — это метка времени последнего изменения большого двоичного объекта, то есть `BlobLastModifiedUtcTime`. Для обработки данных как потока с помощью метки времени в полезных данных события необходимо использовать ключевое слово [TIMESTAMP BY](https://msdn.microsoft.com/library/azure/dn834998.aspx). Задание Stream Analytics извлекает данные из входных данных хранилища BLOB-объектов Azure каждую секунду, если доступен файл большого двоичного объекта. В случае, если этот файл недоступен, применяется экспоненциально увеличивающаяся задержка с максимальным значением, равным 90 секундам.

Для входных данных в формате CSV *необходимо*, чтобы строка заголовка определяла поля для набора данных и все поля строк заголовка были уникальными.

На данный момент в Stream Analytics не поддерживаются сообщения десериализации библиотеки AVRO, созданные захватом концентратора событий или пользовательской конечной точкой контейнера службы хранилища Azure Центра Интернета вещей.

> [!NOTE]
> Stream Analytics не поддерживает добавление содержимого в существующий файл большого двоичного объекта. Stream Analytics просматривает каждый файл только один раз, и любые изменения, которые произойдут в нем после того, как задание прочитает данные, не обрабатываются. Мы рекомендуем отправлять все данные для файла большого двоичного объекта за один раз, а затем добавлять более новые события в другой новый файл большого двоичного объекта.
> 

### <a name="configure-blob-storage-as-a-stream-input"></a>Настройка хранилища BLOB-объектов в качестве потокового входа 

В следующей таблице описываются все параметры на странице **Новые входные данные** на портале Azure при настройке хранилища больших двоичных объектов в качестве потокового входа.

| Свойство | ОПИСАНИЕ |
| --- | --- |
| **Псевдоним входных данных** | Понятное имя, используемое в запросах задания для ссылки на эти входные данные. |
| **Подписка** | Выберите подписку, в которой существуют ресурсы Центра Интернета вещей. | 
| **Учетная запись хранения** | Имя учетной записи хранения, в которой находятся файлы больших двоичных объектов. |
| **Ключ учетной записи хранения** | Секретный ключ, связанный с учетной записью хранения. Этот параметр автоматически заполняется, если только не будет указан вариант ручной настройки параметров хранилища больших двоичных объектов. |
| **Контейнер** | Контейнер для входных данных большого двоичного объекта. Контейнеры обеспечивают логическую группировку BLOB-объектов, хранящихся в службе BLOB-объектов Microsoft Azure. При передаче большого двоичного объекта в службу хранилища BLOB-объектов для него необходимо указать контейнер. Чтобы создать новый контейнер, вы можете выбрать параметр **Use existing** (Использование имеющихся) контейнеров или **Создать**.|
| **Шаблон пути** (необязательно) | Путь к файлу, используемый для поиска больших двоичных объектов в указанном контейнере. В пути можно указать один или более экземпляров следующих трех переменных: `{date}`, `{time}` или `{partition}`.<br/><br/>Пример 1: `cluster1/logs/{date}/{time}/{partition}`<br/><br/>Пример 2: `cluster1/logs/{date}`<br/><br/>Символ `*` является недопустимым значением для префикса пути. Допустимыми являются только <a HREF="https://msdn.microsoft.com/library/azure/dd135715.aspx">символы больших двоичных объектов Azure</a>. Это не касается имен контейнеров и имен файлов. |
| **Формат даты** (необязательное свойство) | При использовании переменной даты в пути это формат даты, по которому упорядочены файлы. Пример: `YYYY/MM/DD` |
| **Формат времени** (необязательное свойство) |  При использовании переменной времени в пути это формат времени, в котором размещаются файлы. В настоящее время единственным поддерживаемым значением в течение нескольких часов является `HH`. |
| **Формат сериализации событий** | Формат сериализации (JSON, CSV или Avro) входного потока данных.  Убедитесь, что формат JSON совпадает со спецификацией и не содержит ведущий "0" в десятичных числах. |
| **Кодирование** | В настоящее время единственным поддерживаемым форматом кодирования файлов CSV и JSON является UTF-8. |
| **Сжатие** | Тип сжатия используется для чтения таких входящих потоков данных, как None (по умолчанию), GZip или Deflate. |

При поступлении данных из хранилища больших двоичных объектов запрос Stream Analytics может получить доступ к следующим полям метаданных:

| Свойство | ОПИСАНИЕ |
| --- | --- |
| **BlobName** |Имя входного большого двоичного объекта, от которого поступило событие. |
| **EventProcessedUtcTime** |Дата и время обработки события службой Stream Analytics. |
| **BlobLastModifiedUtcTime** |Дата и время последнего изменения большого двоичного объекта. |
| **PartitionId** |Идентификатор секции для входного адаптера (нумерация идет от нуля). |

Например, используя эти поля, можно писать запросы, как в следующем примере:

```sql
SELECT
    BlobName,
    EventProcessedUtcTime,
    BlobLastModifiedUtcTime
FROM Input
```

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Краткое руководство по созданию задания Stream Analytics с помощью портала Azure](stream-analytics-quick-create-portal.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: https://go.microsoft.com/fwlink/?LinkID=513299
[stream.analytics.rest.api.reference]: https://go.microsoft.com/fwlink/?LinkId=517301