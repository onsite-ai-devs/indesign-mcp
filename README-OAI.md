# OAI-Quellstand — Provenienz, Audit, Betrieb

> Onsite.ai-Fork von `Rinellasky/indesign-mcp`. **Das Upstream-Repo enthält keinen Quellcode**
> (nur README + Installer); der ausführbare Stand existiert dort ausschließlich als
> Release-Artefakte. Dieser Fork checkt die **extrahierten, auditierten Quellen** ein und ist
> die einzige Bezugsquelle für Onsite.ai-Installationen (Setup-Skill `/oai-marketing:indesign-setup`).

## Provenienz

| Komponente | Ordner | Herkunft |
|---|---|---|
| MCP-Server (Python, 99 Tools) | `server/` | extrahiert aus `indesign-mcp.dxt`, Release **v1.2.1** (Upstream-HEAD `7292d7bf9549d85acb530cb1de7fb24a0b6a551f`, 2026-07-28) |
| UXP-Plugin (JS, InDesign-Panel) | `uxp-plugin/` | extrahiert aus `indesign-mcp-plugin.ccx`, Release v1.2.1 (ccx = Zip; Neupacken des Ordnerinhalts ergibt wieder ein installierbares `.ccx`) |
| Proxy (Node, socket.io) | `proxy/` | **nicht** aus dem 44-MB-Release-Binary — vendort aus dem MIT-Upstream `mikechambers/adb-mcp` (`adb-proxy-socket/`), SHA in `proxy/.upstream-sha` (`afe5d09c…`) |

## Audit (2026-08-09, Claude/Fable 5 auf Weisung Lucas Vöhringer)

- Netzwerkziele: ausschließlich `http://localhost:3001` (Server `server/id-mcp.py` Z. 42,
  Plugin `uxp-plugin/main.js` Z. 35); keine sonstigen Außenverbindungen im Code.
- Keine `eval`/`exec`/`subprocess`/`child_process`-Muster in Server, Plugin oder Proxy.
- UXP-Manifest-Berechtigungen: `localFileSystem: fullAccess`, `network.domains: all`
  (UXP-üblich; real genutzt wird nur localhost). InDesign-Mindestversion **20.2.0**.
- **Eigener Patch:** `proxy/proxy.js` bindet jetzt auf `127.0.0.1` statt alle Interfaces
  (Upstream lauschte LAN-weit — unnötige Angriffsfläche).

## Betrieb (Windows, Kurzform — maßgeblich ist der Setup-Skill)

1. **Proxy:** `cd proxy && npm ci && node proxy.js` → lauscht auf `localhost:3001`.
2. **UXP-Plugin:** `uxp-plugin/` als Zip mit Endung `.ccx` packen, Doppelklick → Creative
   Cloud installiert; in InDesign Panel öffnen, *Connect* klicken.
3. **MCP-Server (Claude-Code-Eintrag):** `uv run --with fonttools --with mcp --with "mcp[cli]"
   --with python-socketio --with requests --with numpy --with pillow --with websocket-client
   mcp run <pfad-zu>/server/id-mcp.py` (aus `server/manifest.json`, dort `${__dirname}` =
   `server/`). Voraussetzungen: Node ≥ 20, uv, InDesign ≥ 20.2.

## Regeln

- Upstream-Releases (Binaries) werden **nicht** übernommen; Updates laufen als neuer
  Artefakt-Extrakt + Audit + Commit in diesen Fork.
- Lizenz: MIT (Upstream Rinellasky und mikechambers/adb-mcp) — `LICENSE.md` bleibt unberührt.
