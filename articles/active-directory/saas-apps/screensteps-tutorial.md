---
title: Руководство по интеграции Azure Active Directory с ScreenSteps | Документация Майкрософт
description: Узнайте, как настроить единый вход Azure Active Directory в приложении ScreenSteps.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: joflore
ms.assetid: 4563fe94-a88f-4895-a07f-79df44889cf9
ms.service: active-directory
ms.component: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/14/2017
ms.author: jeedes
ms.openlocfilehash: 105ec895635a882d562de48203222702a2c6bfed
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/02/2018
ms.locfileid: "39421752"
---
# <a name="tutorial-azure-active-directory-integration-with-screensteps"></a>Учебник. Интеграция Azure Active Directory с ScreenSteps

В этом учебнике описано, как интегрировать ScreenSteps с Azure Active Directory (Azure AD).

Интеграция Azure AD с приложением ScreenSteps обеспечивает следующие преимущества.

- C помощью Azure AD вы можете контролировать доступ к ScreenSteps.
- Вы можете включить автоматический вход пользователей в ScreenSteps (единый вход) с использованием учетной записи Azure AD.
- Вы можете управлять учетными записями централизованно — на портале Azure.

Подробнее узнать об интеграции приложений SaaS с Azure AD можно в разделе [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory](../manage-apps/what-is-single-sign-on.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить интеграцию Azure AD с ScreenSteps, вам потребуется:

- подписка Azure AD;
- подписка ScreenSteps с поддержкой единого входа.

> [!NOTE]
> Мы не рекомендуем использовать рабочую среду для проверки действий в этом учебнике.

При проверке действий в этом учебнике соблюдайте следующие рекомендации:

- Не используйте рабочую среду без необходимости.
- Если у вас нет пробной среды Azure AD, вы можете [получить пробную версию на один месяц](https://azure.microsoft.com/pricing/free-trial/).

## <a name="scenario-description"></a>Описание сценария
В рамках этого руководства проводится проверка единого входа Azure AD в тестовой среде. Сценарий, описанный в этом учебнике, состоит из двух стандартных блоков.

1. Добавление ScreenSteps из коллекции
1. настройка и проверка единого входа в Azure AD.

## <a name="adding-screensteps-from-the-gallery"></a>Добавление ScreenSteps из коллекции
Чтобы настроить интеграцию ScreenSteps с Azure AD, необходимо добавить ScreenSteps из коллекции в список управляемых приложений SaaS.

**Чтобы добавить ScreenSteps из коллекции, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева щелкните значок **Azure Active Directory**. 

    ![Кнопка "Azure Active Directory"][1]

1. Перейдите к разделу **Корпоративные приложения**. Затем выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"][2]
    
1. Чтобы добавить новое приложение, в верхней части диалогового окна нажмите кнопку **Создать приложение**.

    ![Кнопка "Новое приложение"][3]

1. В поле поиска введите **ScreenSteps**, выберите **ScreenSteps** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![ScreenSteps в списке результатов](./media/screensteps-tutorial/tutorial_screensteps_addfromgallery.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Настройка и проверка единого входа в Azure AD

В этом разделе описана настройка и проверка единого входа Azure AD в ScreenSteps с использованием тестового пользователя Britta Simon.

Для работы единого входа в Azure AD необходимо знать, какой пользователь в ScreenSteps соответствует пользователю в Azure AD. Иными словами, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в ScreenSteps.

Чтобы установить эту связь, назначьте **имя пользователя** в Azure AD в качестве значения **имени пользователя** в ScreenSteps.

Чтобы настроить и проверить единый вход Azure AD в ScreenSteps, вам потребуется выполнить действия в следующих стандартных блоках.

1. **[Настройка единого входа Azure AD](#configure-azure-ad-single-sign-on)** необходима, чтобы пользователи могли использовать эту функцию.
1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD от имени пользователя Britta Simon.
1. **[Создание тестового пользователя ScreenSteps](#create-a-screensteps-test-user)** требуется для того, чтобы в ScreenSteps существовал пользователь Britta Simon, связанный с одноименным пользователем в Azure AD.
1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить Britta Simon использовать единый вход Azure AD.
1. **[Проверка единого входа](#test-single-sign-on)** необходима, чтобы убедиться в корректной работе конфигурации.

### <a name="configure-azure-ad-single-sign-on"></a>Настройка единого входа Azure AD

В этом разделе описано, как включить единый вход Azure AD на портале Azure и настроить его в приложении ScreenSteps.

**Чтобы настроить единый вход Azure AD в ScreenSteps, выполните следующие действия.**

1. На портале Azure на странице интеграции с приложением **ScreenSteps** щелкните **Единый вход**.

    ![Ссылка "Настройка единого входа"][4]

1. В диалоговом окне **Единый вход** в разделе **Режим** выберите **Вход на основе SAML**, чтобы включить функцию единого входа.
 
    ![Диалоговое окно "Единый вход"](./media/screensteps-tutorial/tutorial_screensteps_samlbase.png)

1. В разделе **Домены и URL-адреса приложения ScreenSteps** сделайте следующее.

    ![Сведения о домене и URL-адресах единого входа приложения ScreenSteps](./media/screensteps-tutorial/tutorial_screensteps_url.png)

    В текстовом поле **URL-адрес для входа** введите URL-адрес в следующем формате: `https://<tenantname>.ScreenSteps.com`

    > [!NOTE] 
    > Это значение приведено для справки. Замените его на фактический URL-адрес входа, как описано далее в этом учебнике. 

1. В разделе **Сертификат подписи SAML** щелкните **Сертификат (Base64)**, а затем сохраните файл сертификата на компьютере.

    ![Ссылка для скачивания сертификата](./media/screensteps-tutorial/tutorial_screensteps_certificate.png) 

1. Нажмите кнопку **Сохранить** .

    ![Кнопка "Сохранить" в окне настройки единого входа](./media/screensteps-tutorial/tutorial_general_400.png)

1. В разделе **Конфигурация ScreenSteps** щелкните **Настроить ScreenSteps**, чтобы открыть окно **Настройка единого входа**. Скопируйте **URL-адрес выхода и URL-адрес службы единого входа SAML** из раздела **Краткий справочник**.

    ![Конфигурация ScreenSteps](./media/screensteps-tutorial/tutorial_screensteps_configure.png) 

1. В другом окне веб-браузера войдите на сайт ScreenSteps своей компании в качестве администратора.

1. Щелкните **Параметры учетной записи**.

    ![Управление учетными записями](./media/screensteps-tutorial/ic778523.png "Управление учетными записями")

1. Щелкните **Single Sign-on**(Единый вход).

    ![Удаленная аутентификация](./media/screensteps-tutorial/ic778524.png "Удаленная аутентификация")

1. Щелкните **Create Single Sign-on Endpoint** (Создать конечную точку единого входа).

    ![Удаленная аутентификация](./media/screensteps-tutorial/ic778525.png "Удаленная аутентификация")

1. В разделе **Create Single Sign-on Endpoint** (Создать конечную точку единого входа) сделайте следующее:

    ![Создание конечной точки аутентификации](./media/screensteps-tutorial/ic778526.png "Создание конечной точки аутентификации")
    
    a. В текстовом поле **Название** введите название.
    
    b. Из списка **Mode** (Режим) выберите **SAML**.
    
    c. Нажмите кнопку **Создать**.

1. **Измените** новую конечную точку.

    ![Изменение конечной точки](./media/screensteps-tutorial/ic778528.png "Edit endpoint")

1. В разделе **Create Single Sign-on Endpoint** (Изменить конечную точку единого входа) сделайте следующее:

    ![Конечная точка удаленной аутентификации](./media/screensteps-tutorial/ic778527.png "Конечная точка удаленной аутентификации")

    a. Щелкните **Upload new SAML Certificate file** (Отправить новый файл сертификата SAML), а затем отправьте сертификат, скачанный с портала Azure.
    
    b. Вставьте **URL-адрес службы единого входа SAML**, скопированный на портале Azure, в текстовое поле **URL-адрес удаленного входа**.
    
    c. Вставьте **URL-адрес входа**, скопированный на портале Azure, в текстовое поле **URL-адрес выхода**.
    
    d. Выберите **группу** для назначения пользователей при их подготовке.
    
    д. Нажмите кнопку **Обновить**.

    Е. Скопируйте **URL-адрес потребителя SAML** в буфер обмена и вставьте его в текстовое поле **URL-адрес входа** в разделе **Домены и URL-адреса приложения ScreenSteps**.
    
    ж. Вернитесь в раздел **Edit Single Sign-on Endpoint** (Изменить конечную точку единого входа).
    
    h. Нажмите кнопку **Make default for account** (Сделать значением по умолчанию для учетной записи), чтобы использовать эту конечную точку для всех пользователей, осуществляющих вход в ScreenSteps. Кроме того, можно нажать кнопку **Add to Site** (Добавить на сайт), чтобы использовать эту конечную точку для определенных сайтов в **ScreenSteps**.

> [!TIP]
> Краткую версию этих инструкций теперь можно также прочитать на [портале Azure](https://portal.azure.com) во время настройки приложения.  После добавления этого приложения из раздела **Active Directory > Корпоративные приложения** просто выберите вкладку **Единый вход** и откройте встроенную документацию через раздел **Настройка** в нижней части страницы. Дополнительные сведения о встроенной документации см. в разделе [Встроенная документация Azure AD]( https://go.microsoft.com/fwlink/?linkid=845985).
> 

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

Цель этого раздела — создать на портале Azure тестового пользователя с именем Britta Simon.

   ![Создание тестового пользователя Azure AD][100]

**Чтобы создать тестового пользователя в Azure AD, выполните следующие действия:**

1. На портале Azure в области слева нажмите кнопку **Azure Active Directory**.

    ![Кнопка "Azure Active Directory"](./media/screensteps-tutorial/create_aaduser_01.png)

1. Чтобы открыть список пользователей, перейдите в раздел **Пользователи и группы** и щелкните **Все пользователи**.

    ![Ссылки "Пользователи и группы" и "Все пользователи"](./media/screensteps-tutorial/create_aaduser_02.png)

1. Чтобы открыть диалоговое окно **Пользователь**, в верхней части диалогового окна **Все пользователи** щелкните **Добавить**.

    ![Кнопка "Добавить"](./media/screensteps-tutorial/create_aaduser_03.png)

1. В диалоговом окне **Пользователь** сделайте следующее.

    ![Диалоговое окно "Пользователь"](./media/screensteps-tutorial/create_aaduser_04.png)

    a. В поле **Имя** введите **BrittaSimon**.

    Б. В поле **Имя пользователя** введите адрес электронной почты для пользователя Britta Simon.

    c. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.

    d. Нажмите кнопку **Создать**.
 
### <a name="create-a-screensteps-test-user"></a>Создание тестового пользователя ScreenSteps

В этом разделе описано, как создать пользователя Britta Simon в ScreenSteps. Обратитесь в [службу поддержки ScreenSteps](https://www.screensteps.com/contact), чтобы добавить пользователей на платформу ScreenSteps. Перед использованием единого входа необходимо создать и активировать пользователей. 

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как разрешить пользователю Britta Simon использовать единый вход Azure, предоставив этому пользователю доступ к ScreenSteps.

![Назначение роли пользователя][200] 

**Чтобы назначить пользователя Britta Simon приложению ScreenSteps, выполните следующие действия.**

1. На портале Azure откройте представление приложений, перейдите к представлению каталога, а затем выберите **Корпоративные приложения** и щелкните **Все приложения**.

    ![Назначение пользователя][201] 

1. В списке приложений выберите **ScreenSteps**.

    ![Ссылка на ScreenSteps в списке "Приложения"](./media/screensteps-tutorial/tutorial_screensteps_app.png)  

1. В меню слева выберите **Пользователи и группы**.

    ![Ссылка "Пользователи и группы"][202]

1. Нажмите кнопку **Добавить**. Затем в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.

    ![Область "Добавление назначения"][203]

1. В диалоговом окне **Пользователи и группы** в списке пользователей выберите **Britta Simon**.

1. В диалоговом окне **Пользователи и группы** нажмите кнопку **Выбрать**.

1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.
    
### <a name="test-single-sign-on"></a>Проверка единого входа

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью панели доступа.

Щелкнув элемент ScreenSteps на панели доступа, вы автоматически войдете в приложение ScreenSteps.
Дополнительные сведения о панели доступа см. в статье [Общие сведения о панели доступа](../user-help/active-directory-saas-access-panel-introduction.md). 

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Список учебников по интеграции приложений SaaS с Azure Active Directory](tutorial-list.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

<!--Image references-->

[1]: ./media/screensteps-tutorial/tutorial_general_01.png
[2]: ./media/screensteps-tutorial/tutorial_general_02.png
[3]: ./media/screensteps-tutorial/tutorial_general_03.png
[4]: ./media/screensteps-tutorial/tutorial_general_04.png

[100]: ./media/screensteps-tutorial/tutorial_general_100.png

[200]: ./media/screensteps-tutorial/tutorial_general_200.png
[201]: ./media/screensteps-tutorial/tutorial_general_201.png
[202]: ./media/screensteps-tutorial/tutorial_general_202.png
[203]: ./media/screensteps-tutorial/tutorial_general_203.png

