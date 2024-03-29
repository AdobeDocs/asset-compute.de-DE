---
title: Testen und Debuggen von benutzerdefinierten  [!DNL Asset Compute Service] -Programmen
description: Testen und Debuggen von benutzerdefinierten  [!DNL Asset Compute Service] -Programmen.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 100%

---

# Testen und Debuggen eines benutzerdefinierten Programms {#test-debug-custom-worker}

## Ausführen von Komponententests für ein benutzerdefiniertes Programm {#test-custom-worker}

Installieren Sie [Docker Desktop](https://www.docker.com/get-started) auf Ihrem Computer. Um ein benutzerdefiniertes Sekundärprogramm zu testen, führen Sie den folgenden Befehl im Stammverzeichnis des Programms aus:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Dadurch wird ein benutzerdefiniertes Komponententest-Framework für Asset Compute-Programmaktionen im Projekt (wie unten beschrieben) ausgeführt. Die Verbindung wird durch eine Konfiguration in der `package.json`-Datei hergestellt. Es ist auch möglich, JavaScript-Komponententests wie Jest durchzuführen. `aio app test` führt beide aus.

Das Plug-in [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) ist als Entwicklungsabhängigkeit in die App des benutzerdefinierten Programms eingebettet, sodass es nicht auf Build-/Testsystemen installiert werden muss.

### Test-Framework für Programmkomponenten {#unit-test-framework}

Mit dem Asset Compute-Test-Framework für Programmkomponenten können Sie Programme testen, ohne Code schreiben zu müssen. Es beruht auf dem Quell-zu-Ausgabedarstellungsdateiprinzip von Programmen. Eine bestimmte Datei- und Ordnerstruktur muss eingerichtet werden, um Testfälle mit Testquelldateien, optionalen Parametern, erwarteten Ausgabedarstellungen und benutzerdefinierten Validierungsskripten zu definieren. Standardmäßig werden die Ausgabedarstellungen auf Byte-Gleichheit verglichen. Darüber hinaus können externe HTTP-Services mit einfachen JSON-Dateien einfach nachgeahmt werden.

### Hinzufügen von Tests {#add-tests}

Tests werden im `test`-Ordner auf der Stammebene des [!DNL Adobe I/O]-Projekts erwartet. Die Testfälle für jedes Programm sollten sich im Pfad `test/asset-compute/<worker-name>` befinden, wobei für jeden Testfall ein Ordner vorhanden sein muss:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Sehen Sie sich einige [Beispiele für benutzerdefinierte Programme](https://github.com/adobe/asset-compute-example-workers/) an. Nachstehend finden Sie eine ausführliche Referenz.

### Testausgabe {#test-output}

Die detaillierte Testausgabe einschließlich der Protokolle des benutzerdefinierten Programms ist im `build`-Ordner im Stamm der Adobe Developer App Builder-App verfügbar, wie in der `aio app test`-Ausgabe gezeigt.

### Nachahmen externer Services {#mock-external-services}

Es ist möglich, externe Service-Aufrufe in Ihren Aktionen nachzuahmen, indem Sie `mock-<HOST_NAME>.json`-Dateien in Ihren Testfällen definieren, wobei HOST_NAME der Host ist, den Sie nachahmen möchten. Ein Anwendungsbeispiel ist ein Programm, das S3 separat aufruft. Die neue Teststruktur würde folgendermaßen aussehen:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

Die nachgeahmte Datei ist eine HTTP-Antwort im JSON-Format. Weitere Informationen finden Sie in [dieser Dokumentation](https://www.mock-server.com/mock_server/creating_expectations.html). Wenn mehrere Host-Namen nachgeahmt werden müssen, definieren Sie mehrere `mock-<mocked-host>.json`-Dateien. Im Folgenden finden Sie ein Beispiel einer nachgeahmten Datei für `google.com` mit dem Namen `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

Das Beispiel `worker-animal-pictures` enthält eine [nachgeahmte Datei](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) für den Wikimedia-Service, mit dem es interagiert.

#### Gemeinsames Nutzen von Dateien über Testfälle hinweg {#share-files-across-test-cases}

Es wird empfohlen, relative Symlinks zu verwenden, wenn Sie `file.*`-, `params.json`- oder `validate`-Skripte über mehrere Tests hinweg gemeinsam nutzen. Diese werden von git unterstützt. Achten Sie darauf, Ihren gemeinsam genutzten Dateien einen eindeutigen Namen zu geben, da Sie möglicherweise verschiedene Dateien haben. Im folgenden Beispiel mischen und vergleichen die Tests einige gemeinsam genutzte und ihre eigenen Dateien:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Testen erwarteter Fehler {#test-unexpected-errors}

Fehlertestfälle sollten keine erwartete `rendition.*`-Datei enthalten und den erwarteten `errorReason` in der `params.json`-Datei definieren.

>[!NOTE]
>
>Wenn ein Testfall keine erwartete `rendition.*`-Datei enthält und nicht den erwarteten `errorReason` in der `params.json`-Datei definiert, wird davon ausgegangen, dass es sich um einen Fehlerfall mit einem beliebigen `errorReason` handelt.

Struktur von Fehlertestfällen:

```json
<error_test_case>/
    file.jpg
    params.json
```

Parameterdatei mit Fehlerursache:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Sehen Sie dazu die vollständige Liste und Beschreibung der [Asset Compute-Fehlerursachen](https://github.com/adobe/asset-compute-commons#error-reasons).

## Debuggen eines benutzerdefinierten Programms {#debug-custom-worker}

Die folgenden Schritte zeigen, wie Sie Ihr benutzerdefiniertes Programm mit Visual Studio Code debuggen können. Er ermöglicht die Anzeige von Live-Protokollen, das Auffinden von Haltepunkten und das Durchlaufen des Codes sowie das Live-Neuladen lokaler Code-Änderungen bei jeder Aktivierung.

Viele dieser Schritte werden in der Regel von `aio` gemäß der Vorkonfiguration automatisiert. Weitere Informationen finden Sie im Abschnitt zum Debugging des Programms in der [Dokumentation zu Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Die folgenden Schritte beinhalten vorerst eine Problemumgehung.

1. Installieren Sie die neuste Version von [wskdebug](https://github.com/apache/openwhisk-wskdebug) und optional [ngrok](https://www.npmjs.com/package/ngrok) von GitHub.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Fügen Sie Folgendes zu Ihrer JSON-Datei mit den Benutzereinstellungen hinzu. Damit wird weiterhin der alte VS-Code-Debugger verwendet, da der neue [einige Probleme](https://github.com/apache/openwhisk-wskdebug/issues/74) mit wskdebug hat: `"debug.javascript.usePreview": false`.
1. Schließen Sie alle Instanzen von Apps, die über `aio app run` geöffnet sind.
1. Stellen Sie den neuesten Code mit `aio app deploy` bereit.
1. Führen Sie nur das Asset Compute-Entwickler-Tool mit `aio asset-compute devtool` aus. Lassen Sie es geöffnet.
1. Fügen Sie im VS-Code-Editor die folgende Debug-Konfiguration zu Ihrer `launch.json` hinzu:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Rufen Sie den `ACTION NAME` aus der Ausgabe von `aio app deploy` ab.

1. Wählen Sie `wskdebug worker` in der Ausführungs-/Debug-Konfiguration aus und klicken Sie auf das Wiedergabesymbol. Warten Sie auf den Start, bis die Meldung **[!UICONTROL Ready for activations]** im Fenster der **[!UICONTROL Debug Console]** angezeigt wird.

1. Klicken Sie im Entwickler-Tool auf **[!UICONTROL Run]**. Sie können die im VS Code-Editor ausgeführten Aktionen sehen und die Protokolle werden angezeigt.

1. Legen Sie einen Haltepunkt im Code fest und führen Sie den Code erneut aus. Der Haltepunkt sollte wirksam werden.

Alle Code-Änderungen werden in Echtzeit geladen und werden wirksam, sobald die nächste Aktivierung erfolgt.

>[!NOTE]
>
>In benutzerdefinierten Programmen sind für jede Anfrage zwei Aktivierungen vorhanden. Die erste Anfrage ist eine Web-Aktion, die sich im SDK-Code asynchron aufruft. Die zweite Aktivierung verwendet Ihren Code.
