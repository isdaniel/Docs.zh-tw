---
title: 回應快取中 ASP.NET Core 中的介軟體
author: guardrex
description: 了解如何設定和使用 ASP.NET Core 中的回應快取中介軟體。
monikerRange: '>= aspnetcore-1.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/26/2017
uid: performance/caching/middleware
ms.openlocfilehash: f4e5a414b92e3ca65e19188ebd2bfaef6f32fee7
ms.sourcegitcommit: 4cd8dce371d63a66d780e4af1baab2bcf9d61b24
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "43893086"
---
# <a name="response-caching-middleware-in-aspnet-core"></a>回應快取中 ASP.NET Core 中的介軟體

藉由[Luke Latham](https://github.com/guardrex)和[John Luo](https://github.com/JunTaoLuo)

[檢視或下載 ASP.NET Core 2.1 的範例程式碼](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/middleware/samples)([如何下載](xref:tutorials/index#how-to-download-a-sample))

這篇文章說明如何在 ASP.NET Core 應用程式中設定回應快取中介軟體。 中介軟體會判斷何時可快取的回應、 存放區回應，以及可回應從快取。 如需 HTTP 快取的簡介和`ResponseCache`屬性，請參閱[回應快取](xref:performance/caching/response)。

## <a name="package"></a>Package

::: moniker range=">= aspnetcore-2.1"

參考[Microsoft.AspNetCore.App 中繼套件](xref:fundamentals/metapackage-app)或新增的套件參考[Microsoft.AspNetCore.ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/)封裝。

::: moniker-end

::: moniker range="= aspnetcore-2.0"

參考[Microsoft.AspNetCore.All 中繼套件](xref:fundamentals/metapackage)或新增的套件參考[Microsoft.AspNetCore.ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/)封裝。

::: moniker-end

::: moniker range="= aspnetcore-1.1"

新增的套件參考[Microsoft.AspNetCore.ResponseCaching](https://www.nuget.org/packages/Microsoft.AspNetCore.ResponseCaching/)封裝。

::: moniker-end

## <a name="configuration"></a>組態

在  `Startup.ConfigureServices`，將中介軟體新增至服務集合。

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet1&highlight=9)]

設定應用程式使用中的介軟體與`UseResponseCaching`擴充方法，將中介軟體新增至要求處理管線。 範例應用程式會新增[ `Cache-Control` ](https://tools.ietf.org/html/rfc7234#section-5.2)快取可快取的回應達 10 秒的回應標頭。 此範例會傳送[ `Vary` ](https://tools.ietf.org/html/rfc7231#section-7.1.4)標頭，若要設定中介軟體提供快取回的應只有當[ `Accept-Encoding` ](https://tools.ietf.org/html/rfc7231#section-5.3.4)後續的要求標頭是否完全符合原始要求。 在程式碼範例中，如下所示[CacheControlHeaderValue](/dotnet/api/microsoft.net.http.headers.cachecontrolheadervalue)並[HeaderNames](/dotnet/api/microsoft.net.http.headers.headernames)需要`using`陳述式[Microsoft.Net.Http.Headers](/dotnet/api/microsoft.net.http.headers)命名空間。

[!code-csharp[](middleware/samples/2.x/ResponseCachingMiddleware/Startup.cs?name=snippet2&highlight=17,22-29)]

回應快取中介軟體只會快取伺服器的回應，導致 200 （確定） 狀態碼。 任何其他回應，包括[錯誤網頁](xref:fundamentals/error-handling)中, 介軟體會被忽略。

> [!WARNING]
> 包含內容的已驗證的用戶端的回應必須標示為不可快取，以防止從儲存和處理這些回應的中介軟體。 請參閱[快取的條件](#conditions-for-caching)如需有關如何在中介軟體，判斷回應是否可快取。

## <a name="options"></a>選項

中介軟體會提供三個選項可控制的回應快取。

| 選項                | 描述 |
| --------------------- | ----------- |
| UseCaseSensitivePaths | 決定回應會快取在區分大小寫的路徑上。 預設值是 `false`。 |
| MaximumBodySize       | 回應內文，以位元組為單位的最大的快取大小。 預設值是`64 * 1024 * 1024`(64 MB)。 |
| SizeLimit             | 回應快取中介軟體，以位元組為單位的大小限制。 預設值是`100 * 1024 * 1024`(100 MB)。 |

下列範例會設定中的介軟體：

* 快取回應小於或等於 1024 個位元組。
* 將回應儲存透過區分大小寫的路徑 (例如`/page1`和`/Page1`分開儲存)。

```csharp
services.AddResponseCaching(options =>
{
    options.UseCaseSensitivePaths = true;
    options.MaximumBodySize = 1024;
});
```

## <a name="varybyquerykeys"></a>VaryByQueryKeys

使用 MVC/Web API 控制器或 Razor 頁面的頁面模型時`ResponseCache`屬性指定設定適當的標頭的回應快取所需的參數。 唯一的參數`ResponseCache`嚴格需要的中介軟體的屬性是`VaryByQueryKeys`，不會對應到實際的 HTTP 標頭。 如需詳細資訊，請參閱 < [ResponseCache 屬性](xref:performance/caching/response#responsecache-attribute)。

不使用時`ResponseCache`屬性，回應快取可因使用`VaryByQueryKeys`功能。 使用`ResponseCachingFeature`直接從`IFeatureCollection`的`HttpContext`:

```csharp
var responseCachingFeature = context.HttpContext.Features.Get<IResponseCachingFeature>();
if (responseCachingFeature != null)
{
    responseCachingFeature.VaryByQueryKeys = new[] { "MyKey" };
}
```

使用單一的值等於`*`在`VaryByQueryKeys`會快取所要求的所有查詢參數而有所不同。

## <a name="http-headers-used-by-response-caching-middleware"></a>回應快取中介軟體所使用的 HTTP 標頭

回應快取中介軟體會設定使用 HTTP 標頭。

| 頁首 | 詳細資料 |
| ------ | ------- |
| 授權 | 如果標頭存在，未快取的回應。 |
| Cache-Control | 中介軟體只會考慮快取回應標記為`public`快取指示詞。 控制快取使用下列參數：<ul><li>最大壽命</li><li>max-stale&#8224;</li><li>最小全新</li><li>必須過期</li><li>無快取</li><li>無存放區</li><li>僅限 if-快取</li><li>private</li><li>public</li><li>s maxage</li><li>指令&#8225;</li></ul>&#8224;若要指定無限制到`max-stale`中, 介軟體會採取任何動作。<br>&#8225;`proxy-revalidate`具有相同的效果`must-revalidate`。<br><br>如需詳細資訊，請參閱 < [RFC 7231： 要求的快取控制指示詞](https://tools.ietf.org/html/rfc7234#section-5.2.1)。 |
| Pragma | A`Pragma: no-cache`要求標頭會產生相同的效果`Cache-Control: no-cache`。 此標頭中的相關指示詞會覆寫`Cache-Control`標頭，如果有的話。 考慮使用 HTTP/1.0 的回溯相容性。 |
| Set-Cookie | 如果標頭存在，未快取的回應。 設定一或多個 cookie 的要求處理管線中的任何中介軟體會防止從快取的回應的回應快取中介軟體 (例如[cookie 架構 TempData 提供者](xref:fundamentals/app-state#tempdata))。  |
| 而有所不同 | `Vary`標頭，會由另一個標頭用來改變快取回的應。 例如，快取所包含的編碼方式的回應`Vary: Accept-Encoding`標頭，來快取回應的要求標頭`Accept-Encoding: gzip`和`Accept-Encoding: text/plain`分開。 回應標頭值是`*`絕對不會儲存。 |
| 到期 | 此標頭視為過時的回應不是儲存或擷取除非覆寫其他`Cache-Control`標頭。 |
| 如果 If-none-Match | 如果值不是從快取提供完整的回應`*`而`ETag`的回應不符合任何提供的值。 否則，會提供 304 （未修改） 的回應。 |
| 如果修改自 | 如果`If-None-Match`標頭不存在，如果快取的回應日期比所提供的值從快取提供完整的回應。 否則，會提供 304 （未修改） 的回應。 |
| 日期 | 提供從快取時`Date`標頭設定中介軟體，如果它未提供原始回應。 |
| 內容長度 | 提供從快取時`Content-Length`標頭設定中介軟體，如果它未提供原始回應。 |
| 存留期 | `Age`原始回應中傳送的標頭被忽略。 提供快取的回應時中, 介軟體會計算新的值。 |

## <a name="caching-respects-request-cache-control-directives"></a>快取會遵守要求快取控制指示詞

中介軟體會遵守的規則[快取的 HTTP 1.1 規格](https://tools.ietf.org/html/rfc7234#section-5.2)。 規則需要接受有效的快取`Cache-Control`用戶端傳送的標頭。 在規格中，在用戶端可要求`no-cache`標頭值並強制產生新的回應，每個要求的伺服器。 目前沒有任何開發人員會控制此快取的行為時使用的中介軟體，因為中介軟體會遵守官方的快取規格。

進一步控制快取行為的詳細資訊，探索 ASP.NET Core 的其他快取功能。 請參閱下列主題：

* [記憶體中快取](xref:performance/caching/memory)
* [使用分散式快取](xref:performance/caching/distributed)
* [快取中的 ASP.NET Core MVC 標籤協助程式](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [分散式快取標籤協助程式](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)

## <a name="troubleshooting"></a>疑難排解

如果快取行為，將無法如預期般運作，請確認回應的快取，而且能夠從快取所提供。 檢查要求的連入標頭和回應的傳出標頭。 啟用[記錄](xref:fundamentals/logging/index)來協助偵錯。

當進行測試和疑難排解快取行為，在瀏覽器可能設定影響不想要的方式快取的要求標頭。 例如，可能會設定瀏覽器`Cache-Control`標頭`no-cache`或`max-age=0`時重新整理頁面。 下列工具可以明確設定要求標頭，並正適合測試快取：

* [Fiddler](https://www.telerik.com/fiddler)
* [Postman](https://www.getpostman.com/)

### <a name="conditions-for-caching"></a>快取的條件

* 伺服器回應 200 （確定） 狀態碼必須產生要求。
* 要求方法必須是 GET 或 HEAD。
* 終端機中介軟體，例如[靜態檔案中介軟體](xref:fundamentals/static-files)，必須處理前回應快取中介軟體的回應。
* `Authorization`不得存在於標頭。
* `Cache-Control` 標頭參數必須有效，且必須標示為回應`public`而未標示為`private`。
* `Pragma: no-cache`標頭不能有如果`Cache-Control`標頭不存在，作為`Cache-Control`標頭會覆寫`Pragma`標頭時出現。
* `Set-Cookie`不得存在於標頭。
* `Vary` 標頭參數必須是有效且不等於`*`。
* `Content-Length`標頭值 (如果設定) 必須符合回應主體的大小。
* [IHttpSendFileFeature](/dotnet/api/microsoft.aspnetcore.http.features.ihttpsendfilefeature)不會使用。
* 回應不是所指定的過時`Expires`標頭和`max-age`和`s-maxage`快取指示詞。
* 回應緩衝必須成功，而且回應的大小必須小於所設定，或預設`SizeLimit`。
* 回應必須是根據快取[RFC 7234](https://tools.ietf.org/html/rfc7234)規格。 比方說，`no-store`指示詞不能存在於要求或回應標頭欄位。 請參閱*第 3 節： 快取中的 儲存回應*的[RFC 7234](https://tools.ietf.org/html/rfc7234)如需詳細資訊。

> [!NOTE]
> 防偽系統是否有產生安全的權杖，以防止跨網站要求偽造 (CSRF) 攻擊集`Cache-Control`並`Pragma`標頭`no-cache`，因此不會快取的回應。 如需如何停用 HTML 表單元素的 antiforgery 權杖的資訊，請參閱[ASP.NET Core antiforgery 組態](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)。

## <a name="additional-resources"></a>其他資源

* [應用程式啟動](xref:fundamentals/startup)
* [中介軟體](xref:fundamentals/middleware/index)
* [記憶體中快取](xref:performance/caching/memory)
* [使用分散式快取](xref:performance/caching/distributed)
* [使用變更權杖來偵測變更](xref:fundamentals/primitives/change-tokens)
* [回應快取](xref:performance/caching/response)
* [快取標籤協助程式](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [分散式快取標籤協助程式](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)
