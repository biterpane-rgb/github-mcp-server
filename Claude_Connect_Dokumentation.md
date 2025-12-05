# Claude Connect - API-Integration für DOCUframe

## Übersicht
Das Makro `_yit_PP_Claude_Connect` ermöglicht die direkte Kommunikation mit der Anthropic Claude API aus DOCUframe-Makros heraus.

## Funktionssignatur

```
STRING _yit_PP_Claude_Connect(STRING eingabe = "Was ist DOCUframe")
```

**Parameter:**
- `eingabe` (STRING): Die Frage/Anfrage an Claude
  - Optional, Standardwert: "Was ist DOCUframe"
  - Kann beliebigen Text enthalten

**Rückgabe:**
- `STRING`: Die Antwort von Claude oder eine Fehlermeldung

## API-Konfiguration

### Endpoint
```
URL: https://api.anthropic.com/v1/messages
Port: 443 (HTTPS)
Timeout: 30 Sekunden
```

### Modell
```
Model: claude-sonnet-4-5-20250929
Max Tokens: 1024
```

### Authentifizierung
Die Authentifizierung erfolgt über einen API-Key im HTTP-Header:
```
x-api-key: sk-ant-api03-...
anthropic-version: 2023-06-01
content-type: application/json
```

⚠️ **WICHTIG:** Der API-Key sollte aus Sicherheitsgründen nicht im Code gespeichert werden!

## Request-Format

Das Makro sendet folgendes JSON an die API:

```json
{
  "model": "claude-sonnet-4-5-20250929",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": "[eingabe]"
    }
  ]
}
```

## Response-Format

Die Claude API antwortet mit folgendem JSON-Format:

### Erfolgreiche Antwort
```json
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hier ist die Antwort von Claude..."
    }
  ],
  "model": "claude-sonnet-4-5-20250929",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 15,
    "output_tokens": 120
  }
}
```

### Fehlerantwort
```json
{
  "type": "error",
  "error": {
    "type": "authentication_error",
    "message": "invalid x-api-key"
  }
}
```

## Fehlerbehandlung

Das Makro behandelt folgende Fehlerfälle:

### 1. Keine API-Antwort
```
Fehler: Keine Antwort von der API erhalten
```
- Tritt auf bei Netzwerkproblemen oder Timeouts

### 2. API-Fehler
```
API-Fehler: [Fehlermeldung]
```
- Authentifizierungsfehler
- Rate Limits überschritten
- Ungültige Anfragen

### 3. Ungültiges Response-Format
```
Fehler: 'content' ist kein Array oder leer
Fehler: 'text'-Feld nicht gefunden in Content-Array
Fehler: Unerwartete API-Antwort
```

## Code-Ablauf

```
┌─────────────────────────────┐
│ Start: eingabe übergeben    │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ JSON-Request erstellen      │
│ - Model: claude-sonnet-4-5  │
│ - Messages: [user, eingabe] │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ HTTP POST Request senden    │
│ - URL: api.anthropic.com    │
│ - Timeout: 30 Sekunden      │
│ - HTTPS (Port 443)          │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│ Response erhalten?          │
└──────┬──────────────┬───────┘
       │ Nein         │ Ja
       │              │
       ▼              ▼
   ┌─────┐    ┌──────────────┐
   │Error│    │ JSON parsen  │
   └─────┘    └──────┬───────┘
                     │
                     ▼
              ┌──────────────┐
              │ "content"    │
              │ vorhanden?   │
              └──┬───────┬───┘
           Nein  │       │ Ja
                 │       │
                 ▼       ▼
            ┌─────┐  ┌──────────────┐
            │Error│  │ Text aus     │
            └─────┘  │ content[0]   │
                     │ extrahieren  │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │ MessageBox   │
                     │ anzeigen     │
                     └──────┬───────┘
                            │
                            ▼
                     ┌──────────────┐
                     │ RETURN       │
                     │ antwort      │
                     └──────────────┘
```

## Verwendungsbeispiele

### Beispiel 1: Einfache Frage
```
STRING frage = "Was ist DOCUframe?";
STRING antwort = _yit_PP_Claude_Connect(frage);
Trace("Claude antwortet: " + antwort);
```

### Beispiel 2: Dokumentation generieren
```
STRING code = "BOOL ProcessAction( Ausgabeprozessstatus &Status )...";
STRING frage = "Erkläre folgenden Code:\n" + code;
STRING dokumentation = _yit_PP_Claude_Connect(frage);
```

### Beispiel 3: Fehleranalyse
```
STRING fehlerlog = GetLastError();
STRING frage = "Analysiere diesen Fehler und schlage Lösungen vor: " + fehlerlog;
STRING loesung = _yit_PP_Claude_Connect(frage);
MessageBox(loesung);
```

### Beispiel 4: Code-Review
```
STRING meinCode = "...";
STRING frage = "Führe ein Code-Review durch und finde potenzielle Bugs: " + meinCode;
STRING review = _yit_PP_Claude_Connect(frage);
```

## Logging

Das Makro erzeugt folgende Trace-Ausgaben:

```
_yit_PP_Claude_Connect - Start Portal Login
_yit_PP_Claude_Connect - Anfrage: [eingabe]
_yit_PP_Claude_Connect - Antwort erfolgreich erhalten
_yit_PP_Claude_Connect - Ende Portal Login
```

Bei Fehlern:
```
_yit_PP_Claude_Connect - Fehler: [Fehlermeldung]
_yit_PP_Claude_Connect - Response: [API-Response]
```

## Verbesserungsvorschläge

### 1. API-Key aus Konfiguration laden
**Problem:** API-Key ist fest im Code
**Lösung:**
```
STRING apiKey = GetConfigValue("Claude", "APIKey");
STRING loginHeader = "x-api-key: " + apiKey + " \n\r anthropic-version: 2023-06-01 \n\r content-type: application/json";
```

### 2. Max Tokens als Parameter
**Aktuell:** Fest auf 1024 begrenzt
**Verbesserung:**
```
STRING _yit_PP_Claude_Connect(STRING eingabe = "Was ist DOCUframe", INT maxTokens = 1024)
```

### 3. Modellauswahl
**Aktuell:** Fest auf claude-sonnet-4-5
**Verbesserung:**
```
STRING _yit_PP_Claude_Connect(STRING eingabe = "Was ist DOCUframe", STRING model = "claude-sonnet-4-5-20250929")
```

### 4. Retry-Logik bei Netzwerkfehlern
```
INT maxRetries = 3;
FOR(INT i = 0; i < maxRetries; i++)
  HTTPGetResponse(...);
  IF(loginResponse != "")
    BREAK;
  ENDIF
  Sleep(1000 * (i + 1)); // Exponential Backoff
ENDFOR
```

### 5. Response-Caching
Um API-Kosten zu sparen:
```
HASHMAP<STRING, STRING> responseCache;
IF(responseCache.HasKey(eingabe))
  RETURN(responseCache.Get(eingabe));
ENDIF
// ... API-Call ...
responseCache.Set(eingabe, antwort);
```

## Kosten & Limits

### API-Kosten (Claude Sonnet 4.5)
- **Input:** $3.00 / Million Tokens
- **Output:** $15.00 / Million Tokens

**Beispielrechnung:**
- 100 Anfragen à 50 Input-Tokens, 200 Output-Tokens
- Input: (100 × 50) / 1.000.000 × $3.00 = $0.015
- Output: (100 × 200) / 1.000.000 × $15.00 = $0.30
- **Gesamt: $0.315**

### Rate Limits
- Abhängig vom API-Plan
- Standard: 50 Anfragen/Minute

## Sicherheitshinweise

⚠️ **KRITISCH:**
1. **API-Key nicht im Code speichern**
   - Verwende Konfigurationsdateien
   - Verschlüssele sensible Daten
   - Nutze Umgebungsvariablen

2. **Input-Validierung**
   - Prüfe `eingabe` auf schädliche Inhalte
   - Verhindere Injection-Angriffe
   - Limitiere Eingabelänge

3. **Logging**
   - Logge KEINE sensitiven Daten
   - API-Keys niemals in Traces ausgeben

4. **Fehlerbehandlung**
   - Gebe keine internen Fehlerdetails an Endbenutzer weiter
   - Verhindere Information Disclosure

## Changelog

### Version 2.0 (2025-12-05)
- ✅ Vollständige JSON-Response-Verarbeitung implementiert
- ✅ Umfassende Fehlerbehandlung hinzugefügt
- ✅ Korrekte Rückgabe (`antwort` statt `accessToken`)
- ✅ API-Error-Handling implementiert
- ✅ Detailliertes Logging hinzugefügt
- ✅ Response-Validierung verbessert

### Version 1.0 (Original)
- ❌ Unvollständiger JSON-Zugriff
- ❌ Falsche Rückgabe (accessToken)
- ❌ Keine Fehlerbehandlung
