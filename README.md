# GitHub MCP Server - Anleitung und Beispiele

## Was ist der GitHub MCP Server?

Der **GitHub MCP Server** ist der offizielle Model Context Protocol Server von GitHub. Er ermöglicht es AI-Assistenten wie Claude, direkt mit GitHub zu interagieren - Repositories zu durchsuchen, Issues zu erstellen, Pull Requests zu verwalten und vieles mehr.

## Installation und Einrichtung

### Voraussetzungen

1. **GitHub Personal Access Token (PAT)**
   - Gehe zu [GitHub Settings > Developer settings > Personal access tokens](https://github.com/settings/tokens)
   - Klicke auf "Generate new token" (Classic)
   - Wähle die benötigten Scopes:
     - `repo` (für Repository-Zugriff)
     - `read:org` (für Organisation-Zugriff)
     - `gist` (optional, für Gist-Unterstützung)
   - Kopiere den generierten Token (wird nur einmal angezeigt!)

2. **Docker Desktop**
   - Docker Desktop muss installiert und gestartet sein
   - Der GitHub MCP Server sollte bereits in Docker Desktop verfügbar sein

3. **LM Studio**
   - Lade LM Studio herunter von [lmstudio.ai](https://lmstudio.ai)
   - Installiere LM Studio (verfügbar für Windows, macOS und Linux)

### Konfiguration in LM Studio

LM Studio unterstützt MCP Server ab **Version 0.3.0**.

## Option 1: Docker Container nutzen (Empfohlen, da bereits vorhanden)

#### Schritt 1: Docker Container in Docker Desktop starten

1. **Öffne Docker Desktop**
2. Gehe zu **Containers** (linke Seitenleiste)
3. Finde den **GitHub MCP Server** Container
4. Stelle sicher, dass er **läuft** (grüner Status)
   - Falls nicht: Klicke auf **Start/Play-Button**

**Wichtig:** Notiere dir den **Port**, auf dem der Container läuft (z.B. `3000` oder `8080`)

#### Schritt 2: Umgebungsvariablen im Container setzen

**Option A: Über Docker Desktop UI**

1. Klicke auf den Container-Namen
2. Gehe zum Tab **"Inspect"** oder **"Config"**
3. Unter **Environment Variables** füge hinzu:
   - `GITHUB_PERSONAL_ACCESS_TOKEN=dein_token_hier`
4. Starte den Container neu

**Option B: Über Docker Compose (falls verwendet)**

Erstelle oder bearbeite `docker-compose.yml`:

```yaml
services:
  github-mcp-server:
    image: ghcr.io/modelcontextprotocol/server-github:latest
    ports:
      - "3000:3000"  # Port anpassen falls nötig
    environment:
      - GITHUB_PERSONAL_ACCESS_TOKEN=dein_token_hier
    restart: unless-stopped
```

Dann:
```bash
docker-compose up -d
```

**Option C: Über Docker CLI**

```bash
docker run -d \
  --name github-mcp-server \
  -p 3000:3000 \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=dein_token_hier \
  ghcr.io/modelcontextprotocol/server-github:latest
```

#### Schritt 3: LM Studio mit Docker Container verbinden

`https://lmstudio.ai/docs/app/mcp`

**Konfiguration:**
- **Server Name:** `github`
- **Transport:** `http` oder `sse`
- **URL:** `http://localhost:3000` (Port anpassen falls anders)
- **Headers:** (optional)
  ```json
  {
    "Authorization": "Bearer dein_github_token"
  }
  ```

#### Schritt 4: Container-Status überprüfen

**Im Terminal/PowerShell:**

```bash
# Container-Logs anzeigen
docker logs github-mcp-server

# Container-Status prüfen
docker ps | grep github-mcp-server

# In den Container einsteigen (für Debugging)
docker exec -it github-mcp-server sh
```

## Option 2: NPX-basiert (Alternative ohne Docker)

Falls du den Docker-Container nicht nutzen möchtest: 

#### Schritt 1: MCP Server Settings öffnen

`https://lmstudio.ai/docs/app/mcp`

#### Schritt 2: GitHub MCP Server hinzufügen

Füge folgende Konfiguration hinzu:

**Option A: Über die UI (grafische Oberfläche)**

- **Server Name:** `github`
- **Command:** `npx`
- **Arguments:** `-y @modelcontextprotocol/server-github`
- **Environment Variables:**
  - Key: `GITHUB_PERSONAL_ACCESS_TOKEN`
  - Value: `DEIN_GITHUB_TOKEN_HIER`

**Option B: Über Config-Datei (falls verfügbar)**

Öffne die LM Studio Konfigurationsdatei (je nach Version):

**Windows:**
```
%USERPROFILE%\.lmstudio\config.json
```
oder
```
%APPDATA%\LMStudio\config.json
```

**macOS:**
```
~/Library/Application Support/LMStudio/config.json
```

**Linux:**
```
~/.config/lmstudio/config.json
```

Füge im Abschnitt `mcpServers` oder `servers` folgendes hinzu:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "DEIN_GITHUB_TOKEN_HIER"
      }
    }
  }
}
```

**Wichtig:** Ersetze `DEIN_GITHUB_TOKEN_HIER` mit deinem tatsächlichen GitHub Token!

#### Schritt 3: Server aktivieren

1. **Speichere** die Konfiguration
2. **Aktiviere** den GitHub Server (Toggle/Schalter in der UI)
3. **Starte LM Studio neu** (falls erforderlich)

### Server starten und testen

#### Wenn du Docker nutzt:

1. **Stelle sicher, dass der Container läuft:**
   ```bash
   docker ps | grep github-mcp-server
   ```

2. **Teste den HTTP-Endpunkt:**
   ```bash
   curl http://localhost:3000/health
   ```
   oder im Browser: `http://localhost:3000`

