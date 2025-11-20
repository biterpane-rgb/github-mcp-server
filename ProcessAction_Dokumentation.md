# ProcessAction - Belegdruck und Archivierung

## Übersicht
Die Funktion `ProcessAction` steuert den automatischen Druck und die Archivierung von Belegen basierend auf Belegstufe und -typ.

## Drucker-Konfiguration

### Definierte Drucker
| Variable | UNC-Pfad | Standort | Drucker | Fach |
|----------|----------|----------|---------|------|
| `sPrintLB` | `\\SRLDELAS80\LB-SW` | Lauenburg | LB-SW | Fach 3 |
| `sPrintAB` | `\\SRLDELAS80\AB-Kopie` | Ahrensburg | AB-Kopie | Fach 3 |
| `sPrintLBKopie` | `\\SRLDELAS80\LB-Kopie` | Lauenburg | LB-Kopie | Fach 3 |
| `sPrintLS` | `\\SRLDELAS80\Lieferschein` | Lauenburg | Lieferschein | Fach 2 |

## Drucklogik nach Belegart

### 1. Angebot (Belegstufe 1)
- **1x Standard**: PDF - Email
- **1x Kopie**: Drucken auf LB-SW (Fach 3)

**Implementierung:**
```
CASE 1:
  sPrintername = sPrintLB;  //Kasette3
  BREAK;
```

### 2. Auftragsbestätigung (Belegstufe 2)
- **1x Standard**: PDF - Email
- **1x Kopie**: Drucken auf LB-SW (Fach 3)
- **Zusätzlich**: Disposchein + Arbeitsschein auf LB-SW

**Implementierung:**
```
CASE 2:
  sPrintername = sPrintLB; //Kasette3
  BREAK;
```

**Spezielle Drucke:**
- Disposchein (Formular: "Lieferschein", FormDruckName: "Disposchein")
- Arbeitsschein (Formular: "Lieferschein", FormDruckName: "Arbeitsschein")

### 3. Lieferschein / Empfangsschein (Belegstufe 3)
**Wichtig:** Lieferschein und Empfangsschein werden **zusammen in einem Druckvorgang** auf dem Lieferschein-Drucker ausgegeben.

- **Lieferschein**: 1x Standard auf Lieferschein (Fach 2)
- **Empfangsschein**: 1x Standard auf Lieferschein (Fach 2)
- **Wahlweise**: PDF - Email (für beide)
- **Zusätzlich**: Rechnungsvorschau auf LB-SW (Fach 3)

**Implementierung:**
```
ELSEIF(oBeleg.xGSDFaktBelegstufe == 3 && oBeleg.Belegart.xGSDFaktBelegtyp == 1)
  FormAS.FindKeyStr("Lieferschein");
  FormAS.Get(oForm);
  sPrintername = sPrintLS; //Kasette 2
  //Lieferschein drucken
  PrintForm(oForm,oBeleg,FALSE,sPrintername,iCopies);
  //Empfangsschein direkt im Anschluss auf gleichem Drucker
  _yit_Fakt::FormDruckName = "Empfangsschein";
  PrintForm(oForm,oBeleg,FALSE,sPrintername,iCopies);
  _yit_Fakt::FormDruckName = "";
```

### 4. Rechnung (Belegstufe 4)
- **1x Standard**: PDF - Email
- **2x Kopie**: Drucken auf AB-Kopie (Fach 3)
- **1x Kopie**: Drucken auf LB-Kopie (Fach 3)

**Implementierung:**
```
CASE 4:
CASE 5:
  sPrintername = sPrintAB;  //Kasette3 in AB
  iCopies = 2;
  PrintForm(Status.Formular,oBeleg,FALSE,sPrintername,iCopies);
  sPrintername = sPrintLBKopie;  //Kasette3 in LB
  iCopies = 1;
  BREAK;
```

### 5. Rechnungskorrektur (Belegstufe 5)
**Identisch mit Rechnung (Belegstufe 4)**
- **1x Standard**: PDF - Email
- **2x Kopie**: Drucken auf AB-Kopie (Fach 3)
- **1x Kopie**: Drucken auf LB-Kopie (Fach 3)

## E-Mail-Versand (PDF)

Der E-Mail-Versand wird über den Parameter `OutputEmail` gesteuert:
- Wenn `OutputEmail == "TRUE"`: PDF-Email wird versendet (`bEMail = TRUE`)
- Die E-Mail-PDF-Erzeugung erfolgt zusätzlich zum physischen Druck

## Ablaufdiagramm

```
Start
  |
  ├─> Ist E-Mail aktiviert?
  |     |
  |     ├─> Ja: bEMail = TRUE
  |     └─> Nein: bEMail = FALSE
  |
  ├─> Wenn bEMail = TRUE
  |     |
  |     └─> Switch nach Belegstufe
  |           ├─> Stufe 1: LB-SW
  |           ├─> Stufe 2: LB-SW
  |           ├─> Stufe 3: (wird unten gedruckt)
  |           ├─> Stufe 4/5: AB-Kopie (2x) + LB-Kopie (1x)
  |           └─> Default: LB-SW
  |
  └─> Spezielle Drucklogik
        |
        ├─> Stufe 2: Disposchein + Arbeitsschein
        |
        └─> Stufe 3: Lieferschein + Empfangsschein + Rechnungsvorschau
```

## Besonderheiten

### Admin-Benutzer
Bei `DBGetUserName().ToLower() == "gsdadmin"`:
- Disposchein wird ohne Druckerangabe gedruckt (Standard-Drucker)

### FormDruckName
Die Variable `_yit_Fakt::FormDruckName` steuert die Formularvariantennamen:
- "Disposchein"
- "Arbeitsschein"
- "Empfangsschein"
- "Rechnungsvorschau"

Nach dem Druck wird `FormDruckName` immer auf `""` zurückgesetzt.

## Änderungshistorie

### Version 2.0 (2025-11-20)
- **NEU**: Drucker `LB-Kopie` für Rechnungskopien in Lauenburg hinzugefügt
- **GEÄNDERT**: Rechnung/Rechnungskorrektur druckt jetzt 2x auf AB-Kopie + 1x auf LB-Kopie
- **GEÄNDERT**: Lieferschein und Empfangsschein werden jetzt zusammen in einem Druckvorgang ausgegeben
- **ENTFERNT**: Alter Rechnung-Drucker (`sPrintRE`) wurde entfernt
- **OPTIMIERT**: E-Mail-Logik bei Stufe 3 überarbeitet

### Version 1.0
- Initiale Version mit Grundfunktionalität
