# Logbuch

Pro Etappe ein Eintrag. Kurz reicht. Das Logbuch ist Teil der Abgabe und Grundlage eurer Demo.
One entry per stage. Short is fine. The log is part of the submission and the basis for your demo.

## Etappe 0

- **Was wir gemacht haben:** Vorlage „struktur-oekologie-gruppe-XX“ im Repo übernommen. Projektordner mit Python-`.venv` aufgesetzt. Noch zu ergänzen: genaue Schritte und Paketversionen.
- **Wichtigste Prompts** (wörtlich kopieren):
- **Was schiefging und woran wir es gemerkt haben:**
- **Was wir selbst geprüft haben** (von Hand, im Plan, im Modell): Das Modell i Rhino geöffnet, um zu sehen ob es funktioniert und wie es aufgebaut ist. 


# Sprint 1: Carbon-Inventur Pavillon Stockerau

## Etappe 1: Modell lesen

**Test:** Wie viele Stäbe hat der Pavillon? Prüft die Zahl am Modell nach.
Rhino-Datei `Data/Stockerau Rhino Export from RFEM.3dm` mit `rhino3dm` (aus der `.venv`) gelesen. Objekte nach Layer und Geometrietyp gezählt. 
Ergebniss: 2 986 Objekte, 736 Stab-Linien auf den Profil-Layern (Flat Bar, RD 12, RO 139.7×8.0, Rectangle 500/500). Zusätzlich 561 Breps als zweite Darstellung, die nicht gezählt werden.
Prüfung: Zählung über Linien mit Layern verglichen; Brep-Anzahl je Profil-Layer gegengeprüft.

**Learning oder Besonderheit:**
Was schief gelaufen ist: Erste Abfragen über die Shell liefen nicht zuverlässig, Ausgaben wurden gekürzt. Gelöst durch Ausgabe in Dateien. Eine Zählung über alle Breps wurde verworfen, weil Linie und Brep denselben Stab darstellen.
Eine Darstellung im Modell kann doppelt vorkommen (Linie und Körper). Die Zuordnung über den Layer ist eindeutig, das Zählen über den Geometrietyp fehleranfällig.

## Etappe 2: Querschnitte verstehen

**Test:** Rechnet einen Stab, dessen Querschnitt sich über die Länge ändert, von Hand nach und vergleicht mit eurem Programm.
Querschnittsfläche aus den Profilnamen abgeleitet (Flachstab b×h, Rechteck, Rundstab, Rohr mit A = π/4·(D²−(D−2t)²)). Volumen = A × L. Zusätzlich geprüft, ob die Brep-Körper prismatisch sind.
Ergebniss: 736 Stäbe mit Fläche und Volumen in `stab_querschnitt_volumen.csv`; Gesamtvolumen 12,70 m³.
Prüfung:Handrechnung für STAB-0001 (162,5 × 12 mm, L = 0,389693 m): A = 1 950 mm², V = 0,00075990 m³.

**Learning oder Besonderheit:**
Was schief gelaufen ist: Die Kantenzählung der Brep-Körper ergab zuerst „kein Quader“ bei allen 561 Körpern, weil die Kanten nicht gleich lang sind. Gelöst durch Volumenberechnung über die acht Eckpunkte. Die Brep-Querschnittsfläche (Volumen/Länge) weicht bei Flachstäben bis rund 27 % vom Profilnennwert ab. Übernommen wurde der Querschnitt aus dem Profilnamen; die Abweichung ist nicht geklärt (Noch zu ergänzen).
Ein Modell ist nicht automatisch prismatisch. Abweichungen muss man dokumentieren, statt sie still zu übernehmen. Veränderliche Querschnitte sind im Modell nicht eindeutig belegt (Noch zu ergänzen).

## Etappe 3: Material zuordnen

**Test:** Sind die Materialangaben, die schon im Modell stehen, plausibel? Zeigt an einem Stab, wie ihr das geprüft habt.
Materialdefinitionen (B+G steel, timber, concrete, glass) und Layer-Zuordnung im Modell ausgelesen. Entscheidungsleiter festgelegt: Regel (Norm im Profilname), Sprachmodell (Profiltyp), eigene Prüfung (Masse).
Ergebniss: Keine Stab-Layer haben ein Material (alle `RenderMaterialIndex = -1`, Objekte erben über `MaterialFromLayer`). Die Materiallayer `03-steel`, `03-timber`, `03_concrete`, `03-glass` enthalten keine Geometrie. In `stab_material.csv`: 12 Stäbe RO (Regel, EN 10210), 689 Stäbe Stahl (Sprachmodell), 47 Rechteckstäbe „unklar (Vermutung Stahlbeton)“.
Prüfung: Beispielstab STAB-0001: Material „Baustahl (Sprachmodell)“, Masse 5,97 kg bei 7 850 kg/m³. Materialien auf Layer-Ebene im Modell geprüft.

**Learning oder Besonderheit:**
Was schief gelaufen ist: Eine erste Summenangabe für Flachstäbe war falsch (351 + 116 statt 502). Korrigiert nach Neuzählung der CSV.
Die Materialangaben im Modell sind für die Stäbe nicht belegt. Plausibilität ist nur aus Profilnamen ableitbar. Empfehlung: Materialzuweisung im Modell nachholen.

