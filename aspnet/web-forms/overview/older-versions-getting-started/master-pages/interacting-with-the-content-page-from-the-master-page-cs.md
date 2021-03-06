---
uid: web-forms/overview/older-versions-getting-started/master-pages/interacting-with-the-content-page-from-the-master-page-cs
title: Взаимодействие со страницей содержимого на главной странице (C#) | Документация Майкрософт
author: rick-anderson
description: Проверяет как вызывать методы, задайте свойства, т. д., страницы содержимого из кода на главной странице.
ms.author: riande
ms.date: 07/11/2008
ms.assetid: 3282df5e-516c-4972-8666-313828b90fb5
msc.legacyurl: /web-forms/overview/older-versions-getting-started/master-pages/interacting-with-the-content-page-from-the-master-page-cs
msc.type: authoredcontent
ms.openlocfilehash: 815752ee70eb761d7f9da24c9eada9d4c0c833a7
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/16/2018
ms.locfileid: "41829939"
---
<a name="interacting-with-the-content-page-from-the-master-page-c"></a>Взаимодействие со страницей содержимого на главной странице (C#)
====================
по [Скотт Митчелл](https://twitter.com/ScottOnWriting)

[Скачать код](http://download.microsoft.com/download/1/8/4/184e24fa-fcc8-47fa-ac99-4b6a52d41e97/ASPNET_MasterPages_Tutorial_07_CS.zip) или [скачать PDF](http://download.microsoft.com/download/e/b/4/eb4abb10-c416-4ba4-9899-32577715b1bd/ASPNET_MasterPages_Tutorial_07_CS.pdf)

> Проверяет как вызывать методы, задайте свойства, т. д., страницы содержимого из кода на главной странице.


## <a name="introduction"></a>Вступление

Предыдущем учебном курсе увидели, как для программного взаимодействия с главной страницей по страницы содержимого. Вспомним, что мы изменили главной страницы для включения элемента управления GridView, в списке пять наиболее недавно добавлена продуктов. Затем мы создали страницу содержимого, с помощью которой пользователь может добавить нового продукта. При добавлении нового продукта, страницы содержимого необходимо указать главной страницы, чтобы обновить его GridView, таким образом, чтобы он включает только что добавленные продукта. Эта функция осуществлялось путем добавления открытого метода на главную страницу, что обновленные данные, привязанные к GridView и последующим вызовом этого метода на странице содержимого.

Наиболее распространенная форма содержимого, а также взаимодействие с главной страницы исходит от содержимого страницы. Тем не менее это возможно для главной страницы для Русом содержимого текущей страницы в действие, и такая функция может потребоваться, если главная страница содержит элементы пользовательского интерфейса, которые позволяют пользователям изменять данные, которые также отображаются на странице содержимого. Рассмотрим страницу содержимого, что контролировать отображение сведений о продуктах в элементе управления GridView и главной страницы, которая включает кнопки элемента управления, при щелчке, удваивает цены всех продуктов. Как в примере в предыдущем учебном курсе GridView необходимо обновить после двойной цене, кнопки, который будет новым ценам, но в этом случае это главную страницу, необходимо Русом содержимого страницы в действие.

В этом руководстве рассматриваются способы получения вызова функций, определенных на странице содержимого главной страницы.

### <a name="instigating-programmatic-interaction-via-an-event-and-event-handlers"></a>Instigating программного взаимодействия с помощью события и обработчики событий

Вызов функции страницы содержимого на главной странице сложнее, чем в обратном направлении. Так как страницы содержимого имеет одну главную страницу, при instigating программного взаимодействия на странице содержимого мы знаем, что открытые методы и свойства, в нашем распоряжении. Главная страница, тем не менее, может иметь много различных страниц содержимого, каждый из которых свой собственный набор свойств и методов. Как это сделать затем написать код на главной странице, для выполнения некоторых операций в содержимое ее страницы, если не знаете, какие страницы содержимого будет вызываться только во время выполнения?

Рассмотрите возможность элемента управления ASP.NET, такого как Button. Элемент управления Button может отображаться на любом количестве страниц ASP.NET и необходим механизм, по которому он может выдать страницы, он был выполнен щелчок. Это достигается с помощью *события*. В частности, элемент управления Button вызывает его `Click` событие при щелчке; страницы ASP.NET, содержащий кнопку, можно при необходимости ответить этого уведомления через *обработчик событий*.

Этот же шаблон можно использовать для функции триггера главной страницы на его страницах содержимого:

1. Добавьте событие на главную страницу.
2. Событие каждый раз, когда главной страницы приходится обмениваться данными со содержимое ее страницы. Например если главной страницы должен оповещать содержимое ее страницы, что пользователь удвоилось цены, свое событие должно порождаться сразу после удвоения цены.
3. Создайте обработчик событий на этих страницах содержимого, которые необходимо выполнить определенное действие.

Далее в этом руководстве реализует пример, описанные во введении; а именно страницы содержимого, которая перечисляет продукты в базе данных и главной страницы, которая включает кнопки управления удваивать цены.

## <a name="step-1-displaying-products-in-a-content-page"></a>Шаг 1: Отображение продуктов в страницу содержимого

Наши первым делом — создать страницу содержимого, которая перечисляет продукты из базы данных "Борей". (Мы добавили базы данных Northwind в проект в предыдущем учебном курсе, [ *взаимодействие с главной страницей на странице содержимого*](interacting-with-the-master-page-from-the-content-page-cs.md).) Начните с добавления новой страницы ASP.NET `~/Admin` папку с именем `Products.aspx`, не забыв привяжите его к `Site.master` главной страницы. Рис. 1 показан обозреватель решений после добавления этой страницы на веб-сайт.


[![Добавьте новую страницу ASP.NET в папку администрирования](interacting-with-the-content-page-from-the-master-page-cs/_static/image2.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image1.png)

**Рис 01**: добавьте новую страницу ASP.NET к `Admin` папку ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image3.png))


Помните, что в [ *указание названия, метатегов и других заголовков HTML на главной странице* ](specifying-the-title-meta-tags-and-other-html-headers-in-the-master-page-cs.md) руководстве мы создали пользовательскую страницу базовый класс с именем `BasePage` , создающий заголовок страницы, если это не явно задайте. Перейдите к `Products.aspx` фонового кода страницы класса, который будет наследовать `BasePage` (а не из `System.Web.UI.Page`).

Наконец, обновите `Web.sitemap` файл, чтобы включить запись для этого занятия. Добавьте следующую разметку под `<siteMapNode>` для содержимого к занятию взаимодействие страниц Master:


[!code-xml[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample1.xml)]

Добавление этого `<siteMapNode>` отражается на занятиях списке (см. рис. 5).

Вернитесь к `Products.aspx`. В элементе управления содержимым для `MainContent`, добавление элемента управления GridView и назовите его `ProductsGrid`. Привязки GridView к нового элемента управления SqlDataSource с именем `ProductsDataSource`.


[![Привязки GridView к нового элемента управления SqlDataSource](interacting-with-the-content-page-from-the-master-page-cs/_static/image5.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image4.png)

**Рис. 02**: привязки GridView к нового элемента управления SqlDataSource ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image6.png))


Настройте мастер, чтобы он использует базы данных Northwind. Если вы работали в предыдущем учебнике, то необходимо иметь строку подключения с именем `NorthwindConnectionString` в `Web.config`. Выберите из раскрывающегося списка, эта строка подключения, как показано на рис. 3.


[![Настройка элемента управления SqlDataSource для использования базы данных "Борей"](interacting-with-the-content-page-from-the-master-page-cs/_static/image8.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image7.png)

**Рис 03**: Настройка элемента управления SqlDataSource для использования базы данных Northwind ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image9.png))


Затем укажите элемента управления источником данных `SELECT` инструкции, выбрав таблицы Products из раскрывающегося списка и возвращение `ProductName` и `UnitPrice` столбцов (см. рис. 4). Нажмите кнопку Далее, а затем завершить работу мастера настройки источника данных.


[![Возвращать ProductName и UnitPrice поля из таблицы Products](interacting-with-the-content-page-from-the-master-page-cs/_static/image11.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image10.png)

**Рис. 04**: возвращать `ProductName` и `UnitPrice` поля из `Products` таблицы ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image12.png))


