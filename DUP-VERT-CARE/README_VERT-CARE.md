# Projekt VERT-CARE

> **Projekt-ID:** vert-care_v02 · **Status aller Standorte:** [Status-Issues](../../../issues?q=milestone%3A%22VERT-CARE%22) · **Eigenen Status anlegen:** [Neues Status-Issue](../../../issues/new?template=vert-care-status.yml)

## (1) FDPG-Umsetzungsinfo (UInfo)

Das Wichtigste zum Projekt im Überblick: [VERT-CARE_Uinfo.docx](VERT-CARE_Uinfo.docx)

## (2) Prüfprotokoll von den Pilot-DIZ

[Pruefprotokoll_zur_Skriptpruefung_Vert-Care-1.docx](Pruefprotokoll_zur_Skriptpruefung_Vert-Care-1.docx)

## (3) Zeitplan

- Frist zur Datenausleitung an die DMSt: **22.01.2027**
- Frist zur Übermittlung der Daten von der DMSt an die Forschenden: 29.01.2027

## (4) Datenextraktion

Dieses Projekt wird mit der FDPG DUP-Pipeline durchgeführt:

- Anleitung DUP-Pipeline: https://medizininformatik-initiative.github.io/dataportal/data-node/dup-pipeline.html
- Anleitung TORCH: https://github.com/medizininformatik-initiative/dataportal/tree/main/data-node/torch

Ausleitungsformat: **CSV**

Projekt-CRTDL: `vert-care_2026-10-30_v05.json` (in [vert-care_2026-10-30.zip](vert-care_2026-10-30.zip))

### (4.1) DIMP konfigurieren

Für die Pseudonymisierung muss das Geburtsdatum auf Jahr und Monat und die PLZ auf die ersten beiden Stellen vergröbert werden. Der `fhir-pseudonymizer` übernimmt dies über eine entsprechende Konfiguration.

