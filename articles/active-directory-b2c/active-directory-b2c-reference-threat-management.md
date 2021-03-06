---
title: Управление угрозами в Azure Active Directory B2C | Документация Майкрософт
description: Дополнительные сведения о методиках обнаружения и устранения рисков атак типа "отказ в обслуживании" и атак для взлома пароля в Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 03/27/2016
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 1801fe9695aa15850d600300b957df2c7d7cd9ef
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/18/2018
ms.locfileid: "42143586"
---
# <a name="azure-active-directory-b2c-threat-management"></a>Предотвращение угроз в Azure Active Directory B2C

Предотвращение угроз включает в себя планирование защиты от атак, нацеленных на систему и сети. Атаки типа "отказ в обслуживании" могут сделать ресурсы недоступными для предполагаемых пользователей. Атаки для взлома пароля приводят к несанкционированному доступу к ресурсам. Служба Azure Active Directory B2C (Azure AD B2C) оснащена несколькими встроенными средствами защиты данных от таких угроз.

## <a name="denial-of-service-attacks"></a>Атаки типа "отказ в обслуживании"

Для защиты базовых ресурсов от атак типа "отказ в обслуживании" Azure AD B2C использует методики обнаружения и уменьшения негативных последствий, такие как файлы cookie SYN, ограничение скорости и количества подключений.

## <a name="password-attacks"></a>Атаки для взлома пароля

Служба Azure AD B2C также имеет методики противодействия атакам для взлома пароля. Эти методики действуют при атаках методом подбора пароля и при словарных атаках на пароль. Пароли, задаваемые пользователями, должны быть достаточно сложными. Используя различные сигналы, Azure AD B2C анализирует целостность запросов. Azure AD B2C предназначен для того, чтобы интеллектуально отличить предполагаемых пользователей от злоумышленников и ботнетов. Служба Azure AD B2C применяет сложные стратегии для блокировки учетных записей, анализируя вводимые пароли и выявляя потенциальные атаки.

Дополнительные сведения см. в [центре управления безопасностью Майкрософт](https://www.microsoft.com/trustcenter/default.aspx).