Это все, вот! После завершения работы мастера Visual Studio добавляет два поля BoundField, кроме GridView, чтобы отобразить два поля, возвращенные объектом элемента управления SqlDataSource. Ниже приведен разметка GridView и SqlDataSource элементов управления. Рис. 5 показаны результаты выполненной работы в браузере.


[!code-aspx[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample2.aspx)]


[![И его цену каждого продукта указан в GridView](interacting-with-the-content-page-from-the-master-page-cs/_static/image14.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image13.png)

**05 рис**: и его цену каждого продукта указан в GridView ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image15.png))


> [!NOTE]
> Вы можете очистить внешний вид элемента управления GridView. Некоторые предложения, включают форматирование отображаемое значение UnitPrice как денежной единицы и с помощью фоновых цветов и шрифтов, чтобы улучшить внешний вид сетки. Дополнительные сведения о отображения и форматирования данных в ASP.NET, см. Мой [работа с серии руководств данных](../../data-access/index.md).


## <a name="step-2-adding-a-double-prices-button-to-the-master-page"></a>Шаг 2: Добавление кнопки Double цены на главную страницу

Следующим действием будет добавление кнопки веб-элемент управления на главной странице, при нажатии будет double цены всех продуктов в базе данных. Откройте `Site.master` главную страницу и перетащите кнопку из панели элементов в конструктор, поместив его под `RecentProductsDataSource` управления SqlDataSource, мы добавили в предыдущем учебном курсе. Задайте `ID` свойства `DoublePrice` и его `Text` свойства «Double цены продукта».

Добавьте элемент управления SqlDataSource на главную страницу, назовите его `DoublePricesDataSource`. Этот SqlDataSource будет использоваться для выполнения `UPDATE` инструкции выполнить удвоение цены. В частности, нам нужно установить его `ConnectionString` и `UpdateCommand` свойства для соответствующей строкой подключения и `UPDATE` инструкции. Затем необходимо вызвать этот элемент управления SqlDataSource `Update` метод при `DoublePrice` кнопки. Чтобы задать `ConnectionString` и `UpdateCommand` свойства, выберите элемент управления SqlDataSource, а затем перейдите в окно свойств. `ConnectionString` Эти строки подключения, уже хранится в списках свойств `Web.config` в раскрывающемся списке; выберите `NorthwindConnectionString` как показано на рис. 6.


[![Настройка элемента управления SqlDataSource для использования NorthwindConnectionString](interacting-with-the-content-page-from-the-master-page-cs/_static/image17.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image16.png)

**Рис 06**: Настройка элемента управления SqlDataSource для использования `NorthwindConnectionString` ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image18.png))


Чтобы задать `UpdateCommand` свойство, найдите параметр UpdateQuery в окне «Свойства». Это свойство, при выборе отображает кнопку с многоточием; Нажмите эту кнопку, чтобы отобразить редактор команд и параметров диалоговое окно, показанное на рис. 7. Введите следующую команду `UPDATE` инструкции в текстовое поле диалогового окна:


[!code-sql[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample3.sql)]

Эта инструкция, при выполнении возрастет `UnitPrice` значение для каждой записи в `Products` таблицы.


[![Установите свойство UpdateCommand на SqlDataSource](interacting-with-the-content-page-from-the-master-page-cs/_static/image20.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image19.png)

**Рис 07**: задать SqlDataSource `UpdateCommand` свойство ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image21.png))


После настройки этих свойств, декларативная разметка вашей кнопки и элемента управления SqlDataSource элементов управления должен выглядеть следующим образом:


[!code-aspx[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample4.aspx)]

Все, что остается только вызвать его `Update` метод при `DoublePrice` кнопки. Создание `Click` обработчик событий для `DoublePrice` кнопку и добавьте следующий код:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample5.cs)]

Чтобы протестировать эту функцию, посетите `~/Admin/Products.aspx` страницы, мы создали в шаге 1 и нажмите кнопку «Double цены продукта». Нажмите кнопку вызывает обратную передачу и выполняет `DoublePrice` кнопки `Click` обработчик событий, удваивать цены всех продуктов. Затем повторно отображении страницы и разметку возвращается и снова отображается в браузере. GridView на странице содержимого, однако список перед «Double цены продукта» была нажата кнопка же цены. Это, так как данные начальной загрузки в GridView состояние, хранятся в состоянии просмотра, чтобы повторно не загружаются при обратных передачах, если не указано другое. Если обратиться на другую страницу и затем вернитесь к `~/Admin/Products.aspx` страницы, вы увидите обновленные цены.

## <a name="step-3-raising-an-event-when-the-prices-are-doubled"></a>Шаг 3: Создание событий при цены являются вдвое

Так как GridView в `~/Admin/Products.aspx` странице не отражает сразу же цена удвоение производительности, пользователь по понятной причине может показаться, что они были не нажать кнопку «Double цены продукта», или что она не работала. Они могут попробуйте нажать кнопку еще несколько раз, снова и снова удваивать цены. Чтобы исправить это, необходимо иметь в сетке в содержимом новых цен отображения страницы, сразу же после их два раза подряд.

Как уже говорилось ранее в этом учебнике, нам нужно вызвать событие на главной странице каждый раз, когда пользователь нажимает кнопку `DoublePrice` кнопки. Событие — возможность для одного класса (издатель событий) уведомления другой набор других классов (подписчики на события), что произошло нечто интересное. В этом примере главная страница является издателем событий; те содержимого страниц, которые применяются при `DoublePrice` кнопки являются подписчиками.

Класс подписывается на событие, создавая *обработчик событий*, который является методом, который выполняется в ответ на события. Издатель определяет события, он вызывает, определив *делегат события*. Делегат события указывает, какие входные параметры, необходимо принять обработчик событий. В .NET Framework делегатов событий не возвращает никакого значения и принимать два входных параметра:

- `Object`, Который идентифицирует источник событий, и
- Класс, производный от `System.EventArgs`

Второй параметр, передаваемый обработчику событий может включать дополнительные сведения о событии. Хотя базовый `EventArgs` класс не передать информацию, .NET Framework включает ряд классов, расширяющих `EventArgs` и содержит дополнительные свойства. Например `CommandEventArgs` экземпляр передается обработчикам событий, которые отвечают на `Command` событий и включает в себя два свойства информационное: `CommandArgument` и `CommandName`.

> [!NOTE]
> Дополнительные сведения о создании вызов и обработку событий, см. в разделе [события и делегаты](https://msdn.microsoft.com/library/17sde2xt.aspx) и [делегатов событий в простой английской](http://www.codeproject.com/KB/cs/eventdelegates.aspx).


Для определения события используйте следующий синтаксис:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample6.cs)]

Поскольку нам требуется только для оповещения на странице содержимого, когда пользователь нажимает `DoublePrice` кнопку и не обязательно передавать другие дополнительные сведения, можно использовать делегат события `EventHandler`, который определяет обработчик событий, который принимает как второй параметр объекта типа `System.EventArgs`. Чтобы создать событие на главной странице, добавьте следующую строку кода для главной страницы вспомогательного класса:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample7.cs)]

Приведенный выше код добавляет открытое событие на главную страницу с именем `PricesDoubled`. Теперь нам нужно вызывать это событие после удвоения цены. Для вызова события используйте следующий синтаксис:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample8.cs)]

Где *отправителя* и *eventArgs* — это значения, которые вы хотите передать в обработчик событий подписчика.

Обновление `DoublePrice` `Click` обработчик событий следующим кодом:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample9.cs)]

Как и раньше `Click` запускает обработчик событий с помощью вызова `DoublePricesDataSource` элемента управления SqlDataSource `Update` метод выполнить удвоение цены всех продуктов. После того, что существуют две функции обработчика событий. Во-первых, `RecentProducts` обновления данных к GridView. Этот GridView в предыдущем учебном курсе был добавлен на главную страницу и отображает пять наиболее недавно добавленные продукты. Нам нужно обновить эту сетку так, чтобы отобразить только что вдвое цены для этих пяти продуктов. После этого `PricesDoubled` события. Ссылка на главную страницу, сам (`this`) отправляется в обработчик событий как источника событий и пустой `EventArgs` объект отправляется в виде аргументов события.

## <a name="step-4-handling-the-event-in-the-content-page"></a>Шаг 4: Обработка событий на странице содержимого

На этом этапе главной страницы вызывает его `PricesDoubled` событие каждый раз, когда `DoublePrice` нажатии элемента управления кнопки. Тем не менее это только половина дела — мы по-прежнему нужно обработать событие в подписчике. Это состоит из двух этапов: Создание обработчика событий и добавлять код события, чтобы при возникновении события выполняется обработчик событий.

Сначала создайте обработчик событий с именем `Master_PricesDoubled`. Из-за, как мы определили `PricesDoubled` событий на главной странице два входных параметра обработчик событий должны иметь типы `Object` и `EventArgs`, соответственно. При вызове обработчика событий `ProductsGrid` GridView `DataBind` метод повторную привязку данных к сетке.


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample10.cs)]

Полный код для обработчика событий, но еще не знаем wire главной страницы `PricesDoubled` событий в этот обработчик событий. Подписчик связывает события в обработчик событий используется следующий синтаксис:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample11.cs)]

*издатель* является ссылкой на объект, который предоставляет события *eventName*, и *имя_метода* имя обработчика событий, определенных в подписчике, который имеет сигнатуру, соответствующий *eventDelegate*. Другими словами, если делегат события является `EventHandler`, затем *имя_метода* должно быть имя метода в подписчике, не возвращать значение и принимает два входных параметра типов `Object` и `EventArgs`, соответственно.

Этот код события должна быть выполнена на первом посещении страницы и последующих обратных передачах и должно выполняться в момент, предшествующий, когда может быть создано событие жизненного цикла страницы. Самое время добавить код события находится на этапе PreInit, которое происходит очень рано в жизненном цикле страницы.

Откройте `~/Admin/Products.aspx` и создайте `Page_PreInit` обработчик событий:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample12.cs)]

Чтобы выполнить этот код полностью программную ссылку на главную страницу, на странице содержимого. Как отмечалось в предыдущем учебном курсе, существует два способа это сделать:

- Путем приведения слабо типизированной `Page.Master` свойство в тип соответствующей главной страницы, или
- Добавив `@MasterType` директиву `.aspx` страницы и затем с помощью строго типизированного `Master` свойство.

Давайте используем последний подход. Добавьте следующий `@MasterType` директиву в начало декларативная разметка страницы:


[!code-aspx[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample13.aspx)]

Затем добавьте следующий код привязки событий в `Page_PreInit` обработчик событий:


[!code-csharp[Main](interacting-with-the-content-page-from-the-master-page-cs/samples/sample14.cs)]

Этот код в месте, обновляется GridView на странице содержимого всякий раз, когда `DoublePrice` кнопки.

Цифры 8 и 9, иллюстрируют это поведение. Рис. 8 показан при первом посещении страницы. Обратите внимание, что цена значения в обоих `RecentProducts` GridView (в левом столбце главной страницы) и `ProductsGrid` GridView (на странице содержимого). Рис. 9 показана же экране сразу же после `DoublePrice` был выполнен щелчок кнопкой. Как вы видите, новые цены мгновенно отражаются в обоих элементов управления GridView.


