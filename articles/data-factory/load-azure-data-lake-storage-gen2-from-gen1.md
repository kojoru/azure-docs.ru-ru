---
title: Копирование данных из Azure Data Lake Storage 1-го поколения в Azure Data Lake Storage 2-го поколения (предварительная версия) с помощью Фабрики данных Azure
description: Использование Фабрики данных Azure для копирования данных из Azure Data Lake Storage 1-го поколения в Azure Data Lake Storage 2-го поколения (предварительная версия)
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 07/06/2018
ms.author: jingwang
ms.openlocfilehash: 953585ffcc5a40d9ae48055f68a1c1fa84db25cc
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/03/2018
ms.locfileid: "48249338"
---
# <a name="copy-data-from-azure-data-lake-storage-gen1-to-gen2-preview-with-azure-data-factory"></a>Копирование данных из Azure Data Lake Storage 1-го поколения в Azure Data Lake Storage 2-го поколения (предварительная версия) с помощью Фабрики данных Azure

[Хранилище Azure Data Lake Storage 2-го поколения (предварительная версия)](../storage/data-lake-storage/introduction.md) добавляет протокол иерархической файловой системы пространства имен и средства безопасности хранилища BLOB-объектов Azure, что упрощает подключение платформы аналитики к долговременному уровню хранения. В Data Lake Storage Gen2 (предварительная версия) сохранены все качества хранения объектов при добавлении преимуществ файловой системы.

Если вы используете Azure Data Lake Storage 1-го поколения, оцените новые возможности 2-го поколения, скопировав данные из Data Lake Storage 1-го поколения в Data Lake Storage 2-го поколения с помощью Фабрики данных Azure.

