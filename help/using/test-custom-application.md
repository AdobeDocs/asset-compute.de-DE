---
title: Testen und Debuggen von benutzerdefinierten  [!DNL Asset Compute Service] -Programmen
description: Testen und Debuggen von benutzerdefinierten  [!DNL Asset Compute Service] -Programmen.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: ht
source-wordcount: '775'
ht-degree: 100%

---

# Testen und Debuggen eines benutzerdefinierten Programms {#test-debug-custom-worker}

## Ausführen von Komponententests für eine benutzerdefinierte Anwendung {#test-custom-worker}

Installieren Sie [Docker Desktop](https://www.docker.com/get-started) auf Ihrem Computer. Um eine benutzerdefinierte Sekundäranwendung (einen Worker) zu testen, führen Sie den folgenden Befehl im Stammverzeichnis der Anwendung aus:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Dieser Befehl führt ein benutzerdefiniertes Komponententest-Framework für Asset Compute-Anwendungsaktionen im Projekt aus, wie unten beschrieben. Die Verbindung wird durch eine Konfiguration in der `package.json`-Datei hergestellt. Es ist auch möglich, JavaScript-Komponententests wie Jest durchzuführen. Mit `aio app test` wird beides ausgeführt.

Das Plug-in [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) ist als Entwicklungsabhängigkeit in die App der benutzerdefinierten Anwendung eingebettet, sodass es nicht auf Build-/Testsystemen installiert werden muss.

### Test-Framework für Programmkomponenten {#unit-test-framework}

Mit dem Asset Compute-Test-Framework für Anwendungskomponenten können Sie Anwendungen testen, ohne Code schreiben zu müssen. Es beruht auf dem Quell-zu-Ausgabedarstellungsdateiprinzip von Programmen. Es muss eine bestimmte Datei- und Ordnerstruktur eingerichtet werden, um Testfälle mit Testquelldateien, optionalen Parametern, erwarteten Ausgabedarstellungen und benutzerdefinierten Validierungsskripten zu definieren. Standardmäßig werden die Ausgabedarstellungen auf Byte-Gleichheit verglichen. Darüber hinaus können externe HTTP-Services mit einfachen JSON-Dateien einfach nachgeahmt werden.

### Hinzufügen von Tests {#add-tests}

Tests werden im Ordner `test` auf Stammebene des Projekts erwartet. Die Testfälle für jedes Programm sollten sich im Pfad `test/asset-compute/<worker-name>` befinden, wobei für jeden Testfall ein Ordner vorhanden sein muss:

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

Das Verzeichnis `build` im Stammverzeichnis der Adobe Developer App Builder-App befinden sich die ausführlichen Testergebnisse und Protokolle der benutzerdefinierten Anwendung. Diese Details werden auch in der Ausgabe des Befehls `aio app test` angezeigt.

### Nachahmen externer Services {#mock-external-services}

Sie können externe Service-Aufrufe in Ihren Aktionen simulieren, indem Sie `mock-<HOST_NAME>.json`-Dateien für Ihre Testszenarien erstellen. HOST_NAME ist dabei der spezifische Host ist, der imitiert werden soll. Ein Anwendungsbeispiel wäre eine Anwendung, die S3 separat aufruft. Die neue Teststruktur würde folgendermaßen aussehen:

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

Das Beispiel `worker-animal-pictures` enthält eine [Mock-Datei](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) für den Wikimedia-Service, mit dem es interagiert.

#### Gemeinsames Nutzen von Dateien über Testfälle hinweg {#share-files-across-test-cases}

Adobe empfiehlt, relative Symlinks zu verwenden, wenn Sie `file.*`-, `params.json`- oder `validate`-Skripte für mehrere Tests nutzen. Diese werden von Git unterstützt. Achten Sie darauf, Ihren gemeinsam genutzten Dateien einen eindeutigen Namen zu geben, da Sie möglicherweise verschiedene Dateien haben. Im folgenden Beispiel mischen und vergleichen die Tests einige gemeinsam genutzte und ihre eigenen Dateien:

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

Sehen Sie sich dazu die vollständige Liste und Beschreibung der [Asset Compute-Fehlerursachen](https://github.com/adobe/asset-compute-commons#error-reasons) an.

## Debuggen eines benutzerdefinierten Programms {#debug-custom-worker}

Die folgenden Schritte zeigen, wie Sie Ihr benutzerdefiniertes Programm mit Visual Studio Code debuggen können. Er ermöglicht die Anzeige von Live-Protokollen, das Auffinden von Haltepunkten und das Durchlaufen des Codes sowie das Live-Neuladen lokaler Code-Änderungen bei jeder Aktivierung.

Mit `aio` werden viele dieser Schritte standardmäßig automatisiert. Navigieren Sie zum Abschnitt „Debuggen der Anwendung“ in der [Adobe Developer App Builder-Dokumentation](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Die folgenden Schritte beinhalten vorerst eine Problemumgehung.

1. Installieren Sie die neuste Version von [wskdebug](https://github.com/apache/openwhisk-wskdebug) und optional [ngrok](https://www.npmjs.com/package/ngrok) von GitHub.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Nehmen Sie in der JSON-Datei Ergänzungen an Ihren Benutzereinstellungen vor. Der alte Visual Studio Code-Debugger wird weiterhin verwendet. Der neue hat [verschiedene Probleme](https://github.com/apache/openwhisk-wskdebug/issues/74) mit wskdebug: `"debug.javascript.usePreview": false`.
1. Schließen Sie alle über `aio app run` geöffneten App-Instanzen.
1. Stellen Sie den neuesten Code mit `aio app deploy` bereit.
1. Führen Sie nur das Asset Compute-Entwickler-Tool mit `aio asset-compute devtool` aus. Lassen Sie es geöffnet.
1. Fügen Sie im Visual Studio Code-Editor die folgende Debug-Konfiguration zu `launch.json` hinzu:

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

1. Wählen Sie `wskdebug worker` in der Ausführungs-/Debug-Konfiguration aus und klicken Sie auf das Wiedergabesymbol. Warten Sie auf den Start, bis die Meldung **[!UICONTROL Ready for activations]** (Bereit für Aktivierungen) im Fenster der **[!UICONTROL Debugging-Konsole]** angezeigt wird.

1. Klicken Sie im Entwickler-Tool auf **[!UICONTROL Run]**. Sie können die im Visual Studio Code-Editor ausgeführten Aktionen sehen und die Protokolle werden angezeigt.

1. Legen Sie einen Breakpoint in Ihrem Code fest. Führen Sie den Befehl anschließend erneut aus. Alles sollte wie gewünscht funktionieren.

Alle Code-Änderungen werden in Echtzeit geladen und werden wirksam, sobald die nächste Aktivierung erfolgt.

>[!NOTE]
>
>In benutzerdefinierten Programmen sind für jede Anfrage zwei Aktivierungen vorhanden. Die erste Anfrage ist eine Web-Aktion, die sich im SDK-Code asynchron aufruft. Die zweite Aktivierung verwendet Ihren Code.