[![Начальная цена значения](interacting-with-the-content-page-from-the-master-page-cs/_static/image23.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image22.png)

**Рис 08**: начальные значения цены ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image24.png))


[![Just-Doubled цены отображаются в элементов управления GridView](interacting-with-the-content-page-from-the-master-page-cs/_static/image26.png)](interacting-with-the-content-page-from-the-master-page-cs/_static/image25.png)

**Рис 09**: The Just-Doubled цены отображаются в элементов управления GridView ([Просмотр полноразмерного изображения](interacting-with-the-content-page-from-the-master-page-cs/_static/image27.png))


## <a name="summary"></a>Сводка

В идеале главной страницы и его страниц полностью отделены друг от друга и уровень взаимодействия не требуется. Тем не менее, при наличии главной страницы или страницы содержимого, которое отображает данные, которые могут изменяться из главной страницы или страницы содержимого, то может потребоваться иметь главную страницу оповещений страница содержимого (или наоборот a) при изменении данных, чтобы можно было обновить отображение. В предыдущем учебном курсе мы рассмотрели, содержимого страницы программного взаимодействия с главной страницей по; в этом учебнике мы рассмотрели способы у главной страницы, инициирующего взаимодействие.

Хотя программного взаимодействия между содержимым и главной страницы может быть создано из содержимого или главной страницы, шаблон взаимодействия, используемый зависит от происхождения. Различия, из-за того, что страница содержимого есть одна главная страница, но главная страница может иметь много различных страниц содержимого. Вместо того, Главная страница напрямую взаимодействовать со страницей содержимого, лучше иметь главной страницы, которые вызывают событие, чтобы сообщить, что некоторые было выполнено действие. Эти страницы содержимого, которые интересуют действия можно создавать обработчики событий.

Счастливого вам программирования!

### <a name="further-reading"></a>Дополнительные сведения

Дополнительные сведения по темам, обсуждавшимся в этом руководстве см. в следующих ресурсах:

- [Просмотр и обновление данных в ASP.NET](http://aspnet.4guysfromrolla.com/articles/011106-1.aspx)
- [События и делегаты](https://msdn.microsoft.com/library/17sde2xt.aspx)
- [Передача сведений об между содержимым и главные страницы](http://aspnet.4guysfromrolla.com/articles/013107-1.aspx)
- [Работа с данными в руководствах по ASP.NET](../../data-access/index.md)

### <a name="about-the-author"></a>Об авторе

[Скотт Митчелл](http://www.4guysfromrolla.com/ScottMitchell.shtml), автор из нескольких книг по ASP/ASP.NET и основатель веб-узла 4GuysFromRolla.com, работает с веб-технологиями Microsoft с 1998 года. Скотт — независимый консультант, преподаватель и автор. Его последняя книга — [ *Sams Teach самостоятельно ASP.NET 3.5 за 24 часа*](https://www.amazon.com/exec/obidos/ASIN/0672329972/4guysfromrollaco). Скоттом можно связаться по адресу [ mitchell@4GuysFromRolla.com ](mailto:mitchell@4GuysFromRolla.com) или через его блог по адресу [ http://ScottOnWriting.NET ](http://scottonwriting.net/).

### <a name="special-thanks-to"></a>Особая благодарность

В этой серии руководств пособий рецензировалась многими компетентными редакторами. Основной рецензент этого учебного был Банерджи Suchi. Хотите поработать с моих последующих статей для MSDN? Если Да, напишите мне [mitchell@4GuysFromRolla.com](mailto:mitchell@4GuysFromRolla.com)

> [!div class="step-by-step"]
> [Назад](interacting-with-the-master-page-from-the-content-page-cs.md)
> [Вперед](master-pages-and-asp-net-ajax-cs.md)
