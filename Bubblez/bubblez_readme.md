# 🫧 Bubblez - MERN Stack Social Network

> **💡 Hinweis für Recruiter & Senior-Entwickler:**
> Dieses Repository dokumentiert mein erstes, komplett eigenständiges Fullstack-Projekt, das ich 2024 direkt nach meinem WebDev-Intensivkurs umgesetzt habe. 
> 
> Heute, nach meiner Ausbildung zum **Fachinformatiker für Anwendungsentwicklung (FIAE) im Jahr 2026**, würde ich architektonische und strukturelle Entscheidungen völlig anders treffen (siehe Abschnitt *Reflexion & Learnings*). Ich habe mich bewusst entschieden, den Code im Originalzustand zu belassen, um meinen **persönlichen und fachlichen Entwicklungsprozess** transparent und authentisch aufzuzeigen.

---

## 🎯 Projektübersicht

Klassische Social-Media-Plattformen fokussieren sich oft auf Selbstdarstellung statt auf echten Austausch. **Bubblez** bricht dies auf: Die Anwendung ermöglicht es Nutzern, sich in autarken, themenbasierten Gemeinschaften („Bubbles“) für Vereine, Nachbarschaften oder Freundeskreise zu organisieren. Der Fokus liegt auf lokalem Austausch (Gruppen) und gegenseitiger Unterstützung (Sharing- & Skillshare-Marktplatz).

### 🚀 Kern-Features:
- **Individueller Feed:** Personalisierter Aktivitäts-Stream basierend auf abonnierten Freunden und eigenen „Bubbles“.
- **Gruppenfunktion:** Erstellung, Moderation und Filterung geschlossener oder offener Communities.
- **Community-Marktplatz:** Ein durchsuchbares Verzeichnis für Skillsharing und den Verleih von Ressourcen.
- **Sichere Authentifizierung:** Session- & Cookie-Management mit Echtzeit-Passwortstärkenmessung.
- **Responsive Design:** Konsequenter Mobile-First-Ansatz für alle Endgeräte.

---

## 💻 Tech Stack

Dieses Projekt wurde als klassische **MERN-Stack**-Anwendung mit einer strikten Client-Server-Trennung (REST-API) umgesetzt:

### Frontend
- **React.js** (via Vite für performantes Tooling)
- **Tailwind CSS** & **Heroicons** für ein modernes, responsives UI/UX-Design
- **React Router DOM** für das clientseitige Routing
- **JS-Cookie** für das Handling der Client-Authentifizierung

### Backend
- **Node.js** & **Express.js** als stabiles Server-Framework
- **MongoDB** mit **Mongoose** für die relationenähnliche Datenmodellierung (Schemata)
- **JWT (JSON Web Tokens)** & **Bcrypt** für sicheres Passwort-Hashing und Session-Management
- **Zxcvbn** zur client- und serverseitigen Validierung der Passwortstärke
- **Cloudinary API** für das datenbank-schonende Hosting hochgeladener Beitragsbilder

---

## 🧠 Reflexion & Learnings (Stand 2026)

Durch meine intensive FIAE-Ausbildung in den letzten zwei Jahren habe ich einen tiefen Einblick in Enterprise-Architekturen, Clean Code und moderne Best Practices erhalten. Bei einem heutigen Refactoring oder Neuaufbau würde ich folgende Kernpunkte implementieren:

1. **Typensicherheit mit TypeScript:** Anstelle von reinem JavaScript würde ich das Projekt komplett in TypeScript umsetzen, um Fehler bereits zur Compile-Zeit abzufangen und eine sauberere Schnittstellendokumentation zu garantieren.
2. **Architektur & Skalierbarkeit (Clean Architecture):** Die Trennung von Concerns würde ich strikter durchziehen. Im Frontend durch die Auslagerung von Logik in Custom Hooks, im Backend durch eine klare Aufteilung in Controller-, Service- und Data-Access-Schichten anstelle von überladenen Routen-Handlern.
3. **Globales State Management:** Für die komplexe Vernetzung von Profilen, Marktplatz-Einträgen und Feeds würde ich auf eine dedizierte State-Lösung wie *Zustand* oder *Redux Toolkit* setzen, um "Prop-Drilling" und unnötige Re-Renders zu vermeiden.
4. **Automatisierte Teststrategie:** Das Projekt besitzt aktuell keine Testabdeckung. Ich würde Unit-Tests für die Backend-Logik (mit *Vitest*) sowie E2E-Tests für die kritischen User-Flows wie Registrierung und Kauf/Leihprozesse (mit *Playwright*) etablieren.
5. **Modernes UI/UX-Handling:** Während Tailwind das Styling effizient löst, würde ich für Seitenübergänge, das Öffnen von "Bubbles" oder Feedback-Animationen beim Posten Bibliotheken wie *Framer Motion* einbinden, um das App-Feeling flüssiger zu gestalten.