---
title: „[!DNL Asset Compute Service] HTTP-API“
description: „[!DNL Asset Compute Service] HTTP-API zum Erstellen benutzerdefinierter Programme“.
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: ht
source-wordcount: '2862'
ht-degree: 100%

---

# [!DNL Asset Compute Service]-HTTP-API {#asset-compute-http-api}

Die Verwendung der API ist auf Entwicklungszwecke beschränkt. Die API wird bei der Entwicklung benutzerdefinierter Programme als Kontext bereitgestellt. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] verwendet die API, um die Verarbeitungsinformationen an ein benutzerdefiniertes Programm zu übergeben. Weitere Informationen finden Sie unter [Verwenden von Asset-Microservices und Verarbeitungsprofilen](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] ist nur zur Verwendung mit [!DNL Experience Manager] as a [!DNL Cloud Service] verfügbar.

Jeder Client der [!DNL Asset Compute Service]-HTTP-API muss diesem allgemeinen Ablauf folgen:

1. Ein Client wird als [!DNL Adobe Developer Console]-Projekt in einer IMS-Organisation bereitgestellt. Für jeden separaten Client (System oder Umgebung) ist ein eigenes Projekt erforderlich, um den Ereignisdatenfluss zu trennen.

1. Ein Client generiert mithilfe der [JWT (Service Account)-Authentifizierung](https://developer.adobe.com/developer-console/docs/guides/) ein Zugriffs-Token für das technische Konto.

1. Ein Client ruft [`/register`](#register) nur einmal auf, um die Journal-URL abzurufen.

1. Ein Client ruft [`/process`](#process-request) für jedes Asset auf, für das er Ausgabedarstellungen generieren möchte. Der Aufruf ist asynchron.

1. Ein Client fragt das Journal regelmäßig ab, um [Ereignisse zu empfangen](#asynchronous-events). Er empfängt Ereignisse für jede angeforderte Ausgabedarstellung, wenn die Ausgabedarstellung erfolgreich verarbeitet wurde (Ereignistyp `rendition_created`) oder wenn ein Fehler aufgetreten ist (Ereignistyp `rendition_failed`).

Das Modul [adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) vereinfacht die Verwendung der API im Node.js-Code.

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

Für diese Bereiche muss das [!DNL Adobe Developer Console]-Projekt die Services `Asset Compute`, `I/O Events`, und `I/O Management API` abonnieren. Die einzelnen Bereiche werden wie folgt unterteilt:

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

Jeder Client von [!DNL Asset Compute service] – ein eindeutiges [!DNL Adobe Developer Console]-Projekt, das den Service abonniert hat – muss sich [registrieren](#register-request), bevor er Verarbeitungsanfragen stellen kann. Der Registrierungsschritt gibt das eindeutige Ereignisjournal zurück, das zum Abrufen der asynchronen Ereignisse aus der Verarbeitung der Ausgabedarstellung erforderlich ist.

Am Ende seines Lebenszyklus kann ein Client [die Registrierung aufheben](#unregister-request).

### Registrierungsanfrage {#register-request}

Mit diesem API-Aufruf wird ein [!DNL Asset Compute]-Client eingerichtet und die Ereignisjournal-URL bereitgestellt. Dies ist ein idempotenter Vorgang, der für jeden Client nur einmal aufgerufen werden muss. Er kann erneut aufgerufen werden, um die Journal-URL abzurufen.

| Parameter | Wert |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pfad | `/register` |
| Kopfzeile `Authorization` | Alle [autorisierungsbezogenen Kopfzeilen](#authentication-and-authorization). |
| Kopfzeile `x-request-id` | Optional. Wird von Clients für eine eindeutige End-to-End-Kennung der Verarbeitungsanfragen systemübergreifend festgelegt. |
| Hauptteil der Anfrage | Muss leer sein. |

### Registrierungsantwort {#register-response}

| Parameter | Wert |
|-----------------------|------------------------------------------------------|
| MIME-Typ | `application/json` |
| Kopfzeile `X-Request-Id` | Entweder identisch mit der Anfragekopfzeile `X-Request-Id` oder eine eindeutig erstellte. Verwenden Sie diese Option, um systemübergreifende Anfragen und/oder Support-Anfragen zu identifizieren. |
| Hauptteil der Antwort | Ein JSON-Objekt mit den Feldern `journal`, `ok` oder `requestId`. |

Die HTTP-Status-Codes lauten:

* **200 Erfolg**: Bei erfolgreicher Anfrage. Die `journal`-URL empfängt Benachrichtigungen über die Ergebnisse der durch `/process` initiierten asynchronen Verarbeitung. Es wird über erfolgreich abgeschlossene `rendition_created`-Ereignisse oder `rendition_failed`-Ereignisse bei fehlgeschlagenem Prozess informiert.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 Nicht autorisiert**: Tritt auf, wenn die Anfrage keine gültige [Authentifizierung](#authentication-and-authorization) aufweist. Ein Beispiel könnte ein ungültiges Zugriffs-Token oder ein ungültiger API-Schlüssel sein.

* **403 Verboten**: Tritt auf, wenn die Anfrage keine gültige [Autorisierung](#authentication-and-authorization) aufweist. Beispielsweise könnte ein gültiges Zugriffs-Token vorliegen, aber das Adobe Developer Console-Projekt (technisches Konto) hat nicht alle erforderlichen Services abonniert.

* **429 Zu viele Anfragen**: Tritt auf, wenn das System durch diesen Client oder anderweitig überlastet wird. Clients sollten es mit einem [exponentiellen Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) erneut versuchen. Der Hauptteil ist leer.
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

Durch diesen API-Aufruf wird die Registrierung eines [!DNL Asset Compute]-Clients aufgehoben. Danach kann `/process` nicht mehr aufgerufen werden. Wenn Sie den API-Aufruf für einen nicht registrierten Client oder einen noch nicht registrierten Client verwenden, wird ein `404`-Fehler zurückgegeben.

| Parameter | Wert |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pfad | `/unregister` |
| Kopfzeile `Authorization` | Alle [autorisierungsbezogenen Kopfzeilen](#authentication-and-authorization). |
| Kopfzeile `x-request-id` | Optional. Kann von Clients für eine eindeutige End-to-End-Kennung der Verarbeitungsanfragen systemübergreifend festgelegt werden. |
| Hauptteil der Anfrage | Leer. |

### Antwort zum Aufheben der Registrierung {#unregister-response}

| Parameter | Wert |
|-----------------------|------------------------------------------------------|
| MIME-Typ | `application/json` |
| Kopfzeile `X-Request-Id` | Entweder identisch mit der Anfragekopfzeile `X-Request-Id` oder eine eindeutig erstellte. Verwenden Sie diese Option, um systemübergreifende Anfragen oder Support-Anfragen zu identifizieren. |
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

* **404 Nicht gefunden**: Dieser Status wird angezeigt, wenn die angegebenen Anmeldeinformationen nicht registriert oder ungültig sind.

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

Der `process`-Vorgang sendet einen Auftrag, der ein Quell-Asset basierend auf den Anweisungen in der Anfrage in mehrere Ausgabedarstellungen umwandelt. Benachrichtigungen über einen erfolgreichen Abschluss (Ereignistyp `rendition_created`) oder einen Fehler (Ereignistyp `rendition_failed`) werden an ein Ereignisjournal gesendet, das einmal mit [`/register`](#register) abgerufen werden muss, bevor eine beliebige Anzahl von `/process`-Anfragen gestellt werden kann. Fehlerhaft gebildete Anfragen schlagen sofort mit einem Fehler-Code 400 fehl.

Binärdateien werden mithilfe von URLs referenziert, z. B. mit von Amazon AWS S3 vorsignierten URLs oder Azure Blob Storage-SAS-URLs. Sie dienen sowohl zum Lesen des `source`-Assets (`GET`-URLs) als auch zum Schreiben der Ausgabedarstellungen (`PUT`-URLs). Der Client ist für die Generierung dieser vorsignierten URLs verantwortlich.

| Parameter | Wert |
|--------------------------|------------------------------------------------------|
| Methode | `POST` |
| Pfad | `/process` |
| MIME-Typ | `application/json` |
| Kopfzeile `Authorization` | Alle [autorisierungsbezogenen Kopfzeilen](#authentication-and-authorization). |
| Kopfzeile `x-request-id` | Optional. Clients können eine eindeutige End-to-End-Kennung festlegen, um Verarbeitungsanfragen systemübergreifend zu verfolgen. |
| Hauptteil der Anfrage | Muss im JSON-Format für Verarbeitungsanfragen vorliegen, wie unten beschrieben. Er enthält Anweisungen dazu, welches Asset verarbeitet und welche Ausgabedarstellungen generiert werden sollen. |

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
| `source` | `string` | URL des Quell-Assets, das verarbeitet wird. Optional, je nach angefordertem Ausgabedarstellungsformat (z. B. `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Beschreibt das Quell-Asset, das verarbeitet wird. Siehe Beschreibung der [Quellobjektfelder](#source-object-fields) unten. Optional, je nach angefordertem Ausgabedarstellungsformat (z. B. `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
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
| `name` | `string` | Name der Quell-Asset-Datei. Eine Dateierweiterung im Namen kann verwendet werden, wenn kein MIME-Typ erkannt wird. Der Name hat Vorrang vor dem Dateinamen, der im URL-Pfad angegeben ist. Außerdem hat er Vorrang vor dem Dateinamen in der Kopfzeile `content-disposition` der binären Ressource. Der Standardwert ist „file“. | `"image.jpg"` |
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
| Kopfzeile `X-Request-Id` | Entweder identisch mit der Anfragekopfzeile `X-Request-Id` oder eine eindeutig erstellte. Verwenden Sie diese Option, um systemübergreifende Anfragen oder Support-Anfragen zu identifizieren. |
| Hauptteil der Antwort | Ein JSON-Objekt mit den Feldern `ok` und `requestId`. |

Status-Codes:

* **200 Erfolg**: Wenn die Anfrage erfolgreich gesendet wurde. Die Antwort-JSON enthält `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Ungültige Anfrage**: Wenn die Anfrage falsch strukturiert ist, z. B. wenn der JSON-Payload die erforderlichen Felder fehlen. Die Antwort-JSON enthält `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 Nicht autorisiert**: Wenn die Anfrage keine gültige [Authentifizierung](#authentication-and-authorization) aufweist. Ein Beispiel könnte ein ungültiges Zugriffs-Token oder ein ungültiger API-Schlüssel sein.
* **403 Verboten**: Wenn die Anfrage keine gültige [Autorisierung](#authentication-and-authorization) aufweist. Beispielsweise könnte ein gültiges Zugriffs-Token vorliegen, aber das Adobe Developer Console-Projekt (technisches Konto) hat nicht alle erforderlichen Services abonniert.
* **429 Zu viele Anfragen**: Tritt auf, wenn das System überlastet ist, entweder aufgrund dieses bestimmten Clients oder aufgrund der Belastung insgesamt. Die Clients können es mit einem [exponentiellen Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) erneut versuchen. Der Hauptteil ist leer.
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

Die meisten Clients neigen dazu, dieselbe Anfrage mit [exponentiellem Backoff](https://de.wikipedia.org/wiki/Binary_Exponential_Backoff) bei Fehlern zu wiederholen, mit *Ausnahme* von Konfigurationsproblemen wie 401 oder 403 oder ungültigen Anfragen wie 400. Abgesehen von der regulären Ratenbegrenzung über Antworten vom Typ 429 kann ein vorübergehender Service-Ausfall oder eine vorübergehende Service-Beschränkung zu 5xx-Fehlern führen. Es wäre in dem Fall ratsam, es nach einer gewissen Zeit erneut zu versuchen.

Alle JSON-Antworten (sofern vorhanden) enthalten die `requestId`, die dem Wert des `X-Request-Id`-Headers entspricht. Adobe empfiehlt, Informationen aus dem Header zu lesen, da dieser immer vorhanden ist. Die `requestId` wird auch in allen Ereignissen, die mit Verarbeitungsanfragen zusammenhängen, als `requestId` zurückgegeben. Clients dürfen keine Annahmen über das Format dieser Zeichenfolge treffen. Es handelt sich um eine opake Zeichenfolgenkennung.

## Opt-in für die Nachbearbeitung {#opt-in-to-post-processing}

Das [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) unterstützt eine Reihe grundlegender Optionen für die Nachbearbeitung von Bildern. Benutzerdefinierte Sekundärprogramme können sich explizit für die Nachbearbeitung entscheiden, indem sie das Feld `postProcess` im Ausgabedarstellungsobjekt auf `true` setzen.

Folgende Anwendungsfälle werden unterstützt:

* „Zuschneiden“ ist eine Ausgabedarstellung für ein Rechteck, dessen Grenzen durch „crop.w“, „crop.h“, „crop.x“ und „crop.y“ definiert sind. Die Zuschnittdetails werden im Feld `instructions.crop` des Ausgabedarstellungsobjekts angegeben.
* Ändern der Bildgröße unter Verwendung von Breite, Höhe oder beidem. Dies wird durch `instructions.width` und `instructions.height` im Ausgabedarstellungsobjekt definiert. Um die Größe nur unter Verwendung von Breite oder Höhe zu ändern, legen Sie nur einen Wert fest. Compute Service behält das Seitenverhältnis bei.
* Festlegen der Qualität für ein JPEG-Bild. Diese wird durch `instructions.quality` im Ausgabedarstellungsobjekt definiert. Ein Qualitätsniveau von 100 steht für höchste Qualität, niedrigere Zahlen bedeuten einen Qualitätsverlust.
* Erstellen von Zwischenzeilenbildern. Dies wird durch `instructions.interlace` im Ausgabedarstellungsobjekt definiert.
* Stellen Sie die DPI so ein, dass die gerenderte Größe für Desktop-Publishing-Zwecke angepasst wird, indem Sie die auf die Pixel angewendete Skalierung anpassen. Dies wird durch `instructions.dpi` im Ausgabedarstellungsobjekt definiert, um die DPI-Auflösung zu ändern. Verwenden Sie die `convertToDpi`-Anweisungen, um die Größe des Bildes so zu ändern, dass es bei einer anderen Auflösung dieselbe Größe hat.
* Ändern Sie die Größe des Bildes so, dass seine gerenderte Breite oder Höhe bei der angegebenen Zielauflösung (DPI) mit dem Original übereinstimmt. Dies wird durch `instructions.convertToDpi` im Ausgabedarstellungsobjekt definiert.

## Wasserzeichen-Assets {#add-watermark}

Das [Asset Compute SDK](https://github.com/adobe/asset-compute-sdk) unterstützt das Hinzufügen eines Wasserzeichens zu PNG-, JPEG-, TIFF- und GIF-Bilddateien. Das Wasserzeichen wird der Ausgabedarstellung entsprechend den Ausgabedarstellungsanweisungen im `watermark`-Objekt hinzugefügt.

Das Wasserzeichen wird während der Nachbearbeitung der Ausgabedarstellung hinzugefügt. Um Assets mit Wasserzeichen zu versehen, [aktiviert das benutzerdefinierte Sekundärprogramm die Nachbearbeitung](#opt-in-to-post-processing), indem es das Feld `postProcess` im Ausgabedarstellungsobjekt auf `true` setzt. Wenn sich das Sekundärprogramm nicht dafür entscheidet, wird das Wasserzeichen nicht hinzugefügt, auch wenn das Wasserzeichenobjekt im Ausgabedarstellungsobjekt in der Anfrage gesetzt ist.

## Ausgabedarstellungsanweisungen {#rendition-instructions}

Im Folgenden sind die verfügbaren Optionen für das Array `renditions` in [`/process`](#process-request) aufgeführt.

### Allgemeine Felder {#common-fields}

| Name | Typ | Beschreibung | Beispiel |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | Das Zielformat der Ausgabedarstellung kann auch `text` zum Extrahieren von Text und `xmp` zum Extrahieren von XMP-Metadaten als XML sein. Siehe [Unterstützte Formate](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/file-format-support). | `png` |
| `worker` | `string` | URL eines [benutzerdefinierten Programms](develop-custom-application.md). Muss eine `https://`-URL sein. Wenn dieses Feld vorhanden ist, erstellt ein benutzerdefiniertes Programm die Ausgabedarstellung. Jedes andere festgelegte Ausgabedarstellungsfeld wird dann im benutzerdefinierten Programm verwendet. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Die URL, auf die die generierte Ausgabedarstellung mit HTTP PUT hochgeladen werden soll. | `http://w.com/img.jpg` |
| `target` | `object` | Mehrteilige vorsignierte Informationen zum URL-Upload für die generierte Ausgabedarstellung. Diese Informationen gelten für den [direkten binären AEM-/Oak-Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) mit diesem [mehrteiligen Upload-Verhalten](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Felder:<ul><li>`urls`: Array von Zeichenfolgen, eine für jede vorsignierte Teil-URL</li><li>`minPartSize`: die Mindestgröße für eine Teil-URL</li><li>`maxPartSize`: die Maximalgröße für eine Teil-URL</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Optional. Der Client steuert den reservierten Bereich und gibt ihn unverändert an Ausgabedarstellungsereignisse weiter. Ermöglicht es Clients, benutzerdefinierte Informationen zum Identifizieren von Ausgabedarstellungsereignissen hinzuzufügen. Darf in benutzerdefinierten Programmen nicht geändert oder als zuverlässige Quelle verwendet werden, da Clients jederzeit Änderungen daran vornehmen können. | `{ ... }` |

### Ausgabedarstellungsspezifische Felder {#rendition-specific-fields}

Eine Liste der derzeit unterstützten Dateiformate finden Sie unter [Unterstützte Dateiformate](https://experienceleague.adobe.com/de/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Name | Typ | Beschreibung | Beispiel |
|-------------------|----------|-------------|---------|
| `*` | `*` | Erweiterte, benutzerdefinierte Felder können hinzugefügt werden, die ein [benutzerdefiniertes Programm](develop-custom-application.md) versteht. | |
| `embedBinaryLimit` | `number` in Byte | Wenn die Dateigröße der Ausgabedarstellung unter dem angegebenen Wert liegt, wird sie in das Ereignis aufgenommen, das nach Abschluss der Erstellung gesendet wird. Die maximal zulässige Größe für die Einbettung beträgt 32 KB (32 x 1024 Byte). Wenn eine Ausgabedarstellung größer als das Limit `embedBinaryLimit` ist, wird sie an einem Ort im Cloud-Speicher abgelegt und nicht in das Ereignis eingebettet. | `3276` |
| `width` | `number` | Breite in Pixel. Nur für Bilddarstellungen. | `200` |
| `height` | `number` | Höhe in Pixel. Nur für Bilddarstellungen. | `200` |
|                   |          | Das Seitenverhältnis wird in folgenden Fällen immer beibehalten: <ul> <li> Wenn sowohl `width` als auch `height` angegeben werden, wird die Bildgröße angepasst, wobei das Seitenverhältnis beibehalten wird. </li><li> Wenn nur `width` oder `height` angegeben wird, verwendet das resultierende Bild die entsprechende Abmessung und behält dabei das Seitenverhältnis bei.</li><li> Wenn `width` oder `height` nicht angegeben wird, wird die Pixelgröße des Originalbilds verwendet. Diese hängt vom Quelltyp ab. Bei einigen Formaten, z. B. PDF-Dateien, wird eine Standardgröße verwendet. Es kann eine Maximalgröße geben.</li></ul> | |
| `quality` | `number` | Geben Sie die JPEG-Qualität im Bereich von `1` bis `100` an. Gilt nur für Bilddarstellungen. | `90` |
| `xmp` | `string` | Wird nur für das Zurückschreiben von XMP-Metadaten verwendet und ist base64-kodiertes XMP zum Zurückschreiben in die angegebene Ausgabedarstellung. | |
| `interlace` | `bool` | Erstellen Sie PNG- oder GIF-Zwischenzeilenbilder oder progressive JPEG-Bilder, indem Sie den Parameter auf `true` setzen. Hat keine Auswirkungen auf andere Dateiformate. | |
| `jpegSize` | `number` | Ungefähre Größe der JPEG-Datei in Byte. Überschreibt eine eventuelle `quality`-Einstellung. Hat keine Auswirkungen auf andere Formate. | |
| `dpi` | `number` oder `object` | Legen Sie DPI für x und y fest. Der Einfachheit halber kann der Parameter auch auf eine einzelne Zahl gesetzt werden, die sowohl für „x“ als auch für „y“ verwendet wird. Dies hat keine Auswirkung auf das Bild selbst. | `96` oder `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` oder `object` | Neuberechnen der DPI-Werte für x und y unter Beibehaltung der physischen Größe. Der Einfachheit halber kann der Parameter auch auf eine einzelne Zahl gesetzt werden, die sowohl für „x“ als auch für „y“ verwendet wird. | `96` oder `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Liste der Dateien, die in das ZIP-Archiv aufgenommen werden sollen (`fmt=zip`). Jeder Eintrag kann entweder eine URL-Zeichenfolge oder ein Objekt mit folgenden Feldern sein:<ul><li>`url`: URL zum Herunterladen der Datei</li><li>`path`: Datei unter diesem Pfad in der ZIP-Datei speichern</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Handhaben von Duplikaten für ZIP-Archive (`fmt=zip`). Standardmäßig erzeugen mehrere Dateien, die im selben Pfad in der ZIP gespeichert sind, einen Fehler. Wird `duplicate` auf `ignore` gesetzt, wird nur das erste Asset gespeichert und der Rest wird ignoriert. | `ignore` |
| `watermark` | `object` | Enthält Anweisungen zum [Wasserzeichen](#watermark-specific-fields). |  |

### Wasserzeichenspezifische Felder {#watermark-specific-fields}

Das PNG-Format wird für Wasserzeichen verwendet.

| Name | Typ | Beschreibung | Beispiel |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Skalierung des Wasserzeichens zwischen `0.0` und `1.0`. `1.0` bedeutet, dass das Wasserzeichen seinen ursprünglichen Maßstab (1:1) aufweist. Niedrigere Werte reduzieren die Größe des Wasserzeichens. | Ein Wert von `0.5` bedeutet die Hälfte der Originalgröße. |
| `image` | `url` | URL zur PNG-Datei, die für das Wasserzeichen verwendet werden soll. | |

## Asynchrone Ereignisse {#asynchronous-events}

Sobald die Verarbeitung einer Ausgabedarstellung abgeschlossen ist oder wenn ein Fehler auftritt, wird ein Ereignis an ein Adobe [!DNL `I/O Events Journal`] gesendet. Clients müssen auf die Journal-URL lauschen, die über [`/register`](#register) bereitgestellt wird. Die Journalantwort enthält ein `event`-Array, das aus einem Objekt für jedes Ereignis besteht, von dem das Feld `event` die tatsächliche Ereignis-Payload enthält.

Der Adobe [!DNL `I/O Events`]-Typ für alle Ereignisse von [!DNL Asset Compute Service] ist `asset_compute`. Das Journal wird automatisch nur für diesen Ereignistyp abonniert und es ist nicht mehr erforderlich, basierend auf dem [!DNL Adobe Developer]-Ereignistyp zu filtern. Die Service-spezifischen Ereignistypen stehen in der `type`-Eigenschaft des Ereignisses zur Verfügung.

### Ereignistypen {#event-types}

| Ereignis | Beschreibung |
|---------------------|-------------|
| `rendition_created` | Wird für jede erfolgreich verarbeitete und hochgeladene Ausgabedarstellung gesendet. |
| `rendition_failed` | Wird für jede Ausgabedarstellung gesendet, bei der die Verarbeitung oder das Hochladen fehlgeschlagen ist. |

### Ereignisattribute {#event-attributes}

| Attribut | Typ | Ereignis | Beschreibung |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Zeitstempel, zu dem das Ereignis im vereinfachten erweiterten [ISO-8601](https://de.wikipedia.org/wiki/ISO_8601)-Format gesendet wurde, wie durch die JavaScript-Methode [Date.toISOString()](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString) definiert. |
| `requestId` | `string` | `*` | Die Anfrage-ID der ursprünglichen Anfrage an `/process`, identisch mit der `X-Request-Id`-Kopfzeile. |
| `source` | `object` | `*` | Die `source` der `/process`-Anfrage. |
| `userData` | `object` | `*` | Die `userData` der Ausgabedarstellung aus der `/process`-Anfrage, falls festgelegt. |
| `rendition` | `object` | `rendition_*` | Das entsprechende Ausgabedarstellungsobjekt, das in `/process` übergeben wird. |
| `metadata` | `object` | `rendition_created` | Die [Metadaten](#metadata)-Eigenschaften der Ausgabedarstellung. |
| `errorReason` | `string` | `rendition_failed` | [Grund](#error-reasons) für den einen Fehler bei der Ausgabedarstellung, falls vorhanden. |
| `errorMessage` | `string` | `rendition_failed` | Text mit detaillierteren Informationen zu einem eventuellen Fehler bei der Ausgabedarstellung. |

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
| `SourceCorrupt` | Die Quelldaten sind beschädigt. Enthält leere Dateien. |
| `RenditionTooLarge` | Die Ausgabedarstellung konnte nicht mit den unter `target` angegebenen vorsignierten URLs hochgeladen werden. Die tatsächliche Ausgabedarstellungsgröße ist als Metadaten unter `repo:size` verfügbar und wird vom Client verwendet, um diese Ausgabedarstellung mit der richtigen Anzahl vorsignierter URLs erneut zu verarbeiten. |
| `GenericError` | Jeder andere unerwartete Fehler. |
