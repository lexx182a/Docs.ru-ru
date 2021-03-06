---
title: Привязка модели в ASP.NET Core
author: tdykstra
description: Узнайте, как привязка модели в ASP.NET Core MVC сопоставляет данные из HTTP-запросов с параметрами методов действия.
ms.assetid: 0be164aa-1d72-4192-bd6b-192c9c301164
ms.author: tdykstra
ms.date: 08/14/2018
uid: mvc/models/model-binding
ms.openlocfilehash: 0ce20a8040c6b19da1f57e1c053a7ef81d8bcb23
ms.sourcegitcommit: d53e0cc71542b92de867bcce51575b054886f529
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/16/2018
ms.locfileid: "41751636"
---
# <a name="model-binding-in-aspnet-core"></a>Привязка модели в ASP.NET Core

Автор: [Рэйчел Аппель](https://github.com/rachelappel) (Rachel Appel)

## <a name="introduction-to-model-binding"></a>Общие сведения о привязке модели

Привязка модели в ASP.NET Core MVC сопоставляет данные из HTTP-запросов с параметрами методов действия. Параметры могут быть простыми типами, такими как строки, целые числа или значения с плавающей запятой, или они могут быть сложными типами. Это отличная возможность в MVC, так как сопоставление входящих данных с аналогами часто является повторяющейся операцией независимо от размера и сложности данных. В MVC эта проблема решается путем абстрагирования привязки, поэтому разработчикам не нужно переписывать незначительно отличающуюся версию одного и того же кода в каждом приложении. Написание собственного кода для преобразователя типов занимает много времени и подвержено ошибкам.

## <a name="how-model-binding-works"></a>Принцип действия привязки модели

Когда MVC получает HTTP-запрос, он направляет его в определенный метод действия контроллера. Он определяет, какой метод действия необходимо запустить с учетом данных маршрута, а затем привязывает значения из HTTP-запроса к параметрам метода действия. Например, рассмотрим следующий URL-адрес:

`http://contoso.com/movies/edit/2`

Поскольку шаблон маршрута имеет вид `{controller=Home}/{action=Index}/{id?}`, направляет `movies/edit/2` в контроллер `Movies`, и его метод действия `Edit`. Он также принимает необязательный параметр `id`. Код для метода действия должен выглядеть примерно следующим образом:

```csharp
public IActionResult Edit(int? id)
   ```

Примечание. Строки в URL-адресе не учитывают регистр.

MVC попытается привязать данные запроса к параметрам действия по имени. MVC будет искать значения для каждого параметра, используя имя параметра и имена его открытых задаваемых свойств. В приведенном выше примере единственным параметром действия является `id`, который MVC привязывает к значению с тем же именем в значениях маршрута. Помимо значений маршрута, MVC будет привязывать данные из различных частей запроса, следуя установленному порядку. Ниже приведен список источников данных в порядке, в котором их просматривает привязка модели.

1. `Form values`. Это значения формы, включаемые в HTTP-запрос с помощью метода POST. (включая запросы POST jQuery)

2. `Route values`. Набор значений маршрута, предоставляемых функцией [маршрутизации](xref:fundamentals/routing).

3. `Query strings`. Часть строки запроса URI.

<!-- DocFX BUG
The link works but generates an error when building with DocFX
@fundamentals/routing
[Routing](xref:fundamentals/routing)
-->

Примечание. Значения форм, данные маршрутов и строки запросов хранятся как пары "имя-значение".

Так как привязка модели запросила ключ `id`, а в значениях формы нет ничего с именем `id`, она перешла к поиску ключа в значениях маршрута. В нашем примере найдено совпадение. Выполняется привязка, а значение преобразуется в целое число 2. Тот же запрос с использованием Edit(string id) приведет к преобразованию в строку "2".

Пока в примере используются простые типы. В MVC простыми типами являются любые типы-примитивы .NET или типы с преобразователем строковых типов. Если бы параметром метода действия был класс, такой как тип `Movie`, который в качестве свойств содержал простые и сложные типы, привязка модели MVC обработала бы его соответствующим образом. При поиске совпадений она использует отражение и рекурсию для обхода свойств сложных типов. Для привязки значений к свойствам привязка модели ищет шаблон *имя_параметра.имя_свойства*. Если она не находит совпадающие значения этого вида, будет предпринята попытка привязки с помощью только имени свойства. Для таких типов, например типов `Collection`, привязка модели ищет совпадения с *имя_параметра [индекс]* или просто *[индекс]*. Привязка модели аналогичным образом обрабатывает типы `Dictionary`, выполняя поиск *имя_параметра[ключ]* или просто *[ключ]*, при условии, что ключи являются простыми типами. Поддерживаемые ключи совпадают с именами полей HTML и вспомогательными функциями тегов, созданными для того же типа модели. Благодаря полной совместимости значений для удобства пользователей в полях формы остаются введенные значения, например, если привязанные данные не прошли проверку.

Для успешного выполнения привязки класс должен иметь открытый конструктор по умолчанию и элемент для привязки с открытыми свойствами для записи. Когда происходит привязка модели, класс создается только с помощью открытого конструктора по умолчанию, а затем можно задать свойства.

После привязки параметра привязка модели прекращает поиск значений с таким именем и переходит к привязке следующего параметра. В противном случае привязка модели по умолчанию задает для параметров значения по умолчанию в зависимости от их типа:

* `T[]`. За исключением массивов типа `byte[]`, привязка задает параметрам типа `T[]` значение `Array.Empty<T>()`. Массивам типа `byte[]` задается значение `null`.

* Ссылочные типы. Привязка создает экземпляр класса с помощью конструктора по умолчанию без задания свойств. Однако привязка модели задает параметрам типа `string` значение `null`.

* Типы, допускающие значение NULL. Типам, допускающим значение NULL, задается значение `null`. В приведенном выше примере привязка модели задает `id` значение `null`, так как он имеет тип `int?`.

* Типы значений. Типам значений, не допускающим значение NULL и имеющим тип `T`, задается значение `default(T)`. Например, привязка модели присвоит параметру `int id` значение 0. Вместо значений по умолчанию рекомендуется использовать проверку модели или типы, допускающие значение NULL.

Если привязка завершается сбоем, MVC не вызывает ошибку. Каждое действие, которое принимает вводимые пользователем данные, должно проверять свойство `ModelState.IsValid`.

Примечание. Каждая запись в свойстве `ModelState` контроллера является классом `ModelStateEntry`, содержащим свойство `Errors`. Необходимость запрашивать эту коллекцию самостоятельно возникает редко. Взамен рекомендуется использовать `ModelState.IsValid`.

Кроме того, существуют некоторые особые типы данных, которые MVC необходимо учитывать при выполнении привязки модели:

* `IFormFile`, `IEnumerable<IFormFile>`. Один или несколько отправленных файлов, которые являются частью HTTP-запроса.

* `CancellationToken`. Используется для отмены действия в асинхронных контроллерах.

Эти типы можно привязать к параметрам действия или к свойствам в типе класса.

После привязки модели происходит [проверка](validation.md). Привязка модели по умолчанию отлично подходит для подавляющего большинства сценариев разработки. Она также является расширяемой, поэтому если у вас есть особые потребности, можно настроить встроенное поведение.

## <a name="customize-model-binding-behavior-with-attributes"></a>Настройка поведения привязки модели с помощью атрибутов

В MVC есть несколько атрибутов, которые можно использовать для направления поведения привязки модели по умолчанию в другой источник. Например, с помощью атрибутов `[BindRequired]` или `[BindNever]` можно указать, требуется ли привязка для свойства или она вообще не должна выполняться. Кроме того, можно переопределить источник данных по умолчанию и указать источник данных связывателя модели. Ниже приведен список атрибутов привязки модели.

* `[BindRequired]`. Этот атрибут добавляет ошибку состояния модели, если привязку выполнить невозможно.

* `[BindNever]`. Указывает связывателю модели никогда не выполнять привязку к этому параметру.

* `[FromHeader]`, `[FromQuery]`, `[FromRoute]`, `[FromForm]`. Используются для указания точного источника привязки, который необходимо применить.

* `[FromServices]`. Этот атрибут использует [внедрение зависимостей](../../fundamentals/dependency-injection.md) для привязки параметров из служб.

* `[FromBody]`. Используйте настроенные модули форматирования для привязки данных из текста запроса. Модуль форматирования выбирается в зависимости от типа содержимого запроса.

* `[ModelBinder]`. Используется для переопределения связывателя модели, источника привязки и имени по умолчанию.

Атрибуты очень полезны при переопределении поведения по умолчанию привязки модели.

## <a name="customize-model-binding-and-validation-globally"></a>Глобальная настройка привязки и проверки модели

Поведение привязки модели и системы проверки определяется классом [ModelMetadata](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata), который описывает:

* Как модель будет привязана.
* Как выполняется проверка типа и его свойств.

Аспекты поведения системы можно настроить глобально, добавив поставщика сведений в [MvcOptions.ModelMetadataDetailsProviders](/dotnet/api/microsoft.aspnetcore.mvc.mvcoptions.modelmetadatadetailsproviders#Microsoft_AspNetCore_Mvc_MvcOptions_ModelMetadataDetailsProviders). MVC имеет несколько встроенных поставщиков сведений, которые позволяют настраивать такое поведение, как отключение привязки модели или проверки для определенных типов.

Для отключения привязки модели для всех моделей определенного типа добавьте [ExcludeBindingMetadataProvider](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.metadata.excludebindingmetadataprovider) в `Startup.ConfigureServices`. Например, для отключения привязки модели для всех моделей типа `System.Version`:

```csharp
services.AddMvc().AddMvcOptions(options =>
    options.ModelMetadataDetailsProviders.Add(
        new ExcludeBindingMetadataProvider(typeof(System.Version))));
```

Чтобы отключить проверку свойств определенного типа, добавьте [SuppressChildValidationMetadataProvider](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.suppresschildvalidationmetadataprovider) в `Startup.ConfigureServices`. Например, чтобы отключить проверку по свойствам типа `System.Guid`:

```csharp
services.AddMvc().AddMvcOptions(options =>
    options.ModelMetadataDetailsProviders.Add(
        new SuppressChildValidationMetadataProvider(typeof(System.Guid))));
```

## <a name="bind-formatted-data-from-the-request-body"></a>Привязка отформатированных данных из текста запроса

Данные запроса могут иметь различные форматы, включая JSON, XML и многие другие. Если вы используете атрибут [FromBody], чтобы указать необходимость привязки параметра к данным в тексте запроса, MVC использует настроенный набор модулей форматирования для обработки данных запроса на основе его типа содержимого. По умолчанию MVC содержит класс `JsonInputFormatter` для обработки данных JSON, но вы можете добавить дополнительные модули форматирования для обработки XML и других пользовательских форматов.

> [!NOTE]
> Для каждого действия с атрибутом `[FromBody]` может существовать не более одного параметра. Среда выполнения ASP.NET Core MVC делегирует ответственность за считывание потока запроса модулю форматирования. После считывания потока запроса для параметра, как правило, невозможно повторно считать поток запроса для привязки других параметров `[FromBody]`.

> [!NOTE]
> `JsonInputFormatter` является модулем форматирования по умолчанию и основан на [Json.NET](https://www.newtonsoft.com/json).

ASP.NET Core выбирает модули форматирования входных данных на основе заголовка [Content-Type](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html) и типа параметра, если отсутствует примененный к нему атрибут, указывающий иное. Чтобы использовать XML или другой формат, его необходимо настроить в файле *Startup.cs*, но, возможно, сначала потребуется получить ссылку на `Microsoft.AspNetCore.Mvc.Formatters.Xml` с помощью NuGet. Код запуска должен выглядеть примерно следующим образом:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddXmlSerializerFormatters();
   }
```

Код в файле *Startup.cs* содержит метод `ConfigureServices` с аргументом `services`, который можно использовать для построения служб для приложения ASP.NET Core. В примере показано добавление модуля форматирования XML в качестве службы, которую MVC будет предоставлять для этого приложения. Аргумент `options`, переданный в метод `AddMvc`, позволяет добавлять фильтры, модули форматирования и другие системные параметры из MVC при запуске приложения и управлять ими. Затем для работы с необходимым форматом атрибут `Consumes` применяется к классам контроллера или методам действий.

### <a name="custom-model-binding"></a>Пользовательская привязка модели

Для расширения возможностей привязки модели можно написать собственные пользовательские связыватели модели. Узнайте больше о [пользовательской привязке модели](../advanced/custom-model-binding.md).