Фабрика данных Azure — это полностью управляемая облачная служба интеграции данных. Эту службу можно использовать для заполнения озера данными из богатого набора локальных и облачных хранилищ данных и экономии времени при создании аналитических решений. Дополнительные сведения о поддерживаемых соединителях, см. в таблице [Поддерживаемые хранилища данных и форматы](copy-activity-overview.md#supported-data-stores-and-formats).

Фабрика данных Azure предлагает масштабируемое и управляемое решение для перемещения данных. Благодаря архитектуре горизонтального масштабирования ADF, фабрика данных Azure может использовать высокую пропускную способность для приема данных. Дополнительные сведения см. в руководстве [по настройке производительности действия копирования](copy-activity-performance.md).

В этой статье показано, как с помощью средства копирования данных службы "Фабрика данных" скопировать данные из _Azure Data Lake Storage 1-го поколения_ в _Azure Data Lake Storage 2-го поколения_. Чтобы копировать данные из других типов хранилищ, необходимо выполнить аналогичные шаги.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
* Учетная запись Azure Data Lake Storage 1-го поколения с сохраненными в ней данными.
* Учетная запись службы хранилища с включенным хранилищем Azure Data Lake Storage Gen2. Если у вас еще нет учетной записи хранилища, создайте ее щелкнув [здесь](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).

## <a name="create-a-data-factory"></a>Создание фабрики данных

1. В меню слева выберите **Создать** > **Data + Analytics** (Данные и аналитика) > **Фабрика данных**:
   
   ![Создание фабрики данных](./media/load-azure-data-lake-storage-gen2-from-gen1/new-azure-data-factory-menu.png)
2. В полях на странице **Новая фабрика данных** задайте значения, как показано на следующем изображении: 
      
   ![Страница "Новая фабрика данных"](./media/load-azure-data-lake-storage-gen2-from-gen1/new-azure-data-factory.png)
 
    * **Имя.** Введите глобальное уникальное имя фабрики данных Azure. Если отобразится сообщение об ошибке "Имя \"LoadADLSDemo\" фабрики данных недоступно", введите другое имя. Например, вы можете использовать имя _**ваше_имя**_**ADFTutorialDataFactory**. Попробуйте создать фабрику данных еще раз. Правила именования артефактов службы "Фабрика данных" см. в [этой](naming-rules.md) статье.
    * **Подписка.** Выберите подписку Azure, в рамках которой нужно создать фабрику данных. 
    * **Группа ресурсов.** Выберите имеющуюся группу ресурсов из раскрывающегося списка или щелкните параметр **Create new** (Создать) и введите имя группы ресурсов. Сведения о группах ресурсов см. в статье, где описывается [использование групп ресурсов для управления ресурсами Azure](../azure-resource-manager/resource-group-overview.md).  
    * **Версия.** Выберите значение **V2**.
    * **Расположение.** Выберите расположение фабрики данных. В раскрывающемся списке отображаются только поддерживаемые расположения. Хранилища данных, используемые в фабрике данных, могут находиться в других расположениях и регионах. 

3. Нажмите кнопку **Создать**.
4. После создания перейдите к фабрике данных. Вы увидите домашнюю страницу **фабрики данных**, как показано на следующем изображении: 
   
   ![Домашняя страница фабрики данных](./media/load-azure-data-lake-storage-gen2-from-gen1/data-factory-home-page.png)

   Выберите плитку **Создание и мониторинг**, чтобы открыть на отдельной вкладке приложение интеграции данных.

## <a name="load-data-into-azure-data-lake-storage-gen2"></a>Загрузка данных в Azure Data Lake Storage Gen2

1. Чтобы запустить средство копирования данных, на странице **Get started** (Начало работы) выберите плитку **Copy Data** (Копирование данных): 

   ![Плитка средства копирования данных](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-tile.png)
2. На странице **Properties** (Свойства) укажите **CopyFromADLSGen1ToGen2** в поле **Task name** (Имя задачи) и выберите **Далее**.

    ![Страница свойств](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-data-tool-properties-page.png)
3. На странице **Исходное хранилище данных** щелкните **Создать новое подключение**.

    ![Страница исходного хранилища данных](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page.png)
    
    В коллекции соединителя выберите **Azure Data Lake Storage 1-го поколения** и щелкните **Продолжить**.
    
    ![Страница настройки Azure Data Lake Storage 1-го поколения в качестве исходного хранилища данных](./media/load-azure-data-lake-storage-gen2-from-gen1/source-data-store-page-adls-gen1.png)
    
4. На **странице подключения к Azure Data Lake Storage 1-го поколения** выполните следующие действия.
   1. Выберите имя учетной записи Data Lake Storage 1-го поколения.
   2. Введите или подтвердите имя **клиента** и щелкните "Готово".
   3. Щелкните **Далее**.
   
   > [!IMPORTANT]
   > В этом руководстве вы используете управляемое удостоверение для ресурсов Azure для аутентификации в Data Lake Storage 1-го поколения. Не забудьте предоставить Управляемому удостоверению службы соответствующие разрешения в Azure Data Lake Storage 1-го поколения, выполнив [эти инструкции](connector-azure-data-lake-store.md#managed-identity).
   
   ![Выбор учетной записи Data Lake Storage 1-го поколения](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen1-account.png)
   
   4. Появится новое соединение. Щелкните **Далее**.
   
5. На странице **Choose the input file or folder** (Выбор файла или папки входных данных) перейдите в папку и файл, которые необходимо скопировать. Чтобы выбрать папку или файл, нажмите кнопку **Выбрать**.

    ![Выбор файла или папки входных данных](./media/load-azure-data-lake-storage-gen2-from-gen1/choose-input-folder.png)

6. Укажите поведение копирования, установив параметры **Copy files recursively** (Копировать файлы рекурсивно) и **Binary copy** (Двоичное копирование). Нажмите кнопку **Далее**:

    ![Укажите выходную папку.](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-binary-copy.png)
    
7. На странице **Целевое хранилище данных** выберите **Создать новое подключение**, а затем укажите  **Azure Data Lake Storage Gen2 (предварительная версия)** и щелкните **Продолжить**.

    ![Страница целевого хранилища данных](./media/load-azure-data-lake-storage-gen2-from-gen1/destination-data-storage-page.png)

8. На **странице подключения к Azure Data Lake Storage 2-го поколения** выполните следующие действия.

   1. Из раскрывающегося списка "Имя учетной записи хранения" выберите соответствующую учетную запись Data Lake Storage Gen2.
   2. Щелкните **Далее**.
   
   ![Выбор учетной записи Data Lake Storage 2-го поколения](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-account.png)

9. На странице **Choose the output file or folder** (Выбор целевого файла или папки) введите **copyfromadlsgen1** в качестве имени папки с выходными данными, а затем выберите **Далее** 

    ![Укажите выходную папку.](./media/load-azure-data-lake-storage-gen2-from-gen1/specify-adls-gen2-path.png)

10. На странице **Параметры** нажмите кнопку **Далее**, подтверждая использование параметров по умолчанию.

11. Просмотрите параметры на странице **Summary** (Сводка), а затем нажмите кнопку **Далее**.

    ![Страница "Сводка"](./media/load-azure-data-lake-storage-gen2-from-gen1/copy-summary.png)
12. Чтобы отслеживать конвейер, на **странице развертывания** выберите **Монитор**.

    ![Страница развертывания](./media/load-azure-data-lake-storage-gen2-from-gen1/deployment-page.png)
13. Обратите внимание, что слева автоматически выбирается вкладка **Мониторинг**. В столбце **Actions** (Действия) содержатся ссылки на сведения о выполнении действий или повторный запуск конвейера:

    ![Мониторинг выполнений конвейера](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-pipeline-runs.png)

14. Щелкните ссылку **View Activity Runs** (Просмотр выполнений действий) в столбце **Actions** (Действия), чтобы просмотреть выполнения действий, связанные с этим запуском конвейера. В этом конвейере определено только одно действие (действие копирования), поэтому вы увидите только одну запись. Чтобы вернуться к представлению запусков конвейера, щелкните ссылку **Конвейеры** в верхней части окна. Щелкните **Обновить**, чтобы обновить список. 

    ![Мониторинг выполнений действий](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-runs.png)

15. Чтобы отслеживать сведения о выполнении каждого действия копирования, щелкните ссылку **Сведения** (изображение очков) в разделе **Действия** в представлении мониторинга действия. Вы можете отслеживать такие сведения, как объем данных, копируемых из источника в приемник, пропускная способность данных, шаги выполнения с длительностью и используемые параметры:

    ![Мониторинг сведений о выполнении действия](./media/load-azure-data-lake-storage-gen2-from-gen1/monitor-activity-run-details.png)

16. Убедитесь, что данные скопированы в Data Lake Storage Gen2.

## <a name="best-practices"></a>Рекомендации

При копировании большого объема данных из хранилища данных, исходя из их размера, вам будет предложено следующее.

- Разделите файлы на наборы размером от 10 TБ до 30 TБ.
- Чтобы избежать регулирования количества запросов из хранилищ данных источника или приемника, не запускайте одновременно слишком много процессов копирования. Можно начать с одного процесса выполнения копирования и мониторинга пропускной способности, а затем по мере необходимости добавлять еще.

## <a name="next-steps"></a>Дополнительная информация

* [Действие копирования в фабрике данных Azure](copy-activity-overview.md)
* [Copy data to or from Azure Data Lake Storage Gen2 Preview using Azure Data Factory (Preview)](connector-azure-data-lake-storage.md) (Копирование данных в Azure Data Lake Storage Gen2 (предварительная версия) или из него с помощью фабрики данных Azure)