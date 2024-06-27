---
title: „[!DNL Asset Compute Service] HTTP-API“
description: „[!DNL Asset Compute Service] HTTP-API zum Erstellen benutzerdefinierter Programme“.
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 64%

---

# [!DNL Asset Compute Service]-HTTP-API {#asset-compute-http-api}

Die Verwendung der API ist auf Entwicklungszwecke beschränkt. Die API wird bei der Entwicklung benutzerdefinierter Programme als Kontext bereitgestellt. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] verwendet die API, um die Verarbeitungsinformationen an ein benutzerdefiniertes Programm zu übergeben. Weitere Informationen finden Sie unter [Verwenden von Asset-Microservices und Verarbeitungsprofilen](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] ist nur zur Verwendung mit [!DNL Experience Manager] as a [!DNL Cloud Service] verfügbar.

Jeder Client der [!DNL Asset Compute Service]-HTTP-API muss diesem allgemeinen Ablauf folgen:

1. Ein Client wird als [!DNL Adobe Developer Console] Projekt in einer IMS-Organisation. Jeder separate Client (System oder Umgebung) benötigt ein eigenes Projekt, um den Ereignisdatenfluss zu trennen.

1. Ein Client generiert mithilfe der [JWT (Service Account)-Authentifizierung](https://developer.adobe.com/developer-console/docs/guides/) ein Zugriffs-Token für das technische Konto.

1. Ein Client ruft [`/register`](#register) nur einmal auf, um die Journal-URL abzurufen.

1. Ein Client ruft [`/process`](#process-request) für jedes Asset auf, für das er Ausgabedarstellungen generieren möchte. Der Aufruf ist asynchron.

1. Ein Client fragt das Journal regelmäßig ab, um [Ereignisse zu empfangen](#asynchronous-events). Er empfängt Ereignisse für jede angeforderte Ausgabedarstellung, wenn die Ausgabedarstellung erfolgreich verarbeitet wurde (Ereignistyp `rendition_created`) oder wenn ein Fehler aufgetreten ist (Ereignistyp `rendition_failed`).

Die [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) -Modul vereinfacht die Verwendung der API im Code von Node.js.

## Authentifizierung und Autorisierung {#authentication-and-authorization}

Für alle APIs ist eine Zugriffs-Token-Authentifizierung erforderlich. Die Anfragen müssen die folgenden Kopfzeilen festlegen:

1. `Authorization`-Kopfzeile mit Träger-Token, dem technischen Konto-Token, das über den [JWT-Austausch](https://developer.adobe.com/developer-console/docs/guides/) vom Adobe Developer Console-Projekt empfangen wurde. Die [Bereiche](#scopes) sind nachfolgend beschrieben.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id`-Kopfzeile mit der IMS-Organisations-ID.

1. `x-api-key` mit der Client-ID des [!DNL Adobe Developers Console]-Projekts.

### Bereiche {#scopes}

Stellen Sie die folgenden Bereiche für das Zugriffs-Token sicher:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Diese Bereiche erfordern die [!DNL Adobe Developer Console] Projekt, für das ein Abonnement besteht `Asset Compute`, `I/O Events`, und `I/O Management API` Dienste. Die einzelnen Bereiche werden wie folgt unterteilt:

* Einfach
   * Bereiche: `openid,AdobeID`

* Asset Compute
   * Metascope: `asset_compute_meta`
   * Bereiche: `asset_compute,read_organizations`

* Adobe [!DNL `I/O Events`]
   * Metascope: `event_receiver_api`
   * Bereiche: `event_receiver,event_receiver_api`

* Adobe [!DNL `I/O Management API`]
   * Metascope: `ent_adobeio_sdk`
   * Bereiche: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registrierung {#register}

Jeder Client von [!DNL Asset Compute service] – ein eindeutiges [!DNL Adobe Developer Console]-Projekt, das den Service abonniert hat – muss sich [registrieren](#register-request), bevor er Verarbeitungsanfragen stellen kann. Der Registrierungsschritt gibt das eindeutige Ereignisjournal zurück, das zum Abrufen der asynchronen Ereignisse aus der Ausgabedarstellungsverarbeitung erforderlich ist.

Am Ende seines Lebenszyklus kann ein Client die [Registrierung](#unregister-request) aufheben.

### Registrierungsanfrage {#register-request}

Mit diesem API-Aufruf wird ein [!DNL Asset Compute]-Client eingerichtet und die Ereignisjournal-URL bereitgestellt. Dieser Prozess ist ein idempotent-Vorgang und muss nur einmal für jeden Client aufgerufen werden. Er kann erneut aufgerufen werden, um die Journal-URL abzurufen.

| Parameter | Wert |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pfad | `/register` |
| Kopfzeile `Authorization` | Alle [autorisierungsbezogenen Kopfzeilen](#authentication-and-authorization). |
| Kopfzeile `x-request-id` | Optional, festgelegt von Clients für eine eindeutige End-to-End-Kennung der Verarbeitungsanfragen systemübergreifend. |
| Hauptteil der Anfrage | Muss leer sein. |

### Registrierungsantwort {#register-response}

| Parameter | Wert |
|-----------------------|------------------------------------------------------|
| MIME-Typ | `application/json` |
| Kopfzeile `X-Request-Id` | Entweder identisch mit der Anfragekopfzeile `X-Request-Id` oder eine eindeutig erstellte. Verwendung zur systemübergreifenden Identifizierung von Anfragen, Support-Anfragen oder beides. |
| Hauptteil der Antwort | Ein JSON-Objekt mit `journal`, `ok`oder `requestId` -Felder. |

Die HTTP-Status-Codes lauten:

* **200 Erfolg**: Bei erfolgreicher Anfrage. Die `journal` URL empfängt Benachrichtigungen über die Ergebnisse der asynchronen Verarbeitung, die über `/process`. Sie gibt `rendition_created` Ereignisse nach erfolgreichem Abschluss oder `rendition_failed` -Ereignisse, wenn der Prozess fehlschlägt.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Nicht autorisiert**: Tritt auf, wenn die Anfrage keine gültige [Authentifizierung](#authentication-and-authorization) aufweist. Ein Beispiel könnte ein ungültiges Zugriffs-Token oder ein ungültiger API-Schlüssel sein.

* **403 Verboten**: Tritt auf, wenn die Anfrage keine gültige [Autorisierung](#authentication-and-authorization) aufweist. Beispielsweise könnte ein gültiges Zugriffs-Token vorliegen, aber das Adobe Developer Console-Projekt (technisches Konto) hat nicht alle erforderlichen Services abonniert.

* **429 Zu viele Anfragen**: Tritt auf, wenn dieser Client oder anderweitig das System überlädt. Clients sollten es mit einem [exponentiellen Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) erneut versuchen. Der Hauptteil ist leer.
* **4xx-Fehler**: Wenn ein anderer Client-Fehler aufgetreten ist und die Registrierung fehlgeschlagen ist. Normalerweise wird eine JSON-Antwort wie diese zurückgegeben, obwohl dies nicht für alle Fehler garantiert ist:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-Fehler**: Tritt auf, wenn ein anderer Server-seitiger Fehler aufgetreten ist und die Registrierung fehlgeschlagen ist. Normalerweise wird eine JSON-Antwort wie diese zurückgegeben, obwohl dies nicht für alle Fehler garantiert ist:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Anfrage zum Aufheben der Registrierung {#unregister-request}

Durch diesen API-Aufruf wird die Registrierung eines [!DNL Asset Compute]-Clients aufgehoben. Nach dieser Aufhebung der Registrierung ist es nicht mehr möglich, `/process`. Wenn Sie den API-Aufruf für einen nicht registrierten Client oder einen noch nicht registrierten Client verwenden, wird ein `404`-Fehler zurückgegeben.

| Parameter | Wert |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pfad | `/unregister` |
| Kopfzeile `Authorization` | Alle [autorisierungsbezogenen Kopfzeilen](#authentication-and-authorization). |
| Kopfzeile `x-request-id` | Optional. Clients können sie für eine eindeutige End-to-End-Kennung der Verarbeitungsanfragen systemübergreifend festlegen. |
| Hauptteil der Anfrage | Leer. |

### Antwort zum Aufheben der Registrierung {#unregister-response}

| Parameter | Wert |
|-----------------------|------------------------------------------------------|
| MIME-Typ | `application/json` |
| Kopfzeile `X-Request-Id` | Entweder identisch mit der Anfragekopfzeile `X-Request-Id` oder eine eindeutig erstellte. Verwendung zur systemübergreifenden Identifizierung von Anfragen oder Support-Anfragen. |
| Hauptteil der Antwort | Ein JSON-Objekt mit den Feldern `ok` und `requestId`. |

Die Status-Codes sind:

* **200 Erfolg**: Tritt auf, wenn die Registrierung und das Journal gefunden und entfernt wurden.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 Nicht autorisiert**: Tritt auf, wenn die Anfrage keine gültige [Authentifizierung](#authentication-and-authorization) aufweist. Ein Beispiel könnte ein ungültiges Zugriffs-Token oder ein ungültiger API-Schlüssel sein.

* **403 Verboten**: Tritt auf, wenn die Anfrage keine gültige [Autorisierung](#authentication-and-authorization) aufweist. Beispielsweise könnte ein gültiges Zugriffs-Token vorliegen, aber das Adobe Developer Console-Projekt (technisches Konto) hat nicht alle erforderlichen Services abonniert.

* **404 Nicht gefunden**: Dieser Status wird angezeigt, wenn die angegebenen Anmeldeinformationen abgemeldet oder ungültig sind.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Zu viele Anfragen**: Tritt auf, wenn das System überlastet ist. Clients sollten es mit einem [exponentiellen Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) erneut versuchen. Der Hauptteil ist leer.

* **4xx-Fehler**: Tritt auf, wenn ein anderer Client-Fehler aufgetreten ist und die Aufhebung der Registrierung fehlgeschlagen ist. Normalerweise wird eine JSON-Antwort wie diese zurückgegeben, obwohl dies nicht für alle Fehler garantiert ist:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-Fehler**: Tritt auf, wenn ein anderer Server-seitiger Fehler aufgetreten ist und die Registrierung fehlgeschlagen ist. Normalerweise wird eine JSON-Antwort wie diese zurückgegeben, obwohl dies nicht für alle Fehler garantiert ist:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Verarbeitungsanfrage {#process-request}

Der `process`-Vorgang sendet einen Vorgang, der ein Quell-Asset basierend auf den Anweisungen in der Anfrage in mehrere Ausgabedarstellungen umwandelt. Benachrichtigungen über den erfolgreichen Abschluss (Ereignistyp) `rendition_created`) oder etwaige Fehler (Ereignistyp) `rendition_failed`) an ein Ereignisjournal gesendet werden, das abgerufen werden muss mit [`/register`](#register) einmal vor einer beliebigen Anzahl von `/process` -Anfragen. Fehlerhaft gebildete Anfragen schlagen sofort mit einem 400-Fehler-Code fehl.

Binärdateien werden mithilfe von URLs referenziert, z. B. von Amazon AWS S3 vorsignierte URLs oder Azure Blob Storage-SAS-URLs. Wird sowohl zum Lesen der `source` Asset (`GET` URLs) und Schreiben der Ausgabedarstellungen (`PUT` URLs). Der Client ist für die Generierung dieser vorsignierten URLs verantwortlich.

| Parameter | Wert |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pfad | `/process` |
| MIME-Typ | `application/json` |
| Kopfzeile `Authorization` | Alle [autorisierungsbezogenen Kopfzeilen](#authentication-and-authorization). |
| Kopfzeile `x-request-id` | Optional. Clients können eine eindeutige End-to-End-Kennung festlegen, um Verarbeitungsanfragen systemübergreifend zu verfolgen. |
| Hauptteil der Anfrage | Sie muss sich im JSON-Format der Verarbeitungsanfrage befinden, wie unten beschrieben. Es enthält Anweisungen dazu, welches Asset verarbeitet und welche Ausgabedarstellungen generiert werden sollen. |

### JSON der Verarbeitungsanfrage {#process-request-json}

Der Hauptteil der `/process`-Anfrage ist ein JSON-Objekt mit diesem allgemeinen Schema:

```json
{
    "source": "",
    "renditions" : []
}
```

Die folgenden Felder sind verfügbar:

| Name | Typ | Beschreibung | Beispiel |
|--------------|----------|-------------|---------|
| `source` | `string` | URL des verarbeiteten Quell-Assets Optional, basierend auf dem angeforderten Ausgabeformat (z. B. `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Beschreibung des verarbeiteten Quell-Assets. Siehe Beschreibung der [Quellobjektfelder](#source-object-fields) unten. Optional, basierend auf dem angeforderten Ausgabeformat (z. B. `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Aus der Quelldatei zu generierende Ausgabedarstellungen. Jedes Ausgabedarstellungsobjekt unterstützt eine [Ausgabedarstellungsanweisung](#rendition-instructions). Erforderlich. | `[{ "target": "https://....", "fmt": "png" }]` |

`source` kann entweder ein `<string>` sein, der als URL erkannt wird, oder ein `<object>` mit einem zusätzlichen Feld. Die folgenden Varianten sind ähnlich:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Quellobjektfelder {#source-object-fields}

| Name | Typ | Beschreibung | Beispiel |
|-----------|----------|-------------|---------|
| `url` | `string` | URL des zu verarbeitenden Quell-Assets. Erforderlich. | `"http://example.com/image.jpg"` |
| `name` | `string` | Name der Quell-Asset-Datei. Wenn kein MIME-Typ erkannt wird, kann eine Dateierweiterung im Namen verwendet werden. Sie hat Vorrang vor dem Dateinamen, der im URL-Pfad angegeben ist. Außerdem hat sie Vorrang vor dem Dateinamen im `content-disposition` -Kopfzeile der binären Ressource. Der Standardwert ist &quot;file&quot;. | `"image.jpg"` |
| `size` | `number` | Größe der Quell-Asset-Datei in Byte. Hat Vorrang vor der `content-length`-Kopfzeile der binären Ressource. | `10234` |
| `mimetype` | `string` | MIME-Typ der Quell-Asset-Datei. Hat Vorrang vor der `content-type`-Kopfzeile der binären Ressource. | `"image/jpeg"` |

### Beispiel einer vollständigen `process`-Anfrage {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Verarbeitungsantwort {#process-response}

Die `/process`-Anfrage wird sofort mit einem Erfolg oder Fehler basierend auf der grundlegenden Anfragenvalidierung zurückgegeben. Die tatsächliche Asset-Verarbeitung erfolgt asynchron.

| Parameter | Wert |
|-----------------------|------------------------------------------------------|
| MIME-Typ | `application/json` |
| Kopfzeile `X-Request-Id` | Entweder identisch mit der Anfragekopfzeile `X-Request-Id` oder eine eindeutig erstellte. Verwendung zur systemübergreifenden Identifizierung von Anfragen oder Support-Anfragen. |
| Hauptteil der Antwort | Ein JSON-Objekt mit den Feldern `ok` und `requestId`. |

Status-Codes:

* **200 Erfolg**: Wenn die Anfrage erfolgreich gesendet wurde. Die Antwort-JSON enthält `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Ungültige Anfrage**: Wenn die Anfrage falsch strukturiert ist, z. B. wenn es in der JSON-Payload nicht die erforderlichen Felder gibt. Die Antwort-JSON enthält `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Nicht autorisiert**: Wenn die Anfrage keine gültige [Authentifizierung](#authentication-and-authorization) aufweist. Ein Beispiel könnte ein ungültiges Zugriffs-Token oder ein ungültiger API-Schlüssel sein.
* **403 Verboten**: Wenn die Anfrage keine gültige [Autorisierung](#authentication-and-authorization) aufweist. Beispielsweise könnte ein gültiges Zugriffs-Token vorliegen, aber das Adobe Developer Console-Projekt (technisches Konto) hat nicht alle erforderlichen Services abonniert.
* **429 Zu viele Anfragen**: Tritt auf, wenn das System überlastet ist, entweder aufgrund dieses bestimmten Clients oder aufgrund der Gesamtnachfrage. Die Clients können es mit einem [exponentiellen Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) erneut versuchen. Der Hauptteil ist leer.
* **4xx-Fehler**: Wenn ein anderer Client-Fehler aufgetreten ist. Normalerweise wird eine JSON-Antwort wie diese zurückgegeben, obwohl dies nicht für alle Fehler garantiert ist:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **5xx-Fehler**: Wenn ein anderer Server-seitiger Fehler aufgetreten ist. Normalerweise wird eine JSON-Antwort wie diese zurückgegeben, obwohl dies nicht für alle Fehler garantiert ist:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

Die meisten Clients neigen wahrscheinlich dazu, dieselbe Anfrage mit [exponentieller Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) bei jedem Fehler *Außer* Konfigurationsprobleme wie 401 oder 403 oder ungültige Anforderungen wie 400. Abgesehen von der regulären Ratenbegrenzung über 429 Antworten kann ein vorübergehender Dienstausfall oder eine vorübergehende Service-Beschränkung zu 5xx-Fehlern führen. Es wäre dann ratsam, es nach einer gewissen Zeit erneut zu versuchen.

Alle JSON-Antworten (sofern vorhanden) enthalten die `requestId`, der denselben Wert wie der `X-Request-Id` -Kopfzeile. Adobe empfiehlt, aus der Kopfzeile zu lesen, da sie immer vorhanden ist. Die `requestId` wird auch in allen Ereignissen, die mit Verarbeitungsanfragen zusammenhängen, als `requestId` zurückgegeben. Clients dürfen keine Annahmen über das Format dieser Zeichenfolge treffen. Es handelt sich um eine undurchsichtige Zeichenfolgenkennung.

## Aktivieren der Nachbearbeitung {#opt-in-to-post-processing}

Das [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) unterstützt eine Reihe grundlegender Optionen für die Nachbearbeitung von Bildern. Benutzerdefinierte Sekundärprogramme können sich explizit für die Nachbearbeitung entscheiden, indem sie das Feld `postProcess` im Ausgabedarstellungsobjekt auf `true` setzen.

Folgende Anwendungsfälle werden unterstützt:

* &quot;Zuschneiden&quot;ist eine Ausgabedarstellung für ein Rechteck, dessen Beschränkungen nach &quot;crop.w&quot;, &quot;crop.h&quot;, &quot;crop.x&quot;und &quot;crop.y&quot;definiert sind. Die Beschneidungsdetails werden im Ausgabedarstellungsobjekt angegeben. `instructions.crop` -Feld.
* Ändern der Bildgröße unter Verwendung von Breite, Höhe oder beidem. Die `instructions.width` und `instructions.height` definiert es im Ausgabedarstellungsobjekt. Um die Größe nur unter Verwendung von Breite oder Höhe zu ändern, legen Sie nur einen Wert fest. Compute Service behält das Seitenverhältnis bei.
* Festlegen der Qualität für ein JPEG-Bild. Die `instructions.quality` definiert es im Ausgabedarstellungsobjekt. Ein Qualitätsniveau von 100 steht für die höchste Qualität, während niedrigere Zahlen einen Qualitätsverlust bedeuten.
* Erstellen von Zwischenzeilenbildern. Die `instructions.interlace` definiert es im Ausgabedarstellungsobjekt.
* Stellen Sie die DPI so ein, dass die gerenderte Größe für Desktop-Publishing-Zwecke angepasst wird, indem Sie die auf die Pixel angewendete Skalierung anpassen. Die `instructions.dpi` definiert es im Ausgabedarstellungsobjekt, um die DPI-Auflösung zu ändern. Verwenden Sie die `convertToDpi`-Anweisungen, um die Größe des Bildes so zu ändern, dass es bei einer anderen Auflösung dieselbe Größe hat.
* Ändern Sie die Größe des Bildes so, dass seine gerenderte Breite oder Höhe bei der angegebenen Zielauflösung (DPI) mit dem Original übereinstimmt. Die `instructions.convertToDpi` definiert es im Ausgabedarstellungsobjekt.

## Wasserzeichen-Assets {#add-watermark}

Das [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) unterstützt das Hinzufügen eines Wasserzeichens zu PNG-, JPEG-, TIFF- und GIF-Bilddateien. Das Wasserzeichen wird der Ausgabedarstellung entsprechend den Ausgabedarstellungsanweisungen im `watermark`-Objekt hinzugefügt.

Das Wasserzeichen wird während der Nachbearbeitung der Ausgabedarstellung hinzugefügt. Um Assets mit Wasserzeichen zu versehen, [aktiviert das benutzerdefinierte Sekundärprogramm die Nachbearbeitung](#opt-in-to-post-processing), indem es das Feld `postProcess` im Ausgabedarstellungsobjekt auf `true` setzt. Wenn sich der Worker nicht anmeldet, wird kein Wasserzeichen angewendet, auch wenn das Wasserzeichenobjekt auf das Ausgabedarstellungsobjekt in der Anforderung festgelegt ist.

## Ausgabedarstellungsanweisungen {#rendition-instructions}

Die folgenden Optionen stehen für die `renditions` Array in [`/process`](#process-request).

### Allgemeine Felder {#common-fields}

| Name | Typ | Beschreibung | Beispiel |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Das Zielformat der Ausgabedarstellungen kann auch `text` für die Textextraktion und `xmp` zum Extrahieren XMP Metadaten als XML. Siehe [Unterstützte Formate](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support). | `png` |
| `worker` | `string` | URL eines [benutzerdefinierten Programms](develop-custom-application.md). Muss eine `https://`-URL sein. Wenn dieses Feld vorhanden ist, erstellt ein benutzerdefiniertes Programm die Ausgabedarstellung. Jedes andere festgelegte Ausgabedarstellungsfeld wird dann im benutzerdefinierten Programm verwendet. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Die URL, auf die die generierte Ausgabedarstellung mit HTTP-PUT hochgeladen werden soll. | `http://w.com/img.jpg` |
| `target` | `object` | Mehrteilige vorsignierte URL-Upload-Informationen für die generierte Ausgabedarstellung. Diese Informationen dienen [AEM/Oak - direkter binärer Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) mit diesem [Multipart-Upload-Verhalten](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Felder:<ul><li>`urls`: Array von Zeichenfolgen, eine für jede vorsignierte Teil-URL</li><li>`minPartSize`: die Mindestgröße für eine Teil-URL</li><li>`maxPartSize`: die Maximalgröße für eine Teil-URL</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optional. Der Client steuert den reservierten Bereich und übergibt ihn unverändert an Ausgabedarstellungsereignisse. Ermöglicht einem Client das Hinzufügen benutzerdefinierter Informationen zur Identifizierung von Ausgabedarstellungsereignissen. Sie darf nicht in benutzerdefinierten Programmen geändert oder verwendet werden, da Clients sie jederzeit ändern können. | `{ ... }` |

### Ausgabedarstellungsspezifische Felder {#rendition-specific-fields}

Eine Liste der derzeit unterstützten Dateiformate finden Sie unter [Unterstützte Dateiformate](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Name | Typ | Beschreibung | Beispiel |
|-------------------|----------|-------------|---------|
| `*` | `*` | Erweiterte, benutzerdefinierte Felder können hinzugefügt werden, die ein [benutzerdefiniertes Programm](develop-custom-application.md) versteht. | |
| `embedBinaryLimit` | `number` in Byte | Wenn die Dateigröße der Ausgabedarstellung kleiner als der angegebene Wert ist, wird sie in das Ereignis aufgenommen, das nach Abschluss der Erstellung gesendet wird. Die maximal zulässige Größe für die Einbettung beträgt 32 KB (32 x 1024 Byte). Wenn eine Ausgabedarstellung größer ist als die `embedBinaryLimit` -Beschränkung, wird sie an einem Speicherort im Cloud-Speicher abgelegt und nicht in das Ereignis eingebettet. | `3276` |
| `width` | `number` | Breite in Pixel. Nur für Bilddarstellungen. | `200` |
| `height` | `number` | Höhe in Pixel. Nur für Bilddarstellungen. | `200` |
|                   |          | Das Seitenverhältnis wird immer beibehalten, wenn: <ul> <li> Wenn sowohl `width` als auch `height` angegeben werden, wird die Bildgröße angepasst, wobei das Seitenverhältnis beibehalten wird. </li><li> Wenn nur `width` oder `height` angegeben ist, verwendet das resultierende Bild die entsprechende Dimension und behält dabei das Seitenverhältnis bei.</li><li> Wenn `width` oder `height` nicht angegeben ist, wird die Pixelgröße des Originalbilds verwendet. Diese hängt vom Quelltyp ab. Bei einigen Formaten, z. B. PDF-Dateien, wird eine Standardgröße verwendet. Es kann eine Maximalgröße geben.</li></ul> | |
| `quality` | `number` | Geben Sie die JPEG-Qualität im Bereich von `1` bis `100` an. Gilt nur für Bilddarstellungen. | `90` |
| `xmp` | `string` | Wird nur für das Zurückschreiben von XMP-Metadaten verwendet und ist base64-kodiertes XMP zum Zurückschreiben in die angegebene Ausgabedarstellung. | |
| `interlace` | `bool` | Erstellen Sie PNG- oder GIF-Zwischenzeilenbilder oder progressive JPEG-Bilder, indem Sie den Parameter auf `true` setzen. Hat keine Auswirkungen auf andere Dateiformate. | |
| `jpegSize` | `number` | Ungefähre Größe der JPEG-Datei in Byte. Überschreibt eine eventuelle `quality`-Einstellung. Dies hat keine Auswirkungen auf andere Formate. | |
| `dpi` | `number` oder `object` | Legen Sie DPI für x und y fest. Der Einfachheit halber kann sie auch auf eine einzelne Zahl gesetzt werden, die sowohl für x als auch für y verwendet wird. Es hat keine Auswirkung auf das Bild selbst. | `96` oder `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` oder `object` | Neuberechnen der DPI-Werte für x und y unter Beibehaltung der physischen Größe. Der Einfachheit halber kann sie auch auf eine einzelne Zahl gesetzt werden, die sowohl für x als auch für y verwendet wird. | `96` oder `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Liste der Dateien, die in das ZIP-Archiv aufgenommen werden sollen (`fmt=zip`). Jeder Eintrag kann entweder eine URL-Zeichenfolge oder ein Objekt mit folgenden Feldern sein:<ul><li>`url`: URL zum Herunterladen der Datei</li><li>`path`: Datei unter diesem Pfad in der ZIP-Datei speichern</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Handhaben von Duplikaten für ZIP-Archive (`fmt=zip`). Standardmäßig erzeugen mehrere Dateien, die im selben Pfad in der ZIP gespeichert sind, einen Fehler. Wird `duplicate` auf `ignore` gesetzt, wird nur das erste Asset gespeichert und der Rest wird ignoriert. | `ignore` |
| `watermark` | `object` | Enthält Anweisungen zum [Wasserzeichen](#watermark-specific-fields). |  |

### Wasserzeichenspezifische Felder {#watermark-specific-fields}

Das PNG-Format wird für Wasserzeichen verwendet.

| Name | Typ | Beschreibung | Beispiel |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Skalierung des Wasserzeichens zwischen `0.0` und `1.0`. `1.0` bedeutet, dass das Wasserzeichen seinen ursprünglichen Maßstab (1:1) hat und die niedrigeren Werte die Größe des Wasserzeichens reduzieren. | Ein Wert von `0.5` bedeutet die Hälfte der Originalgröße. |
| `image` | `url` | URL zur PNG-Datei, die für das Wasserzeichen verwendet werden soll. | |

## Asynchrone Ereignisse {#asynchronous-events}

Wenn die Verarbeitung einer Ausgabedarstellung abgeschlossen ist oder ein Fehler auftritt, wird ein Ereignis an eine Adobe gesendet [!DNL `I/O Events Journal`]. Clients müssen die Journal-URL abrufen, die über bereitgestellt wird [`/register`](#register). Die Journalantwort enthält ein `event`-Array, das aus einem Objekt für jedes Ereignis besteht, von dem das `event`-Feld die tatsächliche Ereignis-Payload enthält.

Die Adobe [!DNL `I/O Events`] Typ für alle Ereignisse des [!DNL Asset Compute Service] is `asset_compute`. Das Journal wird automatisch nur für diesen Ereignistyp abonniert und es ist nicht mehr erforderlich, basierend auf dem [!DNL Adobe Developer]-Ereignistyp zu filtern. Die Service-spezifischen Ereignistypen stehen in der `type`-Eigenschaft des Ereignisses zur Verfügung.

### Ereignistypen {#event-types}

| Ereignis | Beschreibung |
|---------------------|-------------|
| `rendition_created` | Wird für jede erfolgreich verarbeitete und hochgeladene Ausgabedarstellung gesendet. |
| `rendition_failed` | Wird für jede Ausgabedarstellung gesendet, bei der die Verarbeitung oder das Hochladen fehlgeschlagen ist. |

### Ereignisattribute {#event-attributes}

| Attribut | Typ | Ereignis | Beschreibung |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Zeitstempel, zu dem das Ereignis in vereinfachtem erweiterten [ISO-8601](https://de.wikipedia.org/wiki/ISO_8601) Format, wie durch JavaScript definiert [Date.toISOString()](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | Die Anfrage-ID der ursprünglichen Anfrage an `/process`, identisch mit der `X-Request-Id`-Kopfzeile. |
| `source` | `object` | `*` | Die `source` der `/process`-Anfrage. |
| `userData` | `object` | `*` | Die `userData` der Ausgabedarstellung aus der `/process`-Anfrage, falls festgelegt. |
| `rendition` | `object` | `rendition_*` | Das entsprechende Ausgabedarstellungsobjekt, das in `/process` übergeben wird. |
| `metadata` | `object` | `rendition_created` | Die [Metadaten](#metadata)-Eigenschaften der Ausgabedarstellung. |
| `errorReason` | `string` | `rendition_failed` | [Grund](#error-reasons) für den einen Fehler bei der Ausgabedarstellung, falls vorhanden. |
| `errorMessage` | `string` | `rendition_failed` | Der Text mit detaillierteren Informationen zum Fehlschlagen der Ausgabedarstellung (falls vorhanden). |

### Metadaten {#metadata}

| Eigenschaft | Beschreibung |
|--------|-------------|
| `repo:size` | Die Größe der Ausgabedarstellung in Byte. |
| `repo:sha1` | Der SHA1-Digest der Ausgabedarstellung. |
| `dc:format` | Der MIME-Typ der Ausgabedarstellung. |
| `repo:encoding` | Die Zeichensatzkodierung der Ausgabedarstellung, falls es sich um ein textbasiertes Format handelt. |
| `tiff:ImageWidth` | Die Breite der Ausgabedarstellung in Pixel. Nur für Bilddarstellungen vorhanden. |
| `tiff:ImageLength` | Die Länge der Ausgabedarstellung in Pixel. Nur für Bilddarstellungen vorhanden. |

### Fehlerursachen {#error-reasons}

| Grund | Beschreibung |
|---------|-------------|
| `RenditionFormatUnsupported` | Das angeforderte Ausgabedarstellungsformat wird für die angegebene Quelle nicht unterstützt. |
| `SourceUnsupported` | Die spezifische Quelle wird nicht unterstützt, obwohl der Typ unterstützt wird. |
| `SourceCorrupt` | Die Quelldaten sind beschädigt. Es enthält leere Dateien. |
| `RenditionTooLarge` | Die Ausgabedarstellung konnte nicht mit den vorsignierten URLs hochgeladen werden, die in `target`. Die tatsächliche Ausgabedarstellungsgröße ist als Metadaten in `repo:size` und wird vom Client verwendet, um diese Ausgabedarstellung mit der richtigen Anzahl vorsignierter URLs erneut zu verarbeiten. |
| `GenericError` | Jeder andere unerwartete Fehler. |