- Konfigurationsdatei: `dimp_dup_base.yaml` (im ZIP)
- Anleitung: [Using and Customizing the DUP YAML](https://medizininformatik-initiative.github.io/dataportal/data-node/DIMP.html#using-and-customizing-the-dup-yaml)

In der Vorlage anpassen:

1. Die Bezeichnung für die Domain auf den lokalen Wert setzen.
2. Den CryptoHash für die Pseudonymisierung der technischen ID erzeugen.

Standorte, die diese Konfiguration (oder eine gröbere) bereits im FHIR-Server vorhalten, müssen den `fhir-pseudonymizer` trotzdem neu konfigurieren, und zwar für die Hashes der technischen IDs.

### (4.2) DUP ausführen

Komponenten für FDPG Dataportal 6.2.0, siehe [Kompatibilitätsmatrix](https://medizininformatik-initiative.github.io/dataportal/compatibility-matrix.html). TORCH und Aether in den angegebenen Versionen (oder höher):

- AETHER: [1.4.0](https://github.com/medizininformatik-initiative/aether/releases#release-v1.4.0)
- TORCH: [1.0.1](https://github.com/medizininformatik-initiative/torch/releases#release-v1.0.1)

Konfigurationsdateien (alle im ZIP):

- `vert-care_pipeline.yml`
- `dimp-enrichments.json`
- `flatteningLookup.json`: aus `flattening.zip`, vor dem Ausführen in den Pipeline-Ordner entpacken

```bash
aether pipeline start vert-care_pipeline.yml vert-care_2026-10-30_v05.json
```

### (4.3) Datenvalidierung

Validierung des Consent Handlings: [README](https://github.com/medizininformatik-initiative/Skriptlibrary-FDPG/blob/main/validate_consent/README.md)

```bash
python3 validate_consent_v2.py jobs/[job-hash]/import/[hash].ndjson
```

### (4.4) Kohortenbereinigung

TORCH exportiert nur Daten aus dem Einwilligungszeitraum. Patient:innen, deren einschließender Fall vor der Einwilligung liegt, werden **verpflichtend** mit `filter_by_crtdl.py` aus den CSVs entfernt.

- Skript und Dokumentation: https://github.com/medizininformatik-initiative/Skriptlibrary-FDPG/tree/main/refine_consent_cohort
- Reihenfolge aller Schritte: [ABLAUF_DATA_REFINEMENT.md](https://github.com/medizininformatik-initiative/Skriptlibrary-FDPG/blob/main/ABLAUF_DATA_REFINEMENT.md)

Alle Befehle in 4.4 und 4.5 werden im Projektordner der DUP ausgeführt. Dafür muss die Skriptlibrary auf Ebene der DUP-Ordner liegen (bei bestehendem Klon vorher `git pull`):

```bash
git clone https://github.com/medizininformatik-initiative/Skriptlibrary-FDPG.git
```

```bash
python3 ../Skriptlibrary-FDPG/refine_consent_cohort/filter_by_crtdl.py \
  --crtdl vert-care_2026-10-30_v05.json \
  --csv-dir jobs/[job-hash]/csv \
  --out-dir jobs/[job-hash]/filter_by_crtdl
```

Erwartet ist ein kleiner Anteil Ausgeschlossener (im Pilot unter 1 %, siehe Ende der Konsolenausgabe).

### (4.5) Datenminimierung außerhalb von TORCH

Nach der Kohortenbereinigung werden die CSV-Dateien in zwei weiteren Schritten nachbearbeitet. Die Reihenfolge ist zwingend einzuhalten: Schritt 1 benötigt das Geburtsdatum, das in Schritt 2 entfernt wird.

**Schritt 1: Patient:innen unter 18 Jahren entfernen** ([GitHub](https://github.com/medizininformatik-initiative/Skriptlibrary-FDPG/tree/main/reduce_patients_by_age))

```bash
python3 ../Skriptlibrary-FDPG/reduce_patients_by_age/reduce_patients_by_age.py \
  --csv-dir jobs/[job-hash]/filter_by_crtdl \
  --out-dir jobs/[job-hash]/reduce_by_age \
  --age-threshold 18
```

> [!WARNING]
> `--age-threshold 18` bitte nicht weglassen. Ohne den Parameter gilt die Altersgrenze 19.

**Schritt 2: Geburtsdatum durch Alter ersetzen** ([GitHub](https://github.com/medizininformatik-initiative/Skriptlibrary-FDPG/tree/main/birthday2age))

```bash
python3 ../Skriptlibrary-FDPG/birthday2age/fhir_age.py \
  "jobs/[job-hash]/reduce_by_age/MII PR Person Patient (Pseudonymisiert)_include_reduced.csv" \
  "jobs/[job-hash]/reduce_by_age/MII PR Person Patient (Pseudonymisiert)_include_reduced_AGE.csv"
```

Das Alter wird zum Stichtag 09/2026 berechnet. Der Stichtag soll nicht verändert werden, damit alle Standorte vergleichbar rechnen.

### (4.6) Datenausleitung an die DMSt

> [!IMPORTANT]
> Auszuleiten sind ausschließlich die Dateien aus `jobs/[job-hash]/reduce_by_age/`:
> - `MII PR Person Patient (Pseudonymisiert)_include_reduced_AGE.csv`
> - alle übrigen `*_include_reduced.csv`, außer Consent

Nicht auszuleiten:

- `MII PR Person Patient (Pseudonymisiert)_include_reduced.csv` (enthält das Geburtsdatum)
- die Consent-CSV, auch als `_include_reduced.csv` (nur für die Validierung notwendig)
- alle Dateien aus `csv/` und `filter_by_crtdl/`
- Berichte und Protokolle: `to_be_excluded.csv`, `inclusion_criteria_matches.csv`, `code_matches.csv`, `removed_patient.csv`, `*.log`

## (5) Status und Rückfragen

1. [Neues Status-Issue anlegen](../../../issues/new?template=vert-care-status.yml), einmal pro Standort.
2. Erledigte Schritte im Issue abhaken.
3. Probleme und Fragen als Kommentar im Issue festhalten. Bei Rückfragen bitte **@bkabaoglu** oder **@kleinertp** markieren.

## Änderungsverlauf

| Datum | Änderung |
|---|---|
| 30.10.2026 | CRTDL v05 |
| 18.11.2026 | Kohortenbereinigung (4.4) ergänzt, neue Version von `reduce_patients_by_age` mit Parametern |
