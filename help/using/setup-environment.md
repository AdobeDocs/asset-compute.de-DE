---
title: Legen Sie die erforderliche Entwicklungsumgebung für  [!DNL Asset Compute Service] fest
description: Einrichten einer Entwicklungsumgebung für [!DNL Asset Compute Service] , um benutzerdefinierten Code zu erstellen und zu testen.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: db38b9dc27505aa7e04cf58a646005fc2e0e8782
workflow-type: tm+mt
source-wordcount: '359'
ht-degree: 90%

---

# Einrichten einer Entwicklungsumgebung {#create-dev-environment}

Befolgen Sie diese Anforderungen und Anweisungen, um eine Entwicklungsumgebung für [!DNL Asset Compute Service] einzurichten.

1. [Erhalten Sie Zugriff auf und Anmeldeinformationen](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) für [!DNL Adobe Developer App Builder].

1. [Richten Sie die lokale Umgebung](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) und die erforderlichen Tools ein.

1. Einige weitere Tools, die Ihnen den reibungslosen Einstieg in die Entwicklung erleichtern, sind:

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, ungerade Versionen werden nicht empfohlen) und [NPM](https://www.npmjs.com). OSX HomeBrew-Benutzende können `brew install node` ausführen, um beides zu installieren. Andernfalls laden Sie das Tool von der [NodeJS-Download-Seite](https://nodejs.org/de/) herunter
   * Als IDE, die für NodeJS geeignet ist, empfiehlt Adobe [Visual Studio Code (VS Code)](https://code.visualstudio.com), da dies die unterstützte IDE für den Debugger ist. Sie können jede andere IDE als Code-Editor verwenden, eine erweiterte Verwendung (z. B. Debugger) wird jedoch noch nicht unterstützt.
   * Installieren Sie die neueste Version von Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Stellen Sie sicher, dass Sie die [Voraussetzungen](/help/using/understand-extensibility.md#prerequisites-and-provisioning) erfüllen

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Einrichten eines App Builder-Projekts {#create-App-Builder-project}

1. Stellen Sie sicher, dass in der [!DNL Experience Cloud]-Organisation eine Systemadmin- oder Entwicklerrolle vorhanden ist. Diese Rolle wird von Systemadmins in [Admin Console](https://adminconsole.adobe.com/overview) eingerichtet.

1. Melden Sie sich bei der [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) an. [!DNL Experience Cloud]Stellen Sie sicher, dass Sie Teil derselben Organisation sind wie die [!DNL Experience Manager] as a [!DNL Cloud Service]-Integration. Weitere Informationen zu Adobe Developer Console finden Sie in der [Dokumentation zur Konsole](https://developer.adobe.com/developer-console/docs/guides/).

1. [Erstellen eines App Builder-Projekts](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Klicken Sie auf **[!UICONTROL Neues Projekt erstellen]** > **[!UICONTROL Projekt aus Vorlage]**. Wählen Sie App Builder aus. Ein neues App Builder-Projekt mit zwei Arbeitsbereichen wird erstellt: `Production` und `Stage`. Fügen Sie bei Bedarf zusätzliche Arbeitsbereiche hinzu, z. B. `Development`.

1. Wählen Sie im App Builder-Projekt einen Arbeitsbereich aus und abonnieren Sie die Services, die für Asset Compute erforderlich sind. Klicken Sie auf **Add to Project** > **API** und fügen Sie die Services `Asset Compute`, `IO Events` und `IO Events Management` hinzu. Beim Hinzufügen der ersten API werden Sie aufgefordert, einen privaten Schlüssel zu erstellen. Speichern Sie diese Informationen auf Ihrem Computer, da Sie diesen Schlüssel zum Testen Ihrer benutzerdefinierten Programme mit dem Entwickler-Tool benötigen.

   >[!NOTE]
   >
   >JWT ist veraltet und der private Schlüssel kann nicht heruntergeladen werden. Beachten Sie bei der Aktualisierung der Testwerkzeuge, dass mit OAuth erstellte benutzerdefinierte Mitarbeiter zwar bereitgestellt werden können, Entwicklungstools jedoch nicht funktionieren.

## Nächster Schritt {#next-step}

Nachdem Sie Ihre Umgebung eingerichtet haben, können Sie [eine benutzerdefinierte Anwendung erstellen](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
