# Heizlogik mit Override

Dieses Repository enthält Home-Assistant-Blueprints für eine **manuell freundliche Heizungssteuerung**.

Die Heizungslogik basiert auf einem einfachen Prinzip:
Die Heiz-Saison wird automatisch berechnet (z. B. anhand der Monate),
kann aber jederzeit **bewusst und manuell** über einen Override aktiviert werden.

Manuelle Änderungen am Thermostat werden respektiert.
Es gibt keine erzwungenen Zeitpläne und keine verpflichtende Anwesenheitslogik.
Zeitpläne können optional verwendet werden, sind aber nicht erforderlich.

Optional können Fensterkontakte berücksichtigt werden – **mit Verzögerung**,
damit normales Lüften nicht zu unerwünschtem Abschalten der Heizung führt.

[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](
https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/xXJimnyCricketXx/heizlogik-mit-override/main/blueprints/automation/heizlogik_mit_override.yaml
)

---

## Für wen ist dieses Projekt gedacht?

Dieses Projekt richtet sich an:

- Home-Assistant-Nutzer, die **keine aggressive Heizungsautomatik** möchten
- Haushalte, in denen **manuelle Kontrolle wichtig** ist
- Nutzer, die verstehen wollen, **was ihre Heizung warum tut**
- Smart-Home-Einsteiger, die eine **nachvollziehbare Lösung** suchen
  
Nicht primär gedacht ist diese Logik für:
- vollständig automatische Anwesenheitssteuerung
- sehr komplexe Wochenzeitpläne mit vielen täglichen Umschaltpunkten
- Systeme, die permanent Temperaturen nachregeln sollen

---

## Grundidee & Konzept

Die Heizungssteuerung besteht aus **zwei klar getrennten Ebenen**:

### 1. Globale Heiz-Saison
Die Heiz-Saison gibt vor, **ob grundsätzlich geheizt werden soll**.

- Sie wird automatisch berechnet (z. B. Oktober bis April)
- Kann jederzeit manuell per Override aktiviert werden
- Wird **nicht direkt vom Benutzer geschaltet**

### 2. Raumbezogene Heizlogik
Jeder Raum bekommt eine eigene Automation auf Basis eines Blueprints.

Diese Automation:
- prüft, ob Heiz-Saison aktiv ist
- berücksichtigt optional den Urlaubsstatus
- berücksichtigt optional einen Zeitplan
- setzt Komfort- oder ECO-Temperatur
- reagiert optional auf Fensterkontakte (verzögert)

---

## Voraussetzungen

### Allgemein
- Ein laufender **Home Assistant**
- Grundkenntnisse im Umgang mit Helfern und Automationen

---

### Schritt 1: Globale Helfer anlegen

Diese Helfer werden **einmal zentral** benötigt.

**Navigation in Home Assistant:**
- Einstellungen → Geräte & Dienste
- Reiter **Helfer**
- **„Helfer erstellen“** auswählen

#### Heiz-Saison
- Typ: **Schalter** (`input_boolean`)
- Name: `Heiz-Saison`
- Entity-ID: `input_boolean.heiz_saison`

Dieser Helfer gibt vor, ob grundsätzlich geheizt werden soll.
Er wird **nicht manuell geschaltet**, sondern durch eine Automation gesetzt.

---

#### Heiz-Saison Override
- Typ: **Schalter** (`input_boolean`)
- Name: `Heiz-Saison Override`
- Entity-ID: `input_boolean.heiz_saison_override`

Dieser Helfer ist der **manuelle Eingriff**:
Wenn er eingeschaltet ist, wird die Heiz-Saison unabhängig vom Monat aktiviert.

---

#### Urlaub
- Typ: **Schalter** (`input_boolean`)
- Name: `Urlaub`
- Entity-ID: `input_boolean.urlaub`

Dieser Helfer signalisiert eine längere Abwesenheit.
Räume wechseln dann in den ECO-Betrieb.

---

### Schritt 2: Heiz-Saison automatisch berechnen

Die globale Heiz-Saison wird über eine eigene Automation gesetzt.

**Navigation in Home Assistant:**
- Einstellungen → Automationen & Szenen
- **„Automation erstellen“**
- Rechts oben das **Drei-Punkte-Menü**
- **„In YAML bearbeiten“** auswählen

Eine **kommentierte Beispiel-Automation** findest du hier:

- examples/automation/heiz_saison_berechnen.yaml
  
> 💡 Diese Datei ist **kein Blueprint**, sondern eine bewusst einfache Beispiel-Automation.
> Der Inhalt kann vollständig in den YAML-Editor kopiert und als eigene Automation
> gespeichert und bei Bedarf angepasst werden.


Diese Automation:
- prüft täglich den aktuellen Monat
- berücksichtigt den Override
- setzt `input_boolean.heiz_saison` entsprechend

---

### Schritt 3: Raumbezogene Helfer anlegen

Für **jeden Raum** werden folgende Helfer benötigt:

#### Komfort-Temperatur
- Helfer-Typ: **Zahl** (`input_number`)
- Beispiel: `input_number.schlafzimmer_komfort`
- Einheit: °C
- Minimalwert: z. B. `4`
- Maximalwert: z. B. `30`
- Schrittweite: `0,5`
- Anzeige: **Schieberegler** oder **Eingabe** (beides möglich)

Empfohlen ist ein realistischer Temperaturbereich, damit der Slider
im Dashboard sinnvoll nutzbar ist.

#### ECO-Temperatur
- Helfer-Typ: **Zahl** (`input_number`)
- Beispiel: `input_number.schlafzimmer_eco`
- Einheit: °C
- Minimalwert: z. B. `4`
- Maximalwert: z. B. `30`
- Schrittweite: `0,5`
- Anzeige: **Schieberegler** oder **Eingabe** (beides möglich)

Empfohlen ist ein realistischer Temperaturbereich, damit der Slider
im Dashboard sinnvoll nutzbar ist.

Diese Helfer erlauben es, Temperaturen **direkt über das Dashboard** anzupassen,
ohne Automationen ändern zu müssen.

---

### Schritt 4a: Optional – Fensterkontakte

Wenn Fenster berücksichtigt werden sollen:

- Ein einzelner Fensterkontakt **oder**
- eine **Gruppe aus mehreren Fensterkontakten**

> 💡 **Hinweis bei mehreren Fenstern pro Raum:**  
> Wenn ein Raum mehrere Fenster oder Fensterflügel hat, sollte eine Gruppe
> aus allen relevanten Fensterkontakten erstellt werden.
> Diese Gruppe wird dann im Blueprint ausgewählt.

Die Fensterlogik arbeitet **verzögert**, um normales Lüften nicht zu stören.

### Schritt 4b: Optional – Zeitpläne

Zusätzlich zur manuellen Steuerung können optionale Zeitpläne verwendet werden.

Zeitpläne sind **nicht erforderlich**, können aber sinnvoll sein, z. B.:

- tagsüber ECO-Betrieb, wenn niemand zu Hause ist
- Komfort-Betrieb morgens oder abends
- Vorheizen vor der Rückkehr nach Hause

Dazu wird ein Home-Assistant-Zeitplan-Helfer verwendet:

Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → Zeitplan

- Helfer-Typ: **Zeitplan** (`schedule`)
- Beispiel: `schedule.schlafzimmer_heizen`

Im Blueprint kann festgelegt werden:
- ob ein Zeitplan berücksichtigt werden soll
- welcher Zeitplan verwendet wird

> 💡 Ohne Zeitplan funktioniert die Heizlogik weiterhin vollständig
> manuell in Kombination mit Heiz-Saison und Override.

### Zeitpläne – wichtiges Verständnis

Der Home-Assistant-Zeitplan (`schedule`) steuert **keine Temperaturen**.

Ein Zeitplan kennt nur zwei Zustände:
- **Ein (`on`)**
- **Aus (`off`)**

In dieser Heizlogik bedeutet das:
- **Zeitplan = Ein → Komfort-Temperatur**
- **Zeitplan = Aus → ECO-Temperatur**

**Beispiel:**

Ein Zeitplan mit folgenden aktiven Zeitfenstern:

- 06:00 – 09:00 → Ein (Komfort)
- 17:00 – 23:00 → Ein (Komfort)

führt dazu, dass:
- morgens und abends die Komfort-Temperatur genutzt wird
- außerhalb dieser Zeiten automatisch die ECO-Temperatur aktiv ist

Welche Temperaturen konkret verwendet werden, wird **nicht im Zeitplan**,
sondern über die jeweiligen Temperatur-Helfer (`input_number`) pro Raum festgelegt.

---

## Installation des Blueprints

1. In Home Assistant:
   - **Einstellungen → Automationen & Szenen → Blueprints**
   - **Blueprint importieren**
   - GitHub-URL zur Blueprint-Datei einfügen:

```
https://github.com/xXJimnyCricketXx/heizlogik-mit-override/blob/main/blueprints/automation/heizlogik_mit_override.yaml
```
> 💡 Nach dem Import steht der Blueprint dauerhaft in Home Assistant zur Verfügung
und kann für mehrere Räume wiederverwendet werden.

2. Neue Automation aus dem Blueprint erstellen

3. Entitäten auswählen:
   - Thermostat
   - Heiz-Saison
   - Urlaub
   - Komfort- und ECO-Temperatur
   - optional Fensterkontakt oder Fenster-Gruppe
   - optional Zeitplan wählen

4. Der Automation einen **Raum/Bereich zuweisen** (empfohlen)

---

## Enthaltene Dateien

- `blueprints/automation/heizlogik_mit_override.yaml`  
  → Raumbezogener Heizungs-Blueprint

- `examples/automation/heiz_saison_berechnen.yaml`  
  → Beispiel-Automation zur Berechnung der Heiz-Saison

---

## Lizenz

Dieses Projekt steht unter der **MIT License**.