3. **Öffne einen Chat** in LM Studio
4. **Lade ein Modell**, das Tool-Calling unterstützt (z.B. Llama 3.1 8B Instruct, Mistral, Qwen)
5. **Teste die Verbindung** mit einem einfachen Prompt:
   ```
   Zeige mir die README von github/github-mcp-server
   ```

#### Wenn du NPX nutzt:

1. **Öffne einen Chat** in LM Studio
2. **Lade ein Modell**, das Tool-Calling unterstützt (z.B. Llama 3.1 8B Instruct, Mistral, Qwen)
3. Der GitHub MCP Server sollte automatisch im Hintergrund starten
4. **Teste die Verbindung** mit einem einfachen Prompt wie:
   ```
   Zeige mir die README von github/github-mcp-server
   ```

**Hinweise für LM Studio:**
- MCP Support ist noch relativ neu in LM Studio
- Nicht alle Features sind möglicherweise verfügbar
- Überprüfe die [LM Studio Changelog](https://lmstudio.ai/changelog) für Updates

## Verfügbare Funktionen

Der GitHub MCP Server bietet folgende Hauptfunktionen:

### 1. Repository-Management
- Repositories durchsuchen
- Repository-Details abrufen
- Dateien lesen und bearbeiten
- Commits anzeigen

### 2. Issue-Management
- Issues erstellen
- Issues durchsuchen
- Issues kommentieren
- Issues schließen/öffnen

### 3. Pull Request-Verwaltung
- PRs erstellen
- PRs reviewen
- PRs zusammenführen
- PRs kommentieren

### 4. Branch-Operationen
- Branches erstellen
- Branches löschen
- Branch-Informationen abrufen

## Praktische Beispiele

### Beispiel 1: Repository durchsuchen

**Prompt für LM Studio:**
```
Zeige mir die README-Datei vom Repository "microsoft/vscode"
```

**Was passiert:**
- LM Studio nutzt den GitHub MCP Server
- Ruft die README.md-Datei ab
- Zeigt dir den Inhalt formatiert an

### Beispiel 2: Issue erstellen

**Prompt für LM Studio:**
```
Erstelle ein Issue in meinem Repository "meinname/mein-projekt" mit:
- Titel: "Bug: Login funktioniert nicht"
- Beschreibung: "Beim Versuch sich anzumelden, erscheint ein 500 Fehler"
- Labels: "bug", "priority-high"
```

**Was passiert:**
- Das LLM erstellt automatisch das Issue über den MCP Server
- Setzt die Labels
- Gibt dir die URL zum neu erstellten Issue

### Beispiel 3: Code-Review durchführen

**Prompt für LM Studio:**
```
Analysiere den aktuellen Code in der Datei src/main.py 
im Repository "meinname/projekt" und schlage Verbesserungen vor
```

**Was passiert:**
- Das Modell liest die Datei über die GitHub API
- Analysiert den Code
- Gibt konkrete Verbesserungsvorschläge

### Beispiel 4: Pull Request erstellen

**Prompt für LM Studio:**
```
Erstelle einen Pull Request von dem Branch "feature/neue-funktion" 
zum main Branch mit dem Titel "Feature: Neue Suchfunktion hinzugefügt"
```

**Was passiert:**
- Das LLM erstellt den PR über den MCP Server
- Fügt eine Beschreibung hinzu
- Verlinkt relevante Issues

### Beispiel 5: Repository-Statistiken

**Prompt für LM Studio:**
```
Gib mir eine Übersicht über mein Repository "meinname/projekt":
- Anzahl der offenen Issues
- Anzahl der offenen Pull Requests
- Letzte Commits
```

**Was passiert:**
- Das Modell sammelt die Informationen über den MCP Server
- Präsentiert sie übersichtlich strukturiert

## Erweiterte Nutzung

### Mehrere Repositories verwalten

Du kannst LM Studio bitten, über mehrere Repositories hinweg zu arbeiten:

```
Vergleiche die README-Dateien von "facebook/react" und "vuejs/vue" 
und zeige mir die Unterschiede in der Dokumentationsstruktur
```

### Automatisierte Workflows

**Beispiel: Code-Review-Workflow**

```
1. Analysiere alle offenen Pull Requests in "meinname/projekt"
2. Prüfe den Code auf potenzielle Sicherheitsprobleme
3. Erstelle Kommentare mit Verbesserungsvorschlägen
```

### Issue-Triage

```
Gehe durch alle offenen Issues in meinem Repository 
und kategorisiere sie nach:
- Bugs
- Feature-Requests  
- Fragen
- Dokumentation
```

## Sicherheitshinweise

⚠️ **Wichtig für die Sicherheit:**

1. **Token-Sicherheit:**
   - Teile deinen GitHub Token **niemals** öffentlich
   - Nutze Tokens mit minimalen notwendigen Berechtigungen
   - Rotiere Tokens regelmäßig

2. **Zugriffsrechte:**
   - Der MCP Server kann nur auf Repositories zugreifen, für die dein Token berechtigt ist
   - Private Repositories sind nur mit entsprechenden Token-Rechten zugänglich

3. **Datenschutz:**
   - Sei vorsichtig beim Teilen von Screenshots oder Logs
   - Sie könnten sensible Repository-Informationen enthalten

## Fehlerbehandlung

### Docker Container läuft nicht

**Problem:** Container startet nicht oder stoppt sofort

**Lösungen:**

1. **Logs prüfen:**
   ```bash
   docker logs github-mcp-server
   ```

2. **Port-Konflikt prüfen:**
   ```bash
   # Welcher Prozess nutzt Port 3000?
   netstat -ano | findstr :3000
   # Oder anderen Port verwenden
   docker run -p 8080:3000 ...
   ```

3. **Container neu starten:**
   ```bash
   docker restart github-mcp-server
   ```

4. **Container komplett neu aufsetzen:**
   ```bash
   docker stop github-mcp-server
   docker rm github-mcp-server
   # Dann neu starten mit docker run...
   ```

### LM Studio verbindet sich nicht mit Docker Container

**Problem:** LM Studio zeigt keine GitHub-Funktionen an

**Lösungen:**

1. **Container-Erreichbarkeit testen:**
   ```bash
   curl http://localhost:3000
   ```
   Sollte eine Antwort geben (nicht 404 oder Connection Refused)

2. **Firewall prüfen:**
   - Windows: Docker Desktop muss durch die Firewall dürfen
   - Überprüfe, ob localhost:3000 erreichbar ist

3. **LM Studio Transport-Typ prüfen:**
   - Stelle sicher, dass du **HTTP** oder **SSE** als Transport gewählt hast
   - **Nicht** STDIO (das ist für lokale Prozesse)

4. **Network-Modus prüfen:**
   - Docker Container sollte im **bridge** oder **host** Modus laufen
   - Im host-Modus: direkter Zugriff auf localhost
   ```bash
   docker run --network host ...
   ```

### Server startet nicht (NPX-basiert)

**Problem:** LM Studio zeigt keine GitHub-Funktionen an oder MCP Server ist nicht verfügbar

**Lösungen:**
1. Überprüfe die Konfiguration in den LM Studio Settings
2. Stelle sicher, dass Node.js installiert ist (`node --version` im Terminal)
3. Prüfe, ob der GitHub Token gültig ist
4. Schaue in die LM Studio Logs/Console:
   - In LM Studio: Developer Tools öffnen (falls verfügbar)
   - Terminal-Ausgabe beim Start beachten
5. Teste den Server manuell im Terminal:
   ```bash
   GITHUB_PERSONAL_ACCESS_TOKEN=dein_token npx -y @modelcontextprotocol/server-github
   ```

### Authentifizierungsfehler

**Problem:** "Unauthorized" oder ähnliche Fehler

**Lösungen:**
1. Token-Berechtigungen überprüfen
2. Neuen Token generieren
3. Token in der Konfiguration aktualisieren

### Rate Limiting

**Problem:** "API rate limit exceeded"

**Lösungen:**
- GitHub API hat Limits (5000 Anfragen/Stunde für authentifizierte Nutzer)
- Warte eine Stunde oder nutze einen anderen Token
- Optimiere deine Anfragen (weniger API-Calls)

## Best Practices

### 1. Spezifische Anfragen stellen

❌ **Schlecht:**
```
Zeig mir was über mein Repository
```

✅ **Gut:**
```
Zeige mir die letzten 5 Commits im main Branch 
von Repository "meinname/projekt"
```

### 2. Batch-Operationen

Statt einzelner Anfragen:
```
Analysiere alle offenen PRs und gib mir eine Zusammenfassung 
mit Prioritätsempfehlungen
```

### 3. Templates nutzen

Erstelle Issue-Templates:
```
Erstelle ein Bug-Report Issue mit folgender Struktur:
- Beschreibung
- Schritte zur Reproduktion
- Erwartetes Verhalten
- Tatsächliches Verhalten
- Screenshots
- Umgebung
```

## Tipps und Tricks

### Schnelle Repository-Insights

```
Gib mir einen Report über "facebook/react":
- Hauptprogrammiersprachen
- Anzahl Contributors
- Letzte Aktivität
- Populärste Issues
```

### Code-Suche

```
Suche in allen meinen Repositories nach Dateien, 
die "TODO" oder "FIXME" Kommentare enthalten
```

### Dependency-Analyse

```
Analysiere die package.json in "meinname/projekt" 
und prüfe auf veraltete Dependencies
```

## Weitere Ressourcen

- **Offizielle MCP Dokumentation:** [modelcontextprotocol.io](https://modelcontextprotocol.io)
- **GitHub MCP Server Repository:** [github.com/github/github-mcp-server](https://github.com/github/github-mcp-server)
- **GitHub API Dokumentation:** [docs.github.com/rest](https://docs.github.com/rest)
- **MCP Server Community:** [GitHub Discussions](https://github.com/orgs/modelcontextprotocol/discussions)

## Zusammenfassung

Der GitHub MCP Server ist ein mächtiges Werkzeug, das die Interaktion mit GitHub über AI-Assistenten ermöglicht. Mit der richtigen Konfiguration kannst du:

✅ Repositories effizienter verwalten
✅ Issues und PRs schneller bearbeiten  
✅ Code-Reviews automatisieren
✅ Entwicklungs-Workflows optimieren
✅ Zeit bei wiederkehrenden GitHub-Aufgaben sparen

**Viel Erfolg beim Einsatz des GitHub MCP Servers!** 🚀
