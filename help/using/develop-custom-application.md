---
title: Entwickeln für  [!DNL Asset Compute Service]
description: Erstellen Sie benutzerdefinierte Programme mit  [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: ht
source-wordcount: '1507'
ht-degree: 100%

---

# Entwickeln eines benutzerdefinierten Programms {#develop}

Bevor Sie mit der Entwicklung eines benutzerdefinierten Programms beginnen:

* Stellen Sie sicher, dass alle [Voraussetzungen](/help/using/understand-extensibility.md#prerequisites-and-provisioning) erfüllt sind.
* Installieren Sie die [erforderlichen Softwaretools](/help/using/setup-environment.md#create-dev-environment).
* Lesen Sie auch den Abschnitt [Einrichten der Umgebung](setup-environment.md), um sicherzustellen, dass Sie bereit für die Erstellung eines benutzerdefinierten Programms sind.

## Erstellen eines benutzerdefinierten Programms {#create-custom-application}

Stellen Sie sicher, dass [Adobe aio-cli](https://github.com/adobe/aio-cli) lokal installiert ist.

1. Um ein benutzerdefiniertes Programm zu erstellen, [erstellen Sie ein App Builder-Projekt](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Führen Sie dazu `aio app init <app-name>` im Terminal aus.

   Wenn Sie sich noch nicht angemeldet haben, fordert Sie dieser Befehl in einem Browser auf, sich mit Ihrer Adobe ID bei der [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) anzumelden. Weitere Informationen zum Anmelden von der CLI finden Sie [hier](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli).

   Adobe empfiehlt, dass Sie sich zuerst anmelden. Sollten Sie Probleme haben, befolgen Sie die Anweisungen zum [Erstellen einer App, ohne sich anzumelden](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Befolgen Sie nach dem Anmelden die Anweisungen in der CLI und wählen Sie die Parameter `Organization`, `Project` und `Workspace` für das Programm aus. Wählen Sie das Projekt und den Arbeitsbereich aus, das bzw. den Sie beim [Einrichten der Umgebung](setup-environment.md) erstellt haben. Beantworten Sie die Frage `Which extension point(s) do you wish to implement ?` mit `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. Wenn `Which Adobe I/O App features do you want to enable for this project?` aufgefordert angezeigt wird, wählen Sie `Actions`. Deaktivieren Sie unbedingt die Option `Web Assets`, da Web-Assets andere Authentifizierungs- und Autorisierungsüberprüfungen verwenden.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Beantworten Sie die Frage `Which type of sample actions do you want to create?` mit `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Befolgen Sie die restlichen Anweisungen und öffnen Sie das neue Programm in Visual Studio Code (oder Ihrem bevorzugten Code-Editor). Sie enthält die Strukturvorlage und den Beispiel-Code für ein benutzerdefiniertes Programm.

   Informationen über die [Hauptkomponenten eines App Builder-Programms](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   Die Vorlagenanwendung nutzt das [Asset Compute-SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) von Adobe zum Hochladen, Herunterladen und Orchestrieren von Anwendungsdarstellungen, sodass die Entwickelnden lediglich die benutzerdefinierte Anwendungslogik implementieren müssen. Innerhalb des Ordners `actions/<worker-name>` befindet sich die Datei `index.js`, in der der benutzerdefinierte Programm-Code eingefügt werden kann.

Beispiele und Ideen für benutzerdefinierte Programme finden Sie unter [Beispiele für benutzerdefinierte Programme](#try-sample).

### Hinzufügen von Anmeldeinformationen {#add-credentials}

Bei der Anmeldung während der Erstellung des Programms werden die meisten Anmeldeinformationen von App Builder in Ihrer ENV-Datei gesammelt. Die Verwendung des Entwickler-Tools erfordert jedoch zusätzliche Anmeldeinformationen.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Anmeldeinformationen für die Datenspeicherung des Entwickler-Tools {#developer-tool-credentials}

Das Tool für Entwickelnde zum Auswerten benutzerdefinierter Apps mithilfe des [!DNL Asset Compute service] setzt die Verwendung eines Cloud-Speicher-Containers voraus. Dieser Container ist für die Speicherung von Testdateien sowie für den Empfang und die Darstellung der von den Apps erstellten Ausgabedarstellungen unerlässlich.

>[!NOTE]
>
>Dieser Container ist separat vom Cloud-Speicher von [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. Dies gilt nur für das Entwickeln und Testen mit dem Asset Compute-Entwickler-Tool.

Stellen Sie sicher, dass Sie Zugriff auf einen [unterstützten Cloud-Speicher-Container](https://github.com/adobe/asset-compute-devtool#prerequisites) haben. Dieser Container wird bei Bedarf von verschiedenen Entwickelnden für verschiedene Projekte gemeinsam verwendet.

#### Hinzufügen von Anmeldeinformationen zur ENV-Datei {#add-credentials-env-file}

Fügen Sie die nachfolgenden Anmeldeinformationen für das Entwicklungs-Tool in die `.env`-Datei ein. Die Datei befindet sich im Stammverzeichnis des App Builder-Projekts:

1. So fügen Sie den absoluten Pfad zu der privaten Schlüsseldatei hinzu, die beim Hinzufügen von Services zu Ihrem App Builder-Projekt erstellt wurde:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Laden Sie die Datei über die Adobe Developer Console herunter. Gehen Sie zum Stammverzeichnis des Projekts und klicken Sie in der rechten oberen Ecke auf „Alle herunterladen“. Die Datei wird mit `<namespace>-<workspace>.json` als Dateiname heruntergeladen. Führen Sie einen der folgenden Schritte aus:

   * Benennen Sie die Datei in `console.json` um und verschieben Sie sie in den Stammordner Ihres Projekts.
   * Optional können Sie den absoluten Pfad der JSON-Datei für die Adobe Developer Console-Integration hinzufügen. Hierbei handelt es sich um dieselbe Datei [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user), die Sie in Ihrem Projektarbeitsbereich heruntergeladen haben.

     ```conf
     ASSET_COMPUTE_INTEGRATION_FILE_PATH=
     ```

1. Fügen Sie entweder S3- oder Azure-Speicheranmeldeinformationen hinzu. Sie benötigen nur Zugriff auf eine Cloud-Speicherlösung.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>Die Datei `config.json` enthält Anmeldeinformationen. Fügen Sie die JSON-Datei innerhalb des Projekts zu Ihrer Datei `.gitignore` hinzu, um die Freigabe zu verhindern. Dasselbe gilt für `.env`- und `.aio`-Dateien.

## Ausführen des Programms {#run-custom-application}

Bevor Sie die Anwendung mit dem Asset Compute-Entwickler-Tool ausführen, müssen Sie die [Anmeldeinformationen](#developer-tool-credentials) ordnungsgemäß konfigurieren.

Um das Programm im Entwickler-Tool auszuführen, verwenden Sie den Befehl `aio app run`. Die Aktion wird in Adobe [!DNL I/O Runtime] bereitgestellt und das Entwicklungs-Tool wird auf Ihrem lokalen Computer gestartet. Dieses Tool wird zum Testen von Programmanforderungen während der Entwicklung verwendet. Hier finden Sie ein Beispiel für eine Ausgabedarstellungsanforderung:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>Verwenden Sie das Flag `--local` nicht mit dem Befehl `run`. Es funktioniert nicht mit benutzerdefinierten [!DNL Asset Compute]-Anwendungen und dem Asset Compute-Entwickler-Tool. Benutzerdefinierte Anwendungen werden vom [!DNL Asset Compute]-Service aufgerufen, der nicht auf Aktionen zugreifen kann, die auf den lokalen Entwickler-Computern ausgeführt werden.

Weitere Informationen zum Testen und Debuggen Ihres Programms finden Sie [hier](test-custom-application.md). Wenn Sie mit der Entwicklung Ihres benutzerdefinierten Programms fertig sind, [stellen Sie Ihr benutzerdefiniertes Programm](deploy-custom-application.md) bereit.

## Testen des von Adobe bereitgestellten Beispielprogramms {#try-sample}

Im Folgenden finden Sie Beispiele für benutzerdefinierte Programme:

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [worker-animal-pictures](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Benutzerdefiniertes Vorlagenprogramm {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) ist ein Vorlagenprogramm. Durch einfaches Kopieren der Quelldatei wird eine Ausgabedarstellung generiert. Der Inhalt dieses Programms ist die Vorlage, die Sie bei der Erstellung der aio-App erhalten, wenn Sie `Adobe Asset Compute` auswählen.

Die Programmdatei [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) verwendet das [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview), um die Quelldatei herunterzuladen, die jeweilige Ausgabedarstellungsverarbeitung zu koordinieren und die resultierenden Ausgabedarstellungen zurück in den Cloud-Speicher hochzuladen.

In dem im Anwendungs-Code definierten [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) wird die gesamte Anwendungsverarbeitungslogik ausgeführt. Der Ausgabedarstellungs-Callback in `worker-basic` kopiert einfach den Inhalt der Quelldatei in die Ausgabedarstellungsdatei.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Aufrufen einer externen API {#call-external-api}

Im Programm-Code können Sie externe API-Aufrufe durchführen, um die Programmverarbeitung zu unterstützen. Nachfolgend finden Sie ein Beispiel für eine Anwendungsdatei, die eine externe API aufruft.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Beispielsweise sendet [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) mithilfe der Bibliothek [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) eine Abrufanfrage an eine statische URL von Wikimedia.

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Übergeben benutzerdefinierter Parameter {#pass-custom-parameters}

Sie können benutzerdefinierte Parameter über die Ausgabedarstellungsobjekte übergeben. Sie können innerhalb des Programms in [`rendition` Anweisungen](https://github.com/adobe/asset-compute-sdk#rendition) referenziert werden. Hier finden Sie ein Beispiel für ein Ausgabedarstellungsobjekt:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Dies ist ein Beispiel für eine Anwendungsdatei, die auf benutzerdefinierte Parameter zugreift:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

`example-worker-animal-pictures` übergibt einen benutzerdefinierten Parameter [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39), um zu bestimmen, welche Datei von Wikimedia abgerufen werden soll.

## Unterstützung bei Authentifizierung und Autorisierung {#authentication-authorization-support}

Standardmäßig werden benutzerdefinierte Asset Compute-Anwendungen mit Autorisierungs- und Authentifizierungsprüfungen für das App Builder-Projekt bereitgestellt. Die Aktivierung erfolgt, indem die Anmerkung `require-adobe-auth` in der Datei `manifest.yml` auf `true` gesetzt wird.

### Zugriff auf andere Adobe-APIs {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Fügen Sie die API-Services dem bei der Einrichtung erstellten [!DNL Asset Compute]-Console-Arbeitsbereich hinzu. Diese Services sind Teil des JWT-Zugriffs-Tokens, das von [!DNL Asset Compute Service] erstellt wird. Auf das Token und andere Anmeldeinformationen kann innerhalb des Programmaktionsobjekts `params` zugegriffen werden.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Übergeben von Anmeldeinformationen für Drittanbietersysteme {#pass-credentials-for-tp}

Um Anmeldeinformationen für andere externe Services zu verarbeiten, übergeben Sie diese als Standardparameter für die Aktionen. Sie werden bei der Übertragung automatisch verschlüsselt. Weitere Informationen finden Sie unter [„Erstellen von Aktionen“ im Adobe I/O Runtime-Entwicklerhandbuch](https://developer.adobe.com/runtime/docs/guides/using/creating_actions/). Legen Sie sie dann während der Bereitstellung mithilfe von Umgebungsvariablen fest. Auf diese Parameter kann im Objekt `params` innerhalb der Aktion zugegriffen werden.

Legen Sie die Standardparameter in `inputs` in der Datei `manifest.yml` fest:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

Der Ausdruck `$VAR` liest den Wert aus einer Umgebungsvariablen mit dem Namen `VAR`.

Bei der Entwicklung können Sie den Wert in der lokalen `.env`-Datei zuweisen. Der Grund dafür ist, dass `aio` Umgebungsvariablen aus `.env`-Dateien automatisch zusammen mit den von der Initiierungs-Shell festgelegten Variablen importiert. In diesem Beispiel sieht die `.env`-Datei wie folgt aus:

```CONF
#...
SECRET_KEY=secret-value
```

Bei der Produktionsbereitstellung können die Umgebungsvariablen im CI-System festgelegt werden, z. B. mithilfe von Geheimnissen in GitHub-Aktionen. Greifen Sie auf die Standardparameter im Programm als solche zu:

```javascript
const key = params.secretKey;
```

## Dimensionieren von Programmen {#sizing-workers}

Eine Anwendung wird in einem Container in Adobe [!DNL I/O Runtime] mit [Beschränkungen](https://developer.adobe.com/runtime/docs/guides/using/system_settings/) ausgeführt, die über `manifest.yml` konfiguriert werden können:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Aufgrund der umfangreichen Verarbeitung, die von Asset Compute-Anwendungen durchgeführt wird, müssen Sie diese Beschränkungen anpassen, um eine optimale Leistung (groß genug für binäre Assets) und Effizienz (keine Ressourcenverschwendung durch ungenutzten Container-Speicherplatz) zu erzielen.

Der Standard-Timeout für Aktionen in Runtime ist eine Minute, kann jedoch durch Setzen der `timeout`-Beschränkung (in Millisekunden) erhöht werden. Wenn Sie mit der Verarbeitung größerer Dateien rechnen, erhöhen Sie diese Zeit. Berücksichtigen Sie die Gesamtzeit, die zum Herunterladen der Quelle, zum Verarbeiten der Datei und zum Hochladen der Ausgabedarstellung erforderlich ist. Bei Zeitüberschreitung einer Aktion, wenn also die Aktivierung nicht vor der angegebenen Timeout-Beschränkung zurückgegeben wird, verwirft Runtime den Container und verwendet ihn nicht erneut.

Asset Compute-Anwendungen sind von Natur aus eher an Netzwerk- und Datenträger-E/A-Vorgänge gebunden. Die Quelldatei muss zuerst heruntergeladen werden. Die Verarbeitung ist häufig ressourcenintensiv und die resultierenden Ausgabedarstellungen werden dann erneut hochgeladen.

Mit dem Parameter `memorySize` können Sie den einem Aktions-Container zugewiesenen Speicher in Megabyte festlegen. Derzeit wird mit diesem Parameter auch definiert, inwieweit der Container CPU-Zugriff erhält. Vor allem ist dies aber ein wichtiger Kostenfaktor bei der Verwendung von Runtime (größere Container kosten mehr). Verwenden Sie hier einen größeren Wert, wenn die Verarbeitung mehr Speicher oder CPU erfordert. Achten Sie aber darauf, keine Ressourcen zu verschwenden, denn je größer der Container, desto geringer der Gesamtdurchsatz.

Darüber hinaus ist es möglich, die Parallelität von Aktionen innerhalb eines Containers mithilfe der Einstellung `concurrency` zu steuern. Diese Einstellung entspricht der Anzahl der gleichzeitigen Aktivierungen, die ein einzelner Container (derselben Aktion) erhält. In diesem Modell ähnelt der Aktions-Container einem Node.js-Server, der bis zu diesem Grenzwert mehrere gleichzeitige Anfragen empfängt. Die Standardeinstellung `memorySize` in Runtime ist auf 200 MB eingestellt. Dieser Wert ist ideal für kleinere App Builder-Aktionen. Bei Asset Compute-Anwendungen kann dieser Standardwert aufgrund der stärkeren lokalen Verarbeitung und Festplattenauslastung zu hoch sein. Einige Programme funktionieren je nach Implementierung möglicherweise auch bei gleichzeitigen Aktivitäten nicht gut. Das Asset Compute-SDK stellt sicher, dass Aktivierungen getrennt werden, indem Dateien in verschiedene eindeutige Ordner geschrieben werden.

Testen Sie die Programme, um die optimalen Werte für `concurrency` und `memorySize` zu finden. Größere Container (= höhere Speicherbeschränkung) könnten mehr Parallelität ermöglichen, aber bei geringerem Traffic zu unnötigen Ausgaben führen.
