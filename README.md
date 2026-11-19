# Familienplaner für Home Assistant – Teil 1: Das Tablet-Dashboard

Dateien zum Video **„Familienplaner selbst bauen – Teil 1“** von *Homelab für Normalos*.
Termine der ganzen Familie, Essensplan, Einkaufsliste und Aufgaben auf einem Wand-Tablet – lokal in Home Assistant, ohne Abo.

> Getestet mit Home Assistant 2026.11. Beispielfamilie: Mama, Papa, Mia und Leo.

## Inhalt

| Datei | Wofür |
|---|---|
| `packages/familienplaner.yaml` | Eingabefelder und Abläufe („Termin speichern“, „Essen eintragen“) |
| `themes/familienplaner.yaml` | Farbschema |
| `dashboard.yaml` | das Dashboard (6 Seiten: Heute, Woche, Monat, Aufgaben, Essen, Eintragen) |
| `packages/familienplaner_tablet2.yaml` + `dashboard_tablet2.yaml` | nur für ein **zweites Tablet** (eigene Eingabefelder) |

## Schritt für Schritt

1. **Erweiterungen installieren** – Einstellungen → Marketplace (bei älteren Versionen: HACS):
   Mushroom, Layout Card, card-mod, Calendar Card Pro, Week Planner Card, Kiosk Mode. Danach Seite neu laden.
   Meldung „GitHub-Limit erreicht“? Kein Fehler – etwas später weitermachen.
2. **Kalender anlegen** – Einstellungen → Geräte & Dienste → Integration hinzufügen → „Lokaler Kalender“:
   `Familie`, `Mama`, `Papa`, `Mia`, `Leo`, `Essensplan`.
3. **Listen anlegen** – gleiche Stelle, Integration „Lokales To-do“:
   `Einkaufsliste`, `Familien-To-dos`, `Aufgaben Mama`, `Aufgaben Papa`, `Aufgaben Mia`, `Aufgaben Leo`.
4. **Schulferien (optional)** – Integration „Externer Kalender“ mit einem Ferien-Kalender (ICS-Link) für dein Bundesland, Name `Schulferien`.
   Danach in `dashboard.yaml` unter den Kalendern (bei `calendar.essensplan`) einen Eintrag `calendar.schulferien` ergänzen – fehlt der Kalender, aber der Eintrag steht drin, zeigt die Termine-Karte nur einen Fehler.
5. **Pakete und Farbschema einschalten** – in `configuration.yaml` ergänzen (falls noch nicht vorhanden):
   ```yaml
   homeassistant:
     packages: !include_dir_named packages

   frontend:
     themes: !include_dir_merge_named themes
   ```
   Bearbeiten geht am einfachsten mit der App „File editor“ (Einstellungen → Apps).
   Ordner `packages` und `themes` im Konfigurationsordner anlegen, die Dateien hineinkopieren, Home Assistant **neu starten**.
6. **Dashboard anlegen** – Einstellungen → Dashboards → „Dashboard hinzufügen“ → „Neues Dashboard von Grund auf“.
   Als Titel `Familienplaner` eintragen – Home Assistant schlägt dann die Adresse (URL) `dashboard-familienplaner` vor. **So lassen**, dann stimmen die Links zwischen den Seiten. (Eine Adresse ohne Bindestrich lässt Home Assistant nicht zu.) Heißt sie bei dir anders, in `dashboard.yaml` `/dashboard-familienplaner/` durch deine Adresse ersetzen.
   Öffnen → „Dashboard bearbeiten“ → Menü → „Raw-Konfigurationseditor“ → alles ersetzen durch den Inhalt von `dashboard.yaml` → Speichern.
7. **Namen anpassen** – in `dashboard.yaml` und `packages/familienplaner.yaml` per Suchen & Ersetzen:
   `Mia` → Name deines Kindes **und** `mia` → kleingeschrieben, genauso `Leo`/`leo`, `Mama`/`mama`, `Papa`/`papa`.
   Umlaute werden in Home Assistant zu einfachen Buchstaben: aus dem Kalender „Jürgen“ wird `calendar.jurgen`.
   **Wetter:** Das Dashboard nutzt `weather.forecast_home`. Wie deine Wetter-Entität heißt, siehst du unter Einstellungen → Geräte & Dienste → Entitäten (Suche „weather“). Heißt sie anders, `weather.forecast_home` in `dashboard.yaml` ersetzen.
8. **Tablet** – App *Fully Kiosk Browser*, Startseite: `http://<deine-home-assistant-adresse>:8123/dashboard-familienplaner/heute?hide_sidebar`.
   Eigenen Home-Assistant-Benutzer fürs Tablet anlegen (Einstellungen → Personen → „Person hinzufügen“ → „Anmeldung erlauben“): „Administrator“ **aus**, „Nur lokaler Zugriff“ **an**.

## Mehr oder weniger Kinder (oder andere Personen)
Jede Person kommt in den Dateien an festen Stellen vor. Am Beispiel **Leo**:

**Eine Person weglassen** (z. B. nur ein Kind):
1. Kalender „Leo“ und Liste „Aufgaben Leo“ gar nicht erst anlegen.
2. In `dashboard.yaml` jeden Block mit `calendar.leo` löschen – das sind die Zeile `- entity: calendar.leo` bzw. `- calendar.leo` **und** die direkt darunter eingerückten Zeilen (`name:`, `color:`, `accent_color:`, `label:`).
3. In `dashboard.yaml` in der langen Vorschau-Zeile (oben, „Termine“) den Eintrag `('calendar.leo','Leo')` löschen – samt dem Komma, das ihn vom Nachbarn trennt (z. B. `…('calendar.mia','Mia'),('calendar.leo','Leo')]` → `…('calendar.mia','Mia')]`).
4. Auf der Seite „Aufgaben“ die Karte mit `todo.aufgaben_leo` löschen. Die Seite hat vier Spalten (`repeat(4, …)` und `p1 p2 p3 p4`) – bei drei Personen die `4` in `3` ändern und `p4` streichen. War die gelöschte Karte nicht die letzte, bei den übrigen Karten `grid-area` neu durchnummerieren (`p1`, `p2`, `p3`).
5. In `packages/familienplaner.yaml` die Zeile `- Leo` (Auswahl „Für“) und im Ablauf „Termin speichern“ den Eintrag `'Leo': 'calendar.leo',` löschen.

**Eine Person dazunehmen** (z. B. drittes Kind „Emma“):
1. Kalender „Emma“ und Liste „Aufgaben Emma“ anlegen.
2. Überall, wo bei Schritt 2–5 oben Leo vorkommt, den Leo-Block **kopieren** und `Leo`/`leo` durch `Emma`/`emma` ersetzen (eigene Farbe bei `color`/`accent_color` wählen).
3. Aufgaben-Seite: `repeat(4, …)` → `repeat(5, …)`, in `grid-template-areas` ein `p5` (und ein weiteres `bar`) ergänzen und bei der neuen Karte `grid-area: p5` eintragen.

Danach speichern bzw. Home Assistant neu starten (Paket). Tipp: vorher eine Kopie der Dateien machen.

## Zweites Tablet
Wenn zwei Tablets gleichzeitig benutzt werden, braucht jedes eigene Eingabefelder – sonst tippt man im Flur und in der Küche ändert sich das Feld mit.
Dafür `packages/familienplaner_tablet2.yaml` zusätzlich kopieren und `dashboard_tablet2.yaml` als zweites Dashboard mit der Adresse `familienplaner-tablet2` anlegen. Kalender und Listen sind auf beiden Tablets dieselben.

## Weiter geht's
- Teil 2: Einkaufsliste per Telegram-Bot aufs Handy
- Teil 3: Essensplan fotografieren → landet im Kalender

## Lizenz
MIT – siehe `LICENSE`. Nutzung auf eigene Verantwortung.
