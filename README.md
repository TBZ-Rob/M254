# Restaurantbestellprozess mit Camunda 8

Projektarbeit M254. Modellierung und Ausführung eines Restaurantbesuchs als
BPMN Prozess in Camunda 8, mit lokalem Webinterface als Kundensicht.

## Idee

Der komplette Ablauf eines Restaurantbesuchs, vom Betreten über Bestellung und
Essen bis zur Zahlung, ist als ausführbarer BPMN Prozess modelliert. Ein Gast
steuert den Ablauf über eine Weboberfläche. Camunda 8 führt den Prozess Schritt
für Schritt aus und ist in Operate live nachvollziehbar.

## Aufbau

Die BPMN Collaboration besteht aus drei Pools:

* **Kunde** (`Process_0t4z7yo`): Ausführbarer Hauptprozess, steuert den ganzen Ablauf.
* **Kellner** (`Process_14ksr0u`): Beschreibt die Interaktionen des Kellners.
* **Küche** (`Process_kueche`): Beschreibt die Abläufe in der Küche.

Der Kunde Prozess ist der ausführbare Kern. Kellner und Küche kommunizieren über
BPMN Nachrichten mit ihm. Der Correlation Key dafür ist `bestellId`.

## Komponenten

* **`Camunda_Projektarbeit.bpmn`**: Das Prozessmodell (Collaboration mit drei Pools).
* **`server.js`**: Node.js Backend. Spricht per REST API (v2) mit Camunda 8,
  arbeitet die Service Tasks als Job Worker ab und treibt Kellner und Küche über
  Nachrichten an.
* **`public/index.html`**: Weboberfläche (Kundensicht). Bestellung starten,
  Getränk, Essen und Dessert wählen, zahlen. Kellner und Küche laufen automatisch
  im Hintergrund.

## Technischer Ablauf

1. Der Gast startet über das Webinterface eine Bestellung. Camunda erzeugt eine
   Prozessinstanz des Kunde Prozesses.
2. Service Tasks (zum Beispiel Restaurant betreten, Bestellung aufgeben, bezahlen)
   werden vom Backend als Job Worker automatisch abgearbeitet.
3. Nutzereingaben (Personenzahl, Getränk, Essen, Dessert, Zahlungsart, Trinkgeld)
   laufen über User Tasks und die passenden Formulare im Webinterface.
4. Kellner und Küche werden über BPMN Nachrichten synchronisiert, die das Backend
   zum richtigen Zeitpunkt publiziert.
5. In Operate ist live sichtbar, wie der Token durch den Prozess wandert.

## Voraussetzungen

* **Node.js 18+** (für globales `fetch`)
* **Camunda 8 Run** läuft lokal (Operate erreichbar unter http://localhost:8080/operate)
* Das **BPMN ist deployed** (aktuelle Version im Modeler deployen)

## Starten

```powershell
# 1. Camunda 8 Run starten (im Verzeichnis des Bundles)
.\start.bat
# warten bis "Ready to accept connections on port 8080"

# 2. BPMN im Camunda Modeler deployen (Raketensymbol)

# 3. Backend und Webinterface starten
npm install
npm start
```

Dann im Browser: http://localhost:3000

Beim Start prüft das Backend die Verbindung und gibt im Terminal aus:

```
Verbindung zu Camunda ok (http://localhost:8080/v2), Broker: 1
Job Worker gestartet
Webinterface: http://localhost:3000
```

## Wichtigste Prozessvariablen

* `bestellId`: Correlation Key für alle Nachrichten (automatisch gesetzt).
* `getraenk`, `hauptgericht`: Auswahl des Gasts.
* `vorspeiseGewuenscht`, `dessertGewuenscht`: Steuern die Verzweigungen.
* `zahlungsart`, `trinkgeldProzent`: Zahlung und Trinkgeld.

## Technologie

Camunda 8.9 · BPMN 2.0 · Node.js · Express · Camunda REST API v2