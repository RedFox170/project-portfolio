# Enterprise Text2SQL Framework – KI-gestützte RAG-Infrastruktur

> **Rechtlicher Hinweis:** Da dieses Framework als Abschlussprojekt im Rahmen meines Praktikums und meiner Berufsqualifikation zum *Fachinformatiker für Anwendungsentwicklung* entstanden ist, unterliegt der Quellcode der proprietären Geheimhaltung des Praxisbetriebs. Konkrete Codebeispiele können daher nicht veröffentlicht werden. Die nachfolgende Dokumentation dient als technischer Deep-Dive in die Systemarchitektur, die Kern-Workflows und die gelösten Software-Hürden.

---

## 🏢 Projektübersicht & Core-Konzept

Klassische Ansätze zur Datenabfrage in Enterprise-Umgebungen scheitern oft an der Komplexität relationaler Datenstrukturen. Wer Ad-hoc-Analysen benötigt, steht meist vor hunderten kryptischen Tabellennamen und verschachtelten Beziehungen. 

Dieses Framework löst das Problem durch ein intelligentes **Retrieval-Augmented Generation (RAG)** System auf Tabellenebene. Es entkoppelt den Endanwender vollständig von der SQL-Syntax: Über ein intuitives Web-Frontend werden Fragen in natürlicher deutscher Sprache eingegeben, das System ermittelt dynamisch den relevanten Kontext, generiert lokal den präzisen SQL-Code, führt diesen sicher aus und liefert die Daten tabellarisch zurück.


## 📺 Live-Demo & Intelligentes Error-Handling

Das folgende Video zeigt das React-Frontend im Live-Betrieb. Hierbei wird ein praxistypisches Szenario demonstriert: die fehlertolerante Suche und das automatische Abfangen von leeren Abfrageergebnissen.

1. **Erster Versuch ("Max Mustermann"):** Es wird nach einem fiktiven Namen gesucht, der in der Enterprise-Datenbank nicht existiert. Das System führt den vollständigen RAG-Workflow aus, stellt fest, dass die Ergebnismenge der SQL-View leer ist, und fängt dies sauber ab, ohne dass die Pipeline abstürzt oder kryptische SQL-Fehler ausgibt.
2. **Zweiter Versuch (Realer Datenbestand):** Die Suche nach dem tatsächlich existierenden Namen triggert das semantische Retrieval. Die Vektordatenbank isoliert die passenden Tabellen, das LLM generiert die korrekte `WHERE`-Klausel und das Frontend rendert die Ergebnistabelle in Echtzeit.

<video width="100%" controls>
  <source src="demo_suche.mp4" type="video/mp4">
  Ihr Browser unterstützt das Video-Tag nicht.
</video>

*Hinweis: Das begleitende GIF `ragdemo.gif` im Repository zeigt den vollständigen, ungekürzten Workflow von der deutschsprachigen Nutzerfrage bis zur final validierten Ausgabe noch einmal in der Endlosschleife.*

### Die Vorteile dieses RAG-Ansatzes gegenüber klassischem Fine-Tuning:
* **Keine Modell-Starrheit:** Änderungen am Datenmodell erfordern kein neues, teures KI-Training.
* **Kontext-Optimierung (Prompt-Slicing):** Das System „impft“ das LLM ausschließlich mit den Tabellen-Steckbriefen, die für die spezifische Frage relevant sind.
* **Datensouveränität:** Die gesamte Infrastruktur läuft lokal und self-hosted – zu 100 % DSGVO-konform.

---

## 📐 System-Architektur & Interaktionsfluss

Die Anwendung ist nach dem Prinzip der **Separation of Concerns (Strikte Trennung der Zuständigkeiten)** designt und operiert über asynchrone, eventgesteuerte Pipelines.

1. **Eingabe-Schicht:** Ein modulares **React-Frontend** (erstellt mit Vite, TypeScript und TailwindCSS) nimmt die natürlichsprachliche Frage des Nutzers entgegen und sendet sie per gesicherter REST-Schnittstelle (Webhook) an die Orchestrierungsschicht.
2. **Orchestrierungs- & Logik-Schicht (n8n):** Das Herzstück bildet eine Event-Pipeline in **n8n**. Sie fängt den Request ab, stößt die Semantik-Suche an, steuert die lokale KI-Inferenz und validiert die Ergebnisse.
3. **Vektor-Schicht (PostgreSQL + pgvector):** Dient als semantisches Gedächtnis des Systems. Hier liegen die mathematischen Koordinaten (Embeddings) aller Tabellen-Steckbriefe für das präzise Context-Retrieval.
4. **KI-Inferenz-Schicht (LM Studio):** Stellt die Rechenleistung für das Embedding-Modell (**BGE-M3**) und das Core-Sprachmodell (LLM) bereit. Komplett self-hosted und offline betreibbar.
5. **Ausführungs-Schicht (MySQL):** Eine hochskalierte relationale Enterprise-Testdatenbank (über 600 Spalten, 71 Tabellen), auf der die generierten Queries via Read-Only-Schnittstelle ausgeführt werden.

---

## 🛠️ Die 3 Kern-Workflows (n8n Implementierung)

### 1. DBCommentWriter – Autonome Schema-Dokumentation
Da Enterprise-Datenbanken im Produktivalltag selten sauber dokumentiert sind, analysiert dieser Workflow die physische Tabellenstruktur sowie isolierte Beispieldatensätze auf Spaltenebene. Ein autonomer KI-Agent übersetzt technische, kryptische Bezeichnungen vollautomatisch in verständliche, semantische Kommentare direkt in der Datenbank.
* **Impact:** Der initiale Dokumentationsaufwand für 71 Tabellen sank von über einem Arbeitstag auf ca. **8 Minuten**.

### 2. SchemaSync – Intelligenter Hash-Vergleich & Vector-Indexing
Um unnötige Rechenzeit und LLM-Inferenz-Kosten zu eliminieren, berechnet der Workflow bei jedem Start den **SHA-256-Hash** des aktuellen physischen DB-Schemas und gleicht ihn mit dem letzten Sync-Stand ab.
* *Fall A (Identisch):* Sofortiger Abbruch, null Rechenleistung verbraucht.
* *Fall B (Unterschiedlich/Erster Durchlauf):* Nur die modifizierten Tabellen werden durch das LLM neu als kompakter Steckbrief formuliert, via LM Studio vektorisiert und in `pgvector` indiziert. Das System bleibt ohne manuelle Pflege permanent aktuell.

### 3. Text2SQL – Kontextbasiertes Retrieval & RAG-Abfrage
Gibt ein Nutzer eine Frage ein, wird diese ebenfalls ad hoc vektorisiert. Per mathematischer Abstandsmessung im Vektorraum isoliert die Vektordatenbank exakt die betroffenen Tabellen-Steckbriefe. Das LLM erhält ausschließlich diesen minimierten Kontext. Dadurch ist das System in der Lage, selbst hochgradig komplexe Abfragen über **4 bis 5 verschachtelte Tabellen-Verknüpfungen (JOINs)** fehlerfrei und in unter 10 Sekunden zu generieren.

---

## 🚀 Technische Herausforderungen & Software-Engineering-Lösungen

### 🛡️ Enterprise-Sicherheit & SQL-Injections
* **Herausforderung:** Die direkte Ausführung von KI-generiertem Code auf Produktivdatenbanken birgt massive Risiken (z. B. zerstörerische DML/DDL-Befehle wie `DROP TABLE` oder unbefugter Datenabfluss).
* **Lösung:** Ein kompromissloses *Defense-in-Depth*-Sicherheitskonzept. In der n8n-Pipeline wurde ein deterministischer Validierungsknoten (Regex-Filter) integriert, der destruktive SQL-Befehle sofort abfängt. Zudem besitzt der Datenbank-User des Frameworks ausschließlich restriktive `SELECT`-Rechte auf speziell dafür angelegte, isolierte **SQL-Views**. Das System hat zu keinem Zeitpunkt direkten Schreibzugriff auf Tabellen.

### 🔄 Stabilität bei Datenbank-Wechseln (Echte DB-Agnostizität)
* **Herausforderung:** Koppelt man n8n-Knoten oder die KI starr an ein spezifisches SQL-Dialekt-Modul, bricht die Pipeline zusammen, sobald die Ziel-Datenbank gewechselt wird (z. B. MS SQL oder PostgreSQL statt MySQL).
* **Lösung:** Die n8n-Pipelines wurden vollständig abstrahiert und dynamisiert. Tabellennamen, Schemata und JOIN-Logiken werden rein generisch als Variablen verarbeitet. Das Framework wurde so konzipiert, dass es nativ und mit minimalen Anpassungen auf jedem beliebigen relationalen SQL-System einsatzbereit ist.

### 🔄 LLM-Fehlertoleranz bei komplexen JOINs (Self-Correction Loop)
* **Herausforderung:** Trotz präzisem Kontext neigen LLMs bei tief verschachtelten relationalen Strukturen gelegentlich zu Syntaxfehlern oder halluzinierten Spaltennamen.
* **Lösung:** Implementierung eines automatisierten Fehler-Validierungs-Loops. Schlägt die Ausführung eines Queries auf der SQL-View fehl, fängt n8n die exakte SQL-Fehlermeldung ab, spiegelt sie zusammen mit dem fehlerhaften Code zurück an das LLM (*Self-Correction Loop*) und erzwingt einen korrigierten Zweitversuch. Die Erfolgsquote stieg dadurch im Testbetrieb auf nahezu 100 %.

---

## 🎓 Learnings & persönliches Fazit

Dieses Framework bildete den technologischen Kern meines Abschlussprojekts zur IHK-Berufsqualifikation. Die Entwicklung bot die perfekte Gelegenheit, moderne KI-Infrastrukturen mit klassischem Software-Engineering zu verknüpfen.

### Wichtigste technische Erkenntnisse:
* **Workflow-Abstraktion:** Die saubere Entkopplung von Daten- und Logikschichten in n8n hat mir gezeigt, wie wertvoll wartbarer und flexibler Pipeline-Code im produktiven Umfeld ist.
* **Sicherheitsbewusstsein:** Die Arbeit an der Schnittstelle zwischen KI-Generierung und Live-Datenbanken hat mein Verständnis für proaktive Sicherheitsarchitekturen (*Defense-in-Depth*) extrem geschärft.
* **RAG-Effizienz:** Die mathematische Funktionsweise von Vektordatenbanken (`pgvector`) in der Praxis zu implementieren, war eine wertvolle Erfahrung, um die Grenzen konventioneller Suchalgorithmen zu überwinden.

Das Projekt war für mich der ideale Abschluss meiner Ausbildung. Es hat unglaublich viel Spaß gemacht, eine so komplexe Architektur von der ersten Idee über das Frontend bis zur sicheren Backend-Pipeline eigenständig zu planen, zu bauen und erfolgreich zu validieren.