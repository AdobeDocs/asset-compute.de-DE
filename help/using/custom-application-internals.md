---
title: Machen Sie sich mit der Funktionsweise eines benutzerdefinierten Programms vertraut
description: Interne Funktionsweise eines benutzerdefinierten  [!DNL Asset Compute Service] -Programms, um dessen Funktionsweise besser zu verstehen.
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: ht
source-wordcount: '691'
ht-degree: 100%

---

# Interne Funktionsweise eines benutzerdefinierten Programms {#how-custom-application-works}

Verwenden Sie die folgende Abbildung, um den durchgängigen Workflow zu verstehen, wenn ein digitales Asset mithilfe einem benutzerdefinierten Programm von einem Client verarbeitet wird.

![Workflow für benutzerdefinierte Programme](assets/customworker.svg)

*Abbildung: Schritte zur Verarbeitung eines Assets mit Adobe [!DNL Asset Compute Service]*

## Registrierung {#registration}

Der Client muss vor der ersten Anfrage an [`/process`](api.md#process-request) einmal [`/register`](api.md#register) aufrufen, um die Journal-URL für den Empfang von Adobe [!DNL I/O Events]-Ereignissen einzurichten und abzurufen.

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

Die JavaScript-Bibliothek [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) kann in NodeJS-Programmen verwendet werden, um alle erforderlichen Schritte von der Registrierung über die Verarbeitung bis zur asynchronen Ereignisbehandlung auszuführen. Weitere Informationen zu den erforderlichen Kopfzeilen finden Sie unter [Authentifizierung und Autorisierung](api.md).

## Verarbeitung {#processing}

Der Client sendet eine [Verarbeitungsanfrage](api.md#process-request).

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

Der Client ist für die korrekte Formatierung der Ausgabedarstellungen mit vorab signierten URLs verantwortlich. Die JavaScript-Bibliothek [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) kann in NodeJS-Programmen zum Vorsignieren von URLs verwendet werden. Derzeit unterstützt die Bibliothek nur Azure Blob-Speicher und AWS S3-Container.

Die Verarbeitungsanfrage gibt eine `requestId` zurück, die für die Abfrage von [!DNL Adobe I/O]-Ereignissen verwendet werden kann.

Nachfolgend finden Sie eine Beispielanfrage zur Verarbeitung benutzerdefinierter Programme.

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

[!DNL Asset Compute Service] sendet die Ausgabedarstellungsanfragen für das benutzerdefinierte Programm an das benutzerdefinierte Programm. Der Service sendet eine HTTP-POST-Anfrage an die angegebene Programm-URL, bei der es sich um die gesicherte Web-Aktions-URL von App Builder handelt. Alle Anfragen verwenden das HTTPS-Protokoll, um die Datensicherheit zu maximieren.

Das von einem benutzerdefinierten Programm verwendete [Asset Compute-SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) verarbeitet die HTTP-POST-Anfrage. Es übernimmt auch das Herunterladen der Quelle, das Hochladen von Ausgabedarstellungen, das Senden von Adobe [!DNL I/O Events]-Ereignissen und die Fehlerbehandlung.

<!-- TBD: Add the application diagram. -->

### Programm-Code {#application-code}

Benutzerdefinierter Code muss nur einen Callback bereitstellen, der die lokal verfügbare Quelldatei (`source.path`) akzeptiert. `rendition.path` ist der Speicherort, an dem das Endergebnis einer Asset-Verarbeitungsanfrage platziert werden soll. Das benutzerdefinierte Programm verwendet den Callback, um die lokal verfügbaren Quelldateien unter Verwendung des in (`rendition.path`) angegebenen Namens in eine Ausgabedarstellungsdatei umzuwandeln. Ein benutzerdefiniertes Programm muss in `rendition.path` schreiben, um eine Ausgabedarstellung zu erstellen:

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### Herunterladen von Quelldateien {#download-source}

Ein benutzerdefiniertes Programm behandelt nur lokale Dateien. Das [Asset Compute-SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) übernimmt das Herunterladen der Quelldatei.

### Erstellen von Ausgabedarstellungen {#rendition-creation}

Das SDK ruft für jede Ausgabedarstellung eine asynchrone [Ausgabedarstellungs-Callback-Funktion](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) auf.

Die Callback-Funktion hat Zugriff auf die [Quell](https://github.com/adobe/asset-compute-sdk#source)- und [Ausgabedarstellungsobjekte](https://github.com/adobe/asset-compute-sdk#rendition). `source.path` ist bereits vorhanden und ist der Pfad zur lokalen Kopie der Quelldatei. `rendition.path` ist der Pfad, in dem die verarbeitete Ausgabedarstellung gespeichert werden muss. Sofern das [Flag disableSourceDownload](https://github.com/adobe/asset-compute-sdk#worker-options-optional) nicht gesetzt ist, muss ds Programm den genauen Pfad `rendition.path` verwenden. Andernfalls kann das SDK die Ausgabedarstellungsdatei nicht finden oder identifizieren und schlägt fehl.

Die übermäßige Vereinfachung des Beispiels dient dazu, die Anatomie eines benutzerdefinierten Programms zu illustrieren und sich darauf zu konzentrieren. Das Programm kopiert die Quelldatei einfach in das Ausgabedarstellungsziel.

Weitere Informationen zu den Callback-Parametern für Ausgabedarstellungen finden Sie unter [Asset Compute-SDK-API](https://github.com/adobe/asset-compute-sdk#api-details).

### Hochladen von Ausgabedarstellungen {#upload-rendition}

Nachdem jede Ausgabedarstellung erstellt und in einer Datei mit dem in `rendition.path` angegebenen Pfad gespeichert wurde, lädt das [Asset Compute-SDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) jede Ausgabedarstellung in einen Cloud-Speicher hoch (entweder AWS oder Azure). Ein benutzerdefiniertes Programm erhält genau dann mehrere Ausgabedarstellungen gleichzeitig, wenn die eingehende Anfrage mehrere Ausgabedarstellungen enthält, die auf dieselbe Programm-URL verweisen. Der Upload in den Cloud-Speicher erfolgt nach jeder Ausgabedarstellung und vor dem Ausführen des Callback für die nächste Ausgabedarstellung.

`batchWorker()` verhält sich anders. Es werden alle Ausgabedarstellungen verarbeitet und erst dann hochgeladen, nachdem sie alle verarbeitet wurden.

## [!DNL Adobe I/O]-Ereignisse {#aio-events}

Das SDK sendet Adobe [!DNL I/O Events]-Ereignisse für jede Ausgabedarstellung. Diese Ereignisse sind je nach Ergebnis entweder vom Typ `rendition_created` oder `rendition_failed`. Weitere Informationen finden Sie unter [Asynchrone Asset Compute-Ereignisse](api.md#asynchronous-events).

## Empfangen von [!DNL Adobe I/O]-Ereignissen {#receive-aio-events}

Der Client fragt das Adobe [!DNL I/O Events]-Journal gemäß seiner Verbrauchslogik ab. Die anfängliche Journal-URL ist die in der `/register`-API-Antwort angegebene. Ereignisse können mit der in den Ereignissen vorhandenen `requestId` identifiziert werden, die mit der in `/process` zurückgegebenen übereinstimmt. Jede Ausgabedarstellung verfügt über ein separates Ereignis, das gesendet wird, sobald die Ausgabedarstellung hochgeladen wurde (oder fehlgeschlagen ist). Wenn der Client ein passendes Ereignis erhält, kann er die resultierenden Ausgabedarstellungen anzeigen oder anderweitig verarbeiten.

Die JavaScript-Bibliothek [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) vereinfacht die Journalabfrage mithilfe der Methode `waitActivation()` zum Abrufen aller Ereignisse.

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

Weitere Informationen zum Abrufen von Journalereignissen finden Sie unter „Adobe [[!DNL I/O Events] -API](https://developer.adobe.com/events/docs/guides/api/journaling_api/)“.

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
