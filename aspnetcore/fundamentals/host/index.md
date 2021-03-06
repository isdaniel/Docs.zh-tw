---
title: 在 ASP.NET Core 中代管
author: guardrex
description: 深入了解 ASP.NET Core 的 Web 代管以及 .NET 的一般代管，其負責啟動應用程式以及進行存留期間的管理。
ms.author: riande
ms.custom: mvc
ms.date: 08/28/2018
uid: fundamentals/host/index
ms.openlocfilehash: 9927722b5080beb94e5628d9e7b54e6d50a5bff8
ms.sourcegitcommit: a669c4e3f42e387e214a354ac4143555602e6f66
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43336046"
---
# <a name="host-in-aspnet-core"></a>在 ASP.NET Core 中代管

.NET 應用程式會設定並啟動「主機」。 主機負責應用程式啟動和存留期管理。 提供兩種 API 可供使用：

* [Web 代管](xref:fundamentals/host/web-host) &ndash; 適合代管 Web 應用程式。
* [一般代管](xref:fundamentals/host/generic-host) (ASP .NET Core 2.1 或更新版本) &ndash; 適合代管非 Web 應用程式 (例如，執行背景工作的應用程式)。 在後續版本中，一般代管將適用於代管任何種類的應用程式，包括 Web 應用程式。 一般代管最後將會取代 Web 代管。

若要裝載 ASP.NET Core「Web 應用程式」，開發人員應該使用以 <xref:Microsoft.AspNetCore.Hosting.IWebHostBuilder> 為基礎的 Web 主機。 若要裝載 ASP.NET Core「非 Web 應用程式」，開發人員應使用以 <xref:Microsoft.Extensions.Hosting.HostBuilder> 為基礎的一般主機。

<xref:fundamentals/host/hosted-services>  
了解如何在 ASP.NET Core 中使用託管服務實作背景工作。

<xref:fundamentals/configuration/platform-specific-configuration>  
了解如何使用 <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> 實作，從參考或非參考的組件增強 ASP.NET Core 應用程式。
