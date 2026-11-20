# Anleitung für das FDPG-Team

## Einmalig vor dem Pilot

1. **Labels anlegen:** Unter *Issues → Labels* das Label `status` anlegen. Es gilt für alle Projekte. Die Status-Formulare vergeben es automatisch, aber nur, wenn es bereits existiert.
2. **Projekt-Board (optional):** In der Organisation unter *Projects → New project* ein Board „FDPG-Projekte – Standortstatus“ anlegen, Tabellenansicht, gruppiert nach Milestone. Unter *Workflows → Auto-add to project* alle Issues mit dem Label `status` aufnehmen lassen.

## Neues Projekt anlegen

1. **Ordner anlegen:** `DUP-[PROJEKT]/` mit einer `README.md` nach dem Muster von `DUP-VERT-CARE/README.md`. Dort auch UInfo, Prüfprotokoll und das ZIP mit CRTDL und Konfiguration ablegen.
2. **Status-Formular anlegen:** `.github/ISSUE_TEMPLATE/vert-care-status.yml` kopieren nach `[projekt]-status.yml`. Projektname, Titel, Link zur Anleitung und die Schritte anpassen (nicht benötigte Schritte entfernen, zusätzliche ergänzen). Das Label `status` bleibt.
3. **Milestone anlegen:** Unter *Issues → Milestones → New milestone* einen Milestone mit dem Projektnamen anlegen (genau wie im Link der Projekt-README, z. B. `VERT-CARE`). Als Fälligkeitsdatum die Frist zur Datenausleitung an die DMSt eintragen.
4. **Projekttabelle ergänzen:** Das Projekt in der Tabelle der Haupt-README eintragen.

## Während eines Projekts: neue Status-Issues zuordnen

Formulare können keinen Milestone setzen. Neue Status-Issues daher regelmäßig zuordnen: In der Issue-Liste nach `label:status no:milestone` filtern, die Issues markieren und über *Milestone* dem Projekt zuweisen. Der Milestone zeigt dann den Fortschritt: Ein Standort gilt als fertig, wenn sein Status-Issue geschlossen ist. Am Projektende den Milestone schließen.

## Änderungen während eines Projekts

- **Anleitung ändern:** README im Projektordner anpassen und die Änderung im *Änderungsverlauf* eintragen.
- **Neuer Schritt:** Im Status-Formular ergänzen. Bereits angelegte Issues ändern sich dadurch nicht. Den Schritt dort von Hand ergänzen (Personen mit Schreibrechten können den Issue-Text bearbeiten).

## Hinweise zum öffentlichen Repository

- Kästchen in einem Issue kann abhaken, wer es angelegt hat oder Schreibrechte hat. Deshalb legt jeder Standort sein Status-Issue selbst an. Ein GitHub-Account genügt, Zugriffsrechte sind nicht nötig.
- Alles im Repository ist öffentlich sichtbar: Anleitungen, UInfo, CRTDL, Issues und Kommentare. Keine E-Mail-Adressen, Patientendaten oder Ergebnisse eintragen.
