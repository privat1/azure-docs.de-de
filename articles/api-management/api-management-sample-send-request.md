---
title: "Verwenden des API Management-Diensts zum Generieren von HTTP-Anforderungen"
description: "Erfahren Sie mehr über die Verwendung von Anforderungs- und Antwortrichtlinien in API Management, um externe Dienste aus Ihrer API abzurufen."
services: api-management
documentationcenter: 
author: vladvino
manager: erikre
editor: 
ms.assetid: 4539c0fa-21ef-4b1c-a1d4-d89a38c242fa
ms.service: api-management
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/15/2016
ms.author: apimpm
ms.openlocfilehash: 6b7f1268ea4893307713931e7288f5d38c5ee080
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 10/11/2017
---
# <a name="using-external-services-from-the-azure-api-management-service"></a>Verwenden externer Dienste über den Azure API Management-Dienst
Mit den im Azure API Management-Dienst verfügbaren Richtlinien können Sie eine breite Palette sinnvoller Aufgaben durchführen, ausschließlich basierend auf der eingehenden Anforderung, der ausgehenden Antwort und den grundlegenden Konfigurationsinformationen. Die Interaktionsfähigkeit mit externen Diensten mithilfe von API Management-Richtlinien eröffnet jedoch viele weitere Möglichkeiten.

Wir haben bereits gesehen, wie wir mit dem [Azure Event Hub-Dienst für die Protokollierung, Überwachung und Analyse](api-management-log-to-eventhub-sample.md)interagieren können. In diesem Artikel werden Richtlinien gezeigt, die die Interaktion mit externen HTTP-basierten Diensten ermöglichen. Diese Richtlinien können zum Auslösen von Remoteereignissen oder zum Abrufen von Informationen verwendet werden, die wiederum zum Verändern der ursprünglichen Anforderung und Antwort verwendet werden.

## <a name="send-one-way-request"></a>Send-One-Way-Request
Die einfachste externe Interaktion ist möglicherweise eine Anforderung im „Fire and Forget“-Stil (Auslösen und Vergessen), mit der ein externer Dienst über ein wichtiges Ereignis jeglicher Art benachrichtigt werden kann. Wir können die Ablaufsteuerungsrichtlinie `choose` verwenden, um beliebige interessante Bedingungen zu erkennen. Wenn die Bedingung erfüllt ist, können wir mithilfe der [send-one-way-request](https://msdn.microsoft.com/library/azure/dn894085.aspx#SendOneWayRequest)-Richtlinie eine externe HTTP-Anforderung ausführen. Dabei kann es sich um eine Anforderung an ein Messagingsystem wie Hipchat oder Slack oder an eine E-Mail-API wie SendGrid oder MailChimp handeln. Alternativ können für kritische Supportfälle Anforderungen an beispielsweise PagerDuty gesendet werden. Alle diese Messagingsysteme verfügen über einfache HTTP-APIs, die wir mühelos aufrufen können.

### <a name="alerting-with-slack"></a>Benachrichtigung mit Slack
Im folgenden Beispiel wird gezeigt, wie eine Nachricht an einen Slack-Chatraum gesendet wird, wenn der Statuscode der HTTP-Antwort größer als oder gleich 500 ist. Ein 500-Bereichsfehler deutet auf ein Problem mit unserer Back-End-API hin, den der API-Client nicht selbst beheben kann. In der Regel müssen wir auf die eine oder andere Weise eingreifen.  

```xml
<choose>
    <when condition="@(context.Response.StatusCode >= 500)">
      <send-one-way-request mode="new">
        <set-url>https://hooks.slack.com/services/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</set-url>
        <set-method>POST</set-method>
        <set-body>@{
                return new JObject(
                        new JProperty("username","APIM Alert"),
                        new JProperty("icon_emoji", ":ghost:"),
                        new JProperty("text", String.Format("{0} {1}\nHost: {2}\n{3} {4}\n User: {5}",
                                                context.Request.Method,
                                                context.Request.Url.Path + context.Request.Url.QueryString,
                                                context.Request.Url.Host,
                                                context.Response.StatusCode,
                                                context.Response.StatusReason,
                                                context.User.Email
                                                ))
                        ).ToString();
            }</set-body>
      </send-one-way-request>
    </when>
</choose>
```

Bei Slack kommen eingehende Webhooks zum Einsatz. Beim Konfigurieren eines eingehenden Webhook generiert Slack eine besondere URL, mit der Sie eine einfache POST-Anforderung tätigen und eine Nachricht an den Slack-Kanal übergeben können. Der erstellte JSON-Text basiert auf einem von Slack definierten Format.

![Slack-Webhook](./media/api-management-sample-send-request/api-management-slack-webhook.png)

### <a name="is-fire-and-forget-good-enough"></a>Reicht „Fire and Forget“ aus?
Es gibt einige Nachteile bei einer Anforderung im Fire-and-Forget-Stil. Wenn die Anforderung aus irgendeinem Grund fehlschlägt, wird der Fehler nicht gemeldet. In einer solchen Situation sind die Komplexität eines sekundären Fehlermeldesystems und die zusätzlichen Leistungseinbußen aufgrund der Antwortwartezeiten nicht gerechtfertigt. Bei Szenarien, in denen Sie die Antwort unbedingt überprüfen müssen, ist die [send-request](https://msdn.microsoft.com/library/azure/dn894085.aspx#SendRequest) -Richtlinie die bessere Option.

## <a name="send-request"></a>send-request
Die `send-request` -Richtlinie ermöglicht die Nutzung eines externen Diensts zum Durchführen komplexer Verarbeitungsfunktionen und zum Zurückgeben von Daten an den API Management-Dienst, der für die weitere Richtlinienverarbeitung verwendet werden kann.

### <a name="authorizing-reference-tokens"></a>Autorisieren von Verweistoken
Eine wichtige Funktion von API Management ist der Schutz der Back-End-Ressourcen. Wenn der von Ihrer API verwendete Autorisierungsserver [JWT-Token](http://jwt.io/) als Teil des OAuth2-Ablaufs erstellt, wie dies bei [Azure Active Directory](../active-directory/active-directory-aadconnect.md) der Fall ist, dann können Sie mithilfe der `validate-jwt`-Richtlinie die Gültigkeit des Tokens überprüfen. Einige Autorisierungsserver erstellen jedoch sogenannte [Verweistoken](http://leastprivilege.com/2015/11/25/reference-tokens-and-introspection/) , die nicht ohne Rückruf des Autorisierungsservers überprüft werden können.

### <a name="standardized-introspection"></a>Standardisierte Introspection
In der Vergangenheit gab es keine standardisierte Methode zur Überprüfung eines Verweistokens mit einem Autorisierungsserver. Die IETF hat jedoch den vor kurzem vorgeschlagenen Standard [RFC 7662](https://tools.ietf.org/html/rfc7662) veröffentlicht, der definiert, wie ein Ressourcenserver die Gültigkeit eines Tokens überprüfen kann.

### <a name="extracting-the-token"></a>Extrahieren des Tokens
Der erste Schritt besteht im Extrahieren des Tokens aus dem Autorisierungsheader. Der Headerwert sollte mit dem `Bearer` -Autorisierungsschema, einem einzelnen Leerzeichen, gefolgt vom Autorisierungstoken gemäß [RFC 6750](http://tools.ietf.org/html/rfc6750#section-2.1)formatiert werden. Es gibt leider Fälle, in denen das Autorisierungsschema weggelassen wird. Um diesem Umstand bei der Analyse Rechnung zu tragen, wird der Headerwert mit einem Leerzeichen geteilt und die letzte Zeichenfolge aus dem zurückgegebenen Array von Zeichenfolgen ausgewählt. Dies ist eine mögliche Lösung für falsch formatierte Autorisierungsheader.

```xml
<set-variable name="token" value="@(context.Request.Headers.GetValueOrDefault("Authorization","scheme param").Split(' ').Last())" />
```

### <a name="making-the-validation-request"></a>Ausführen der Überprüfungsanforderung
Nach Erhalt des Autorisierungstokens können wir die Anforderung ausführen, um das Token zu überprüfen. In RFC 7662 wird dieser Prozess „Introspection“ genannt. Dazu müssen Sie mittels `POST` ein HTML-Formular in der Introspection-Ressource veröffentlichen. Das HTML-Formular muss mindestens ein Schlüssel/Wert-Paar mit dem Schlüssel `token` enthalten. Diese Anforderung an den Autorisierungsserver muss ebenfalls authentifiziert werden, um sicherzustellen, dass bösartige Clients nicht nach gültigen Token fischen können.

```xml
<send-request mode="new" response-variable-name="tokenstate" timeout="20" ignore-error="true">
  <set-url>https://microsoft-apiappec990ad4c76641c6aea22f566efc5a4e.azurewebsites.net/introspection</set-url>
  <set-method>POST</set-method>
  <set-header name="Authorization" exists-action="override">
    <value>basic dXNlcm5hbWU6cGFzc3dvcmQ=</value>
  </set-header>
  <set-header name="Content-Type" exists-action="override">
    <value>application/x-www-form-urlencoded</value>
  </set-header>
  <set-body>@($"token={(string)context.Variables["token"]}")</set-body>
</send-request>
```

### <a name="checking-the-response"></a>Überprüfen der Antwort
Das `response-variable-name` -Attribut wird verwendet, um auf die zurückgegebene Antwort zuzugreifen. Der in dieser Eigenschaft definierte Name kann als Schlüssel für das `context.Variables`-Wörterbuch verwendet werden, um auf das `IResponse`-Objekt zuzugreifen.

Aus dem Antwortobjekt können wir den Text abrufen. RFC 7622 legt fest, dass die Antwort ein JSON-Objekt sein muss und mindestens eine Eigenschaft namens `active` enthalten muss, bei der es sich um einen booleschen Wert handelt. Wenn `active` true ist, dann wird das Token als gültig erachtet.

### <a name="reporting-failure"></a>Melden eines Fehlers
Wir verwenden eine `<choose>`-Richtlinie, um zu ermitteln, ob das Token ungültig ist. Wenn dies der Fall ist, wird eine 401-Antwort zurückgegeben.

```xml
<choose>
  <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
    <return-response response-variable-name="existing response variable">
      <set-status code="401" reason="Unauthorized" />
      <set-header name="WWW-Authenticate" exists-action="override">
        <value>Bearer error="invalid_token"</value>
      </set-header>
    </return-response>
  </when>
</choose>
```

Gemäß [RFC 6750](https://tools.ietf.org/html/rfc6750#section-3), in der die Verwendung von `bearer`-Token beschrieben wird, wird auch ein `WWW-Authenticate`-Header mit der 401-Antwort zurückgegeben. WWW-Authenticate soll einen Client anweisen, wie eine ordnungsgemäß autorisierte Anforderung erstellt wird. Aufgrund der vielfältigen Ansätze, die beim OAuth2-Framework möglich sind, ist es schwierig, alle erforderlichen Informationen mitzuteilen. Glücklicherweise gibt es derzeit Bemühungen, die [Clients dabei helfen sollen, herauszufinden, wie Anforderungen an einen Ressourcenserver ordnungsgemäß autorisiert werden](http://tools.ietf.org/html/draft-jones-oauth-discovery-00).

### <a name="final-solution"></a>Endgültige Lösung
Nach Berücksichtigung all dieser Aspekte erhalten wir die folgende Richtlinie:

```xml
<inbound>
  <!-- Extract Token from Authorization header parameter -->
  <set-variable name="token" value="@(context.Request.Headers.GetValueOrDefault("Authorization","scheme param").Split(' ').Last())" />

  <!-- Send request to Token Server to validate token (see RFC 7662) -->
  <send-request mode="new" response-variable-name="tokenstate" timeout="20" ignore-error="true">
    <set-url>https://microsoft-apiappec990ad4c76641c6aea22f566efc5a4e.azurewebsites.net/introspection</set-url>
    <set-method>POST</set-method>
    <set-header name="Authorization" exists-action="override">
      <value>basic dXNlcm5hbWU6cGFzc3dvcmQ=</value>
    </set-header>
    <set-header name="Content-Type" exists-action="override">
      <value>application/x-www-form-urlencoded</value>
    </set-header>
    <set-body>@($"token={(string)context.Variables["token"]}")</set-body>
  </send-request>

  <choose>
          <!-- Check active property in response -->
          <when condition="@((bool)((IResponse)context.Variables["tokenstate"]).Body.As<JObject>()["active"] == false)">
              <!-- Return 401 Unauthorized with http-problem payload -->
              <return-response response-variable-name="existing response variable">
                  <set-status code="401" reason="Unauthorized" />
                  <set-header name="WWW-Authenticate" exists-action="override">
                      <value>Bearer error="invalid_token"</value>
                  </set-header>
              </return-response>
          </when>
      </choose>
  <base />
</inbound>
```

Dies ist nur eines von vielen Beispielen, wie die `send-request` -Richtlinie verwendet werden kann, um nützliche externe Dienste in den Anforderungs- und Antwortprozess zu integrieren, der den API Management-Dienst durchläuft.

## <a name="response-composition"></a>Antwortkomposition
Mit der `send-request`-Richtlinie können Sie eine primäre Anforderung an ein Back-End-System verbessern, wie im vorherigen Beispiel gezeigt. Sie kann auch als vollständiger Ersatz für den Back-End-Aufruf verwendet werden. Mithilfe dieser Technik können wir mühelos zusammengesetzte Ressourcen erstellen, die aus verschiedenen Systemen zusammengestellt werden.

### <a name="building-a-dashboard"></a>Erstellen eines Dashboards
Gelegentlich möchten Sie in der Lage sein, Informationen aus mehreren Back-End-Systemen verfügbar zu machen, um beispielsweise ein Dashboard zu steuern. Die KPIs stammen von komplett unterschiedlichen Back-Ends, aber Sie ziehen es vor, keinen direkten Zugriff auf diese bereitzustellen. Es wäre schön, wenn alle Informationen in einer einzelnen Anforderung abgerufen werden könnten. Einige der Back-End-Informationen müssen womöglich aufgegliedert und zunächst etwas bereinigt werden! Eine Zwischenspeicherung dieser zusammengesetzten Ressource wäre hilfreich für die Reduzierung der Back-End-Last, da Benutzer die Gewohnheit haben, mehrfach die F5-Taste zu drücken, um festzustellen, ob sich die schwachen Metriken ändern.    

### <a name="faking-the-resource"></a>Falsche Ressource
Der erste Schritt beim Erstellen der Dashboardressource ist die Konfiguration eines neuen Vorgangs im API Management-Herausgeberportal. Dabei handelt es sich um einen Platzhaltervorgang, der zum Konfigurieren der Kompositionsrichtlinie verwendet wird, um die dynamische Ressource zu erstellen.

![Dashboardvorgang](./media/api-management-sample-send-request/api-management-dashboard-operation.png)

### <a name="making-the-requests"></a>Durchführen der Anforderungen
Sobald der `dashboard` -Vorgang erstellt wurde, können wir eine Richtlinie speziell für diesen Vorgang konfigurieren. 

![Dashboardvorgang](./media/api-management-sample-send-request/api-management-dashboard-policy.png)

Der erste Schritt besteht im Extrahieren von Abfrageparametern aus der eingehenden Anforderung, damit wir sie an das Back-End weiterleiten können. In diesem Beispiel zeigt das Dashboard Informationen basierend auf einem Zeitraum. Es verfügt daher über einen `fromDate`- und einen `toDate`-Parameter. Wir können die `set-variable`-Richtlinie verwenden, um die Informationen aus der Anforderungs-URL zu extrahieren.

```xml
<set-variable name="fromDate" value="@(context.Request.Url.Query["fromDate"].Last())">
<set-variable name="toDate" value="@(context.Request.Url.Query["toDate"].Last())">
```

Sobald wir diese Informationen haben, können wir Anforderungen an alle Back-End-Systeme durchführen. Jede Anforderung erstellt eine neue URL mit den Parameterinformationen, ruft den entsprechenden Server auf und speichert die Antwort in einer Kontextvariablen.

```xml
<send-request mode="new" response-variable-name="revenuedata" timeout="20" ignore-error="true">
  <set-url>@($"https://accounting.acme.com/salesdata?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>

<send-request mode="new" response-variable-name="materialdata" timeout="20" ignore-error="true">
  <set-url>@($"https://inventory.acme.com/materiallevels?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>

<send-request mode="new" response-variable-name="throughputdata" timeout="20" ignore-error="true">
<set-url>@($"https://production.acme.com/throughput?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>

<send-request mode="new" response-variable-name="accidentdata" timeout="20" ignore-error="true">
<set-url>@($"https://production.acme.com/throughput?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
  <set-method>GET</set-method>
</send-request>
```

Diese Anforderungen werden nacheinander ausgeführt, was nicht ideal ist. In einer zukünftigen Version wird eine neue Richtlinie namens `wait` eingeführt, mit der alle diese Anforderungen parallel ausgeführt werden können.

### <a name="responding"></a>Antworten
Um die zusammengesetzte Antwort zu generieren, kann die [return-response](https://msdn.microsoft.com/library/azure/dn894085.aspx#ReturnResponse) -Richtlinie verwendet werden. Das `set-body`-Element kann einen Ausdruck verwenden, um ein neues `JObject` mit allen Komponentendarstellungen zu erstellen, die als Eigenschaften eingebettet sind.

```xml
<return-response response-variable-name="existing response variable">
  <set-status code="200" reason="OK" />
  <set-header name="Content-Type" exists-action="override">
    <value>application/json</value>
  </set-header>
  <set-body>
    @(new JObject(new JProperty("revenuedata",((IResponse)context.Variables["revenuedata"]).Body.As<JObject>()),
                  new JProperty("materialdata",((IResponse)context.Variables["materialdata"]).Body.As<JObject>()),
                  new JProperty("throughputdata",((IResponse)context.Variables["throughputdata"]).Body.As<JObject>()),
                  new JProperty("accidentdata",((IResponse)context.Variables["accidentdata"]).Body.As<JObject>())
                  ).ToString())
  </set-body>
</return-response>
```

Die vollständige Richtlinie sieht folgendermaßen aus:

```xml
<policies>
    <inbound>

  <set-variable name="fromDate" value="@(context.Request.Url.Query["fromDate"].Last())">
  <set-variable name="toDate" value="@(context.Request.Url.Query["toDate"].Last())">

    <send-request mode="new" response-variable-name="revenuedata" timeout="20" ignore-error="true">
      <set-url>@($"https://accounting.acme.com/salesdata?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <send-request mode="new" response-variable-name="materialdata" timeout="20" ignore-error="true">
      <set-url>@($"https://inventory.acme.com/materiallevels?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <send-request mode="new" response-variable-name="throughputdata" timeout="20" ignore-error="true">
    <set-url>@($"https://production.acme.com/throughput?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <send-request mode="new" response-variable-name="accidentdata" timeout="20" ignore-error="true">
    <set-url>@($"https://production.acme.com/throughput?from={(string)context.Variables["fromDate"]}&to={(string)context.Variables["fromDate"]}")"</set-url>
      <set-method>GET</set-method>
    </send-request>

    <return-response response-variable-name="existing response variable">
      <set-status code="200" reason="OK" />
      <set-header name="Content-Type" exists-action="override">
        <value>application/json</value>
      </set-header>
      <set-body>
        @(new JObject(new JProperty("revenuedata",((IResponse)context.Variables["revenuedata"]).Body.As<JObject>()),
                      new JProperty("materialdata",((IResponse)context.Variables["materialdata"]).Body.As<JObject>()),
                      new JProperty("throughputdata",((IResponse)context.Variables["throughputdata"]).Body.As<JObject>()),
                      new JProperty("accidentdata",((IResponse)context.Variables["accidentdata"]).Body.As<JObject>())
                      ).ToString())
      </set-body>
    </return-response>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
</policies>
```

Bei der Konfiguration des Platzhaltervorgangs kann die Dashboardressource so konfiguriert werden, dass sie mindestens eine Stunde lang zwischengespeichert wird. Da wir das Wesen der Daten kennen und verstehen, sind sie auch dann  weiterhin hinreichend aussagekräftig, um wertvolle Informationen für die Benutzer zu übermitteln, wenn sie schon eine Stunde veraltet sind.

## <a name="summary"></a>Zusammenfassung
Der Azure API Management-Dienst bietet flexible Richtlinien, die selektiv auf den HTTP-Verkehr angewendet werden können und die Zusammensetzung der Back-End-Dienste ermöglichen. Unabhängig davon, ob Sie das API-Gateway mit Alarmfunktionen, Überprüfungs- und Validierungsfunktionen verbessern möchten oder ob Sie neue zusammengesetzte Ressourcen basierend auf mehreren Back-End-Diensten erstellen möchten, bieten die `send-request` -Richtlinie und verwandte Richtlinien vollkommen neue Möglichkeiten.

## <a name="watch-a-video-overview-of-these-policies"></a>Überblicksvideo zu diesen Richtlinien
Informieren Sie sich im folgenden Video ausführlicher über die in diesem Artikel behandelten Richtlinien [send-one-way-request](https://msdn.microsoft.com/library/azure/dn894085.aspx#SendOneWayRequest), [send-request](https://msdn.microsoft.com/library/azure/dn894085.aspx#SendRequest) und [return-response](https://msdn.microsoft.com/library/azure/dn894085.aspx#ReturnResponse).

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Send-Request-and-Return-Response-Policies/player]
> 
> 

