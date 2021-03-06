---
uid: web-forms/overview/older-versions-getting-started/deploying-web-site-projects/processing-unhandled-exceptions-vb
title: Обработка необработанных исключений (Visual Basic) | Документация Майкрософт
author: rick-anderson
description: При возникновении ошибки среды выполнения веб-приложения в рабочей среде важно для уведомления разработчик и записать ошибку в журнал, чтобы она может обнаружить в a la...
ms.author: riande
ms.date: 06/09/2009
ms.assetid: 051296f0-9519-4e78-835c-d868da13b0a0
msc.legacyurl: /web-forms/overview/older-versions-getting-started/deploying-web-site-projects/processing-unhandled-exceptions-vb
msc.type: authoredcontent
ms.openlocfilehash: 29ea7f376f61c242ab93cfb71e1a7b435c575482
ms.sourcegitcommit: 45ac74e400f9f2b7dbded66297730f6f14a4eb25
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/16/2018
ms.locfileid: "41824065"
---
<a name="processing-unhandled-exceptions-vb"></a>Обработка необработанных исключений (Visual Basic)
====================
по [Скотт Митчелл](https://twitter.com/ScottOnWriting)

[Просмотреть или скачать образец кода](https://github.com/aspnet/Docs/tree/master/aspnet/web-forms/overview/older-versions-getting-started/deploying-web-site-projects/processing-unhandled-exceptions-vb/samples) ([как скачивать](/aspnet/core/tutorials/index#how-to-download-a-sample))

> При возникновении ошибки среды выполнения веб-приложения в рабочей среде важно для уведомления разработчик и регистрировать в журнале ошибки может быть диагностированы последующий момент времени. Этот учебник содержит общие сведения о том, как ASP.NET обрабатывает ошибки времени выполнения и рассматривает позволяют обеспечить выполнение всякий раз, когда необработанное исключение пузырьки до среды выполнения ASP.NET пользовательский код.


## <a name="introduction"></a>Вступление

При возникновении необработанного исключения в приложении ASP.NET, поднимается среду выполнения ASP.NET, которая вызывает `Error` событий и отображает страницу соответствующее сообщение об ошибке. Существует три типа страниц ошибок: среда выполнения ошибка желтый экрана из смерти (YSOD); сведения об исключении YSOD; и настраиваемые страницы ошибок. В [предыдущем учебном курсе](displaying-a-custom-error-page-vb.md) мы настраивали приложение для использования на пользовательскую страницу ошибки для удаленных пользователей и YSOD сведения об исключении для пользователей, посещающими локально.

С помощью на понятном пользовательскую страницу ошибки, соответствующий вид сайта является предпочтительным по умолчанию YSOD ошибка среды выполнения, но отображение страницы пользовательской ошибки является только частью комплексное решение ошибок. При возникновении ошибки в приложении в рабочей среде, важно, что разработчики получает уведомление об ошибке, чтобы они могли новые причину исключения и устранить ее. Кроме того сведения об ошибке должны войти, чтобы ошибки можно проверить и диагностируются последующий момент времени.

Этом руководстве показано, как получить доступ к аспекты необработанное исключение, таким образом, они могут регистрироваться и разработчик получает уведомление. Два руководства, которые после этого изучите библиотек ведения журнала ошибки, которые после небольшой настройки, автоматически уведомление разработчиков о ошибки времени выполнения и регистрировать их данные.

> [!NOTE]
> Сведения, рассмотренный наиболее полезна в тех случаях, если требуется обработка необработанных исключений некоторым способом уникальный или настроенные. В случаях, где требуется занести исключение в журнал и уведомить разработчик библиотек ведения журнала ошибок с является наилучшим вариантом. В следующих двух учебниках содержатся общие сведения о двух таких библиотеках.


## <a name="executing-code-when-theerrorevent-is-raised"></a>Выполнение кода при`Error`события

События предоставляют объект механизм для сигнализации, что произошло нечто интересное и другим объектом, чтобы выполнить код в ответ. Как разработчик ASP.NET вы привыкли думать с точки зрения событий. Если вы хотите выполнять определенный код, при выборе определенной кнопки, можно создать обработчик событий для этой кнопки `Click` событий и поместите код существует. Учитывая, что среда выполнения ASP.NET вызывает его [ `Error` событий](https://msdn.microsoft.com/library/system.web.httpapplication.error.aspx) всякий раз, когда происходит необработанное исключение, следует, что код для записи сведений об ошибках, попадут в обработчике событий. Но как создать обработчик событий для `Error` событий?

`Error` Событий является одним из многих событий в [ `HttpApplication` класс](https://msdn.microsoft.com/library/system.web.httpapplication.aspx) вызываются на определенном этапе в конвейере HTTP в течение времени существования запроса. Например `HttpApplication` класса [ `BeginRequest` событий](https://msdn.microsoft.com/library/system.web.httpapplication.beginrequest.aspx) вызывается в начале каждого запроса; его [ `AuthenticateRequest` событие](https://msdn.microsoft.com/library/system.web.httpapplication.authenticaterequest.aspx) возникает, когда модуль безопасности определил запрашивающей стороне. Эти `HttpApplication` события предоставления разработчикам страниц способа исполнения пользовательскую логику в различные моменты времени жизни запроса.

Обработчики событий для `HttpApplication` события можно поместить в специальном файле с именем `Global.asax`. Для создания этого файла в веб-сайта, Добавление нового элемента в корень веб-сайта с помощью шаблона глобальный класс приложения с именем `Global.asax`.

[![](processing-unhandled-exceptions-vb/_static/image2.png)](processing-unhandled-exceptions-vb/_static/image1.png)

**Рис. 1**: добавление `Global.asax` в веб-приложение  
 ([Просмотр полноразмерного изображения](processing-unhandled-exceptions-vb/_static/image3.png))

Содержимое и структуру `Global.asax` файл, созданный средой Visual Studio немного отличаются в использовании проекта приложений Web (WAP) или проекта веб-сайта (WSP). С WAP `Global.asax` реализуется как два отдельных файла - `Global.asax` и `Global.asax.vb`. `Global.asax` Ничего не содержит файл, но `@Application` директива, которая ссылается на `.vb` файл; события обработчики интерес определяются в `Global.asax.vb` файл. Для пакетов WSP, создается только один файл, `Global.asax`, и обработчики событий определяются в `<script runat="server">` блока.

`Global.asax` Файла, созданного в WAP шаблоном глобальный класс приложения Visual Studio включает в себя обработчики событий с именем `Application_BeginRequest`, `Application_AuthenticateRequest`, и `Application_Error`, которые являются обработчики событий для `HttpApplication` события `BeginRequest`, `AuthenticateRequest`, и `Error`, соответственно. Также существуют обработчики событий с именем `Application_Start`, `Session_Start`, `Application_End`, и `Session_End`, которые являются обработчики событий, которые возникают при запуске веб-приложения, при запуске нового сеанса при завершении приложения, и при завершении сеанса соответственно. `Global.asax` Файл, созданный Visual Studio в WSP-ФАЙЛ содержит только `Application_Error`, `Application_Start`, `Session_Start`, `Application_End`, и `Session_End` обработчики событий.

> [!NOTE]
> При развертывании приложения ASP.NET необходимо скопировать `Global.asax` файл в рабочую среду. `Global.asax.vb` Файл, который создается в WAP, необходимо скопировать в рабочей среде, так как этот код компилируется в сборку проекта.


Обработчики событий, создаваемого шаблоном глобальный класс приложения Visual Studio не являются исчерпывающими. Можно добавить обработчик событий для любого `HttpApplication` событий путем именования в обработчике событий `Application_EventName`. Например, можно добавить следующий код, чтобы `Global.asax` файл, чтобы создать обработчик событий для [ `AuthorizeRequest` событий](https://msdn.microsoft.com/library/system.web.httpapplication.authorizerequest.aspx):

[!code-vb[Main](processing-unhandled-exceptions-vb/samples/sample1.vb)]

Аналогичным образом вы можете удалить все обработчики событий создан с помощью шаблона глобальный класс приложения, которые не требуются. В этом руководстве потребуется только обработчик событий для `Error` задание, можно удалить другие обработчики событий из `Global.asax` файла.

> [!NOTE]
> *HTTP-модули* другим способом для определения обработчиков событий для `HttpApplication` события. Модули HTTP, создаются как файл класса, который может быть непосредственно в проект веб-приложения или выделены в отдельную библиотеку классов. Так как они могут быть разделены в библиотеку классов, модулей HTTP предлагают более гибкую и многократно используемых модель для создания `HttpApplication` обработчики событий. Тогда как `Global.asax` относится файл веб-приложение, в которых они находятся, можно скомпилировать модули HTTP в сборки, после чего модуль HTTP к веб-сайта добавляется простым, например удаление сборки `Bin` папку и регистрации Модуль в `Web.config`. Этот учебник не рассмотрим создание и использование модулей HTTP, но библиотеки два ведения журнала ошибок, используемые в следующих двух учебных реализованы как модули HTTP. Дополнительные сведения о преимуществах модулей HTTP, см. в разделе [с помощью модулей и обработчиков HTTP для создания подключаемых компонентов ASP.NET](https://msdn.microsoft.com/library/aa479332.aspx).


## <a name="retrieving-information-about-the-unhandled-exception"></a>Получение сведений о необработанном исключении

На этом этапе у нас есть файл Global.asax с `Application_Error` обработчик событий. При выполнении этого обработчика событий, нам нужно уведомлять разработчика ошибки и журнал сведения о нем. Для выполнения этих задач необходимо сначала определить подробности исключения, которое было вызвано. Используйте объект сервера [ `GetLastError` метод](https://msdn.microsoft.com/library/system.web.httpserverutility.getlasterror.aspx) для получения сведений об необработанное исключение, вызвавшее `Error` событие.

[!code-vb[Main](processing-unhandled-exceptions-vb/samples/sample2.vb)]

`GetLastError` Метод возвращает объект типа `Exception`, который является базовым типом для всех исключений в .NET Framework. Тем не менее, в приведенном выше коде я приведение объекта исключения, возвращаемые `GetLastError` в `HttpException` объекта. Если `Error` событие, так как возникло исключение во время обработки ресурса ASP.NET то исключение, выданное переносится внутри `HttpException`. Чтобы получить фактическое исключение, ставшее причиной использования событий ошибки `InnerException` свойство. Если `Error` события из-за исключения на основе HTTP, например запрос для страницы не существует, `HttpException` возникает исключение, но не предоставляет внутреннее исключение.

В следующем коде используется `GetLastErrormessage` для получения сведений об исключении, вызвавшей `Error` событий, хранение `HttpException` в переменной с именем `lastErrorWrapper`. Она сохраняет тип, сообщение и трассировка стека исходного исключения в три строковые переменные, проверять `lastErrorWrapper` — это фактическое исключение, вызвавшее `Error` событий (в случае исключения на основе HTTP), или это просто Программа-оболочка для исключения, выданное при обработке запроса.

[!code-vb[Main](processing-unhandled-exceptions-vb/samples/sample3.vb)]

На этом этапе у вас есть все сведения, необходимые для написания кода, которая будет записывать сведения об исключении в таблицу базы данных. Можно создать таблицу базы данных со столбцами для каждого из сведения об ошибке таким образом, интерес - тип, сообщение, трассировка стека и т. д. — а также другие важные сведения, такие как URL-адрес запрошенной страницы и имя текущего пользователя. В `Application_Error` обработчика событий необходимо затем соединиться с базой данных и вставить запись в таблицу. Аналогичным образом можно добавить код для создания предупреждения, разработчик ошибки по электронной почте.

Ошибка библиотеки ведения журналов, проверены в следующих двух учебных курсах обеспечения возможности по умолчанию, поэтому нет необходимости это ведение журнала ошибок и уведомления самостоятельно создавать. Тем не менее чтобы проиллюстрировать, `Error` вызывается событие и что `Application_Error` обработчик событий может использоваться в журнал сведения об ошибке и уведомлять разработчика, давайте добавим код, который уведомляет разработчика при возникновении ошибки.

## <a name="notifying-a-developer-when-an-unhandled-exception-occurs"></a>Уведомления разработчик, при возникновении необработанного исключения

При возникновении необработанного исключения в рабочей среде важно для предупреждения разработчиков, чтобы они могут оценить ошибки и определить, какие действия необходимо выполнить. К примеру при возникновении ошибки в подключении к базе данных, вам потребуется double Проверьте строки подключения и, возможно, откройте запрос в службу поддержки с веб-хостинга компанию. Если исключение произошло из-за ошибки программирования, дополнительный код или логику проверки может потребоваться добавить избежать таких ошибок в будущем.

Классы .NET Framework в [ `System.Net.Mail` пространства имен](https://msdn.microsoft.com/library/system.net.mail.aspx) позволяют легко отправить сообщение электронной почты. [ `MailMessage` Класс](https://msdn.microsoft.com/library/system.net.mail.mailmessage.aspx) представляет сообщение электронной почты и имеет свойства, такие как `To`, `From`, `Subject`, `Body`, и `Attachments`. `SmtpClass` Используется для отправки `MailMessage` с помощью указанного сервера SMTP; параметры SMTP-сервера могут быть заданы программным или декларативным способом в [ `<system.net>` элемент](https://msdn.microsoft.com/library/6484zdc1.aspx) в `Web.config file`. Дополнительные сведения об отправке по электронной почте сообщений в приложении ASP.NET ознакомьтесь с моей статье [отправки электронной почты в ASP.NET](http://aspnet.4guysfromrolla.com/articles/072606-1.aspx)и [System.Net.Mail часто задаваемые вопросы о](http://systemnetmail.com/).

> [!NOTE]
> `<system.net>` Элемент содержит параметры SMTP-сервера, используемые `SmtpClient` класса при отправке сообщения электронной почты. Веб-хостинга компанию, скорее всего, имеет SMTP-сервер, который можно использовать для отправки электронной почты из приложения. Сведения о параметры SMTP-сервера, который следует использовать в веб-приложении см в разделе поддержки работы веб-узла.


Добавьте следующий код, чтобы `Application_Error` обработчик событий для отправки сообщения электронной почты разработчика, при возникновении ошибки:

[!code-vb[Main](processing-unhandled-exceptions-vb/samples/sample4.vb)]

Хотя приведенный выше код является довольно длинный, основная часть его создается HTML, отображаемый в сообщении электронной почты, отправленных для разработчика. Запускает код, ссылаясь на `HttpException` возвращаемые `GetLastError` метод (`lastErrorWrapper`). Фактическое исключение, возникшее в запросе получается через `lastErrorWrapper.InnerException` и присваивается переменной `lastError`. Тип, сообщение и стек трассировки информация извлекается из `lastError` и хранятся в трех строковых переменных.

Далее, `MailMessage` объект с именем `mm` создается. Текст сообщения электронной почты в формате HTML и отображает URL-адрес запрошенной страницы, имя текущего пользователя и сведения об исключении (тип, сообщение и трассировка стека). Одно из замечательных особенностей `HttpException` класс является то, что можно создать HTML-код, используемый для создания исключения сведения желтый экрана из смерти (YSOD) путем вызова [GetHtmlErrorMessage метод](https://msdn.microsoft.com/library/system.web.httpexception.gethtmlerrormessage.aspx). Этот метод используется здесь для получения разметки YSOD сведения об исключении и его добавления в виде вложения сообщения электронной почты. Будьте осторожны: Если на исключение, которое вызвало `Error` событие произошло исключение на основе HTTP (например, запрос страницы не существуют) то `GetHtmlErrorMessage` метод возвратит `null`.

Последним шагом является отправка `MailMessage`. Это делается путем создания нового `SmtpClient` метод и вызвать его `Send` метод.

> [!NOTE]
> Для использования этого кода в веб-приложения нужно будет изменить значения в `ToAddress` и `FromAddress` константы из support@example.com на любой адрес электронной почты должны отправляться на адрес уведомления по электронной почте об и получаются из. Также необходимо указать параметры SMTP-сервера в `<system.net>` статьи `Web.config`. См. в поставщике веб-размещения, чтобы определить параметры SMTP-сервера для использования.


В таком коде каждый раз, когда возникает ошибка разработчик отправляется сообщение электронной почты, содержатся сведения о ней и YSOD. В предыдущем учебном курсе мы продемонстрировали ошибка времени выполнения, посетив Genre.aspx и передача недействительного `ID` значение через строку запроса, например `Genre.aspx?ID=foo`. Посетив страницу с `Global.asax` файла на месте создает пользователям те же возможности, как в предыдущем учебном курсе — в среде разработки вы перейдете к см. в разделе исключения сведения желтый экрана из смертью, хотя в рабочей среде вам потребуется см. в разделе пользовательскую страницу ошибки. Помимо этого поведения существующих разработчик отправляется сообщение электронной почты.

**Рис. 2** показаны сообщения, полученные при посещении `Genre.aspx?ID=foo`. Текст сообщения электронной почты представлены сведения об исключении, хотя `YSOD.htm` вложения отображается содержимое, которое отображается в YSOD подробности исключения (см. в разделе **рис. 3**).

[![](processing-unhandled-exceptions-vb/_static/image5.png)](processing-unhandled-exceptions-vb/_static/image4.png)

**Рис. 2**: разработчик отправляется уведомление по электронной почте при каждом обнаружении необработанного исключения  
 ([Просмотр полноразмерного изображения](processing-unhandled-exceptions-vb/_static/image6.png))

[![](processing-unhandled-exceptions-vb/_static/image8.png)](processing-unhandled-exceptions-vb/_static/image7.png)

**Рис. 3**: уведомление по электронной почте включает в себя сведения об исключении YSOD как вложение  
 ([Просмотр полноразмерного изображения](processing-unhandled-exceptions-vb/_static/image9.png))

## <a name="what-about-using-the-custom-error-page"></a>Как насчет используете пользовательскую страницу ошибки?

Этом руководстве было показано, как использовать `Global.asax` и `Application_Error` обработчик событий для выполнения кода при возникновении необработанного исключения. В частности мы использовали этот обработчик событий для уведомления разработчиков ошибки; Теперь можно расширить ее также могут регистрировать сведения об ошибке в базе данных. Наличие `Application_Error` обработчик событий не влияет на взаимодействие с пользователем. Они по-прежнему см. в разделе страницы настроенные ошибки, будь то YSOD сведения об ошибке, YSOD ошибка среды выполнения или пользовательскую страницу ошибки.

Это бы логичным проверить ли `Global.asax` файл и `Application_Error` событий требуется, при использовании на пользовательскую страницу ошибки. При возникновении ошибки, то пользователю будет показано пользовательскую страницу ошибки так почему не удается мы поместите код уведомлять разработчика и журнала сведения об ошибке в класс кода программной части страницы настраиваемых ошибок? Хотя безусловно можно добавить код к вспомогательному классу страницы настраиваемых ошибок у вас нет доступа к сведениям о исключение, вызвавшее `Error` событие при помощи технологии мы изучили в предыдущем учебном курсе. Вызов `GetLastError` метод из пользовательскую страницу ошибки возвращает `Nothing`.

Причина такого поведения — так, как пользовательскую страницу ошибки достигается с помощью перенаправления. Когда необработанное исключение достигает среда выполнения ASP.NET механизм ASP.NET вызывает его `Error` событий (которая выполняет `Application_Error` обработчик событий) и затем *перенаправляет* пользователя на пользовательскую страницу ошибки, выполнив `Response.Redirect(customErrorPageUrl)`. `Response.Redirect` Метод отправляет ответ клиенту с кодом состояния HTTP 302, предписывая браузер для запроса нового URL-адрес, а именно пользовательскую страницу ошибки. Затем браузер автоматически запрашивает эта новая страница. Чтобы узнать что пользовательскую страницу ошибки была запрошена отдельно на странице где возникла ошибка, так как в адресной строке браузера изменяется на URL-адрес страницы настраиваемых ошибок (см. в разделе **рис. 4**).

[![](processing-unhandled-exceptions-vb/_static/image11.png)](processing-unhandled-exceptions-vb/_static/image10.png)

**Рис. 4**: при возникновении ошибки, браузер перенаправляется на URL-адрес страницы настраиваемых ошибок  
 ([Просмотр полноразмерного изображения](processing-unhandled-exceptions-vb/_static/image12.png))

В итоге получается, что запрос, где произошло необработанное исключение заканчивается, когда сервер не ответит перенаправления HTTP 302. Последующие запросы к пользовательскую страницу ошибки — это новый запрос; к этому моменту ASP.NET ядра отменила сведения об ошибке и, кроме того, не должен быть сопоставлен необработанное исключение в предыдущем запросе нового запроса для пользовательскую страницу ошибки. Вот почему `GetLastError` возвращает `null` при вызове из пользовательскую страницу ошибки.

Тем не менее это может быть пользовательскую страницу ошибки во время тот же запрос, который вызвал ошибку. [ `Server.Transfer(url)` ](https://msdn.microsoft.com/library/system.web.httpserverutility.transfer.aspx) Метод передает выполнение на указанный URL-адрес и обрабатывает его в тот же запрос. Можно переместить код `Application_Error` обработчик событий к вспомогательному классу страницы настраиваемых ошибок, заменяя его в `Global.asax` следующим кодом:

[!code-vb[Main](processing-unhandled-exceptions-vb/samples/sample5.vb)]

Когда происходит необработанное исключение `Application_Error` обработчик событий передает управление соответствующие пользовательскую страницу ошибки на основании кода состояния HTTP. Так как элемент управления был передан, пользовательскую страницу ошибки имеет доступ к сведениям необработанное исключение с помощью `Server.GetLastError` уведомлять разработчика ошибки и журнал сведения о нем. `Server.Transfer` Вызов останавливает ядро ASP.NET из перенаправления пользователя на пользовательскую страницу ошибки. Вместо этого содержимое страницы пользовательской ошибки возвращаются в виде ответа на страницу, вызвавшего ошибку.

## <a name="summary"></a>Сводка

При возникновении необработанного исключения в веб-приложении ASP.NET среда выполнения ASP.NET вызывает `Error` событий и отображает страницу настроенные ошибки. Мы сообщим, разработчик ошибки, войдите в сведения о нем или обрабатывает каким-либо другим образом, Создание обработчика событий для события ошибки. Существует два способа, чтобы создать обработчик событий для `HttpApplication` событий, таких как `Error`: в `Global.asax` файла или с помощью модуля HTTP. Этом руководстве были рассмотрены способы создания `Error` обработчик событий в `Global.asax` файл, который уведомляет разработчиков ошибки в сообщении электронной почты.

Создание `Error` обработчик событий полезно, если требуется обработка необработанных исключений некоторым способом уникальный или настроенные. Однако создание собственной `Error` обработчик событий исключений в журнале или для уведомления разработчик не наиболее эффективно использовать свое время, так как уже существующие библиотеки ведения журналов бесплатный и простой в использовании ошибки, которые можно настроить в считанные минуты. Следующие два руководства изучите двух таких библиотеках.

Счастливого вам программирования!

### <a name="further-reading"></a>Дополнительные сведения

Дополнительные сведения по темам, обсуждавшимся в этом руководстве см. в следующих ресурсах:

- [HTTP-модули ASP.NET и общие сведения об обработчиках HTTP](https://support.microsoft.com/kb/307985)
- [Корректно отвечать на необработанные исключения - обработка необработанных исключений](http://aspnet.4guysfromrolla.com/articles/091306-1.aspx)
- [`HttpApplication` Класс и объект приложения ASP.NET](http://www.eggheadcafe.com/articles/20030211.asp)
- [HTTP-обработчики и модули HTTP в ASP.NET](http://www.15seconds.com/Issue/020417.htm)
- [Отправка сообщения электронной почты в ASP.NET](http://aspnet.4guysfromrolla.com/articles/072606-1.aspx)
- [Основные сведения о `Global.asax` файла](http://aspalliance.com/1114_Understanding_the_Globalasax_file.all)
- [Создание подключаемых компонентов ASP.NET с помощью HTTP-модули и обработчики](https://msdn.microsoft.com/library/aa479332.aspx)
- [Работа с ASP.NET `Global.asax` файла](http://articles.techrepublic.com.com/5100-10878_11-5771721.html)
- [Работа с `HttpApplication` экземпляров](https://msdn.microsoft.com/library/a0xez8f2.aspx)

> [!div class="step-by-step"]
> [Назад](displaying-a-custom-error-page-vb.md)
> [Вперед](logging-error-details-with-asp-net-health-monitoring-vb.md)