## Etappe 4: Embodied Carbon aus der ÖKOBAUDAT

**Test:** Rechnet einen Stab von seiner Länge bis zu kg CO₂e (A1–A3) von Hand nach.
ÖKOBAUDAT-Prozesse live über die API abgerufen: „Baustahl: Profile und Stabstahl“ (UUID `20c29cbd-f9a2-4518-a1ce-c681246c29c8`, Bezug 1 000 kg, GWP-total) und „Betonstahl in Stäben“ (UUID `a8ebf8f2-2fcc-4431-b730-93bf5efd8111`, Bezug 1 kg). Kg CO₂e je Stab und Modul berechnet, Gesamttabelle `stab_gesamttabelle.csv` erstellt.
Ergebniss. A1–A3 Stahl 4 152,15 kg CO₂e; Betonstahl-Annahme für 47 Rechteckstäbe 592,55 kg CO₂e; Summe A1–A3 4 744,70 kg CO₂e.
Prüfung: Handrechnung STAB-0001: 5,9652 kg ÷ 1 000 kg × 560,287 kg CO₂e = 3,3422 kg (Datei: 3,34224). Summen der Gesamttabelle gegen die Einzelauswertung abgeglichen.

**Learning oder Besonderheit:**
Was schief gegangen ist: Betonstahl-Modulnamen weichen ab (`A1|Standard Szenario`, `A2`, `A3`), die erste Auswertung lieferte dadurch 0. Eine Spaltenbezeichnung in der Gesamttabelle war falsch und wurde korrigiert. Die Suchparameter der API filtern nicht; gelöst durch vollständiges Laden und lokales Filtern (7 050 Einträge).
Die Bezugseinheit des Prozesses muss man vor der Rechnung prüfen (1 000 kg bzw. 1 kg). Ein Prozess kann eine andere Methode nutzen (GWP statt GWP-total); das ist vor einem Vergleich zu klären.


## Etappe 5: Web-App

**Test:** Schaltet beim Stahl auf die Alternative um. Was erwartet ihr vorher, was passiert, und warum?
Web-App `webapp/` gebaut: Three.js lokal (`vendor/`), 3D-Modell mit 736 Zylindern, Einfärbung nach kg CO₂e, Umschalter Standard / Alternative (XCarb) / Was-wäre-wenn, Modul-Auswahl A1–A3 / C / D, Stabgruppe, Stabliste mit vier Reitern, Gesamtbilanz, CO₂-Diagramm, Szenarien (localStorage), Optimierung. Generator `build_data.py` erzeugt `data.js`; `server.py` startet den lokalen Server auf `http://localhost:8000/`. Lokaler Branch `webapp-cache` mit Commit `646366b`.
Eigene Überprüfungen: Generator-Summen gegen `stab_gesamttabelle.csv` (Abweichungsprüfung im Skript); Handrechnung STAB-0001. Nicht geprüft: Syntax aller JavaScript-Dateien und Browserlauf (Noch zu ergänzen).

**Learning oder Besonderheit:**
Was schief gegangen ist:
  - Die Shell lieferte bei vielen Befehlen keine Ausgabe; Prüfungen wurden in Dateien umgeleitet, teils blieben sie ungelöst.
  - `node` war in der Shell nicht verfügbar, deshalb keine Syntaxprüfung der JavaScript-Dateien.
  - Der GitHub-Push ist nicht bestätigt: `git ls-remote` zeigt nur `main`.
  - XCarb-Prozesse nutzen die Methode GWP, nicht GWP-total; der Alternativ-Datensatz hat nur C3 und C4.
  - Die Optimierung rechnete zuerst A1–A3 plus C; korrigiert auf A1–A3.
  - Ein Generatorlauf brach wegen eines fehlenden `return` ab; behoben.
Learning / Besonderheiten:
  - Die Alternative (XCarb) ist kein direkter Ersatz: andere Methode, nur Teilmodule.
  - Bauteilgruppen (Stützen, Träger, Dach) sind in den Daten nicht vorhanden; die Näherung über die Stabausrichtung ist gekennzeichnet.
  - Holz- und Betonmaterialien für den Was-wäre-wenn-Modus fehlen: keine verifizierten ÖKOBAUDAT-Werte vorhanden (Noch zu ergänzen).
  - Offen: Dropdown für weitere Materialien und dynamisches Stab-Diagramm.
  - Modul D bleibt getrennt, weil die Gutschrift die Bilanz sonst verzerrt.


## Zum Schluss
Wie habt ihr mit dem Agenten gearbeitet, und was würdet ihr der nächsten Gruppe mitgeben?
Arbeitsweise mit dem Agenten: Am Anfang war es recht schwierig mit dem Agenten zu arbeiten, vor allem ihm genaue Prompts zu geben. Wir haben uns deshalb von der KI genaue Prompts schreiben lassen, die wir dann anschließend den Agenten gegeben haben. 
Was wir der nächsten Gruppe mitgeben: Materialzuweisungen im Rhino-Modell abkären, genaue/detailierte Prompts und zuerst eine für sich erstellte Zusammenfassung machen bevor man die Web-App erstellt. 