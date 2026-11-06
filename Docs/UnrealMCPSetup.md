# Unreal MCP v projekte AnimationsTest

Stav overený 2026-11-06 na UE 5.8.4 (`E:\Epic_Games\UE_5.8`).

## Čo je nastavené

| Súbor | Nastavenie |
|---|---|
| `AnimationsTest.uproject` | pluginy `PythonScriptPlugin`, `ModelContextProtocol`, `EditorToolset` = Enabled |
| `Config/DefaultEditorPerProjectUserSettings.ini` | `[/Script/ModelContextProtocolEngine.ModelContextProtocolSettings]` `bAutoStartServer=True`, `ServerPortNumber=8000` |
| `.mcp.json` | server `unreal` -> `http://127.0.0.1:8000/mcp` |

Bez `bAutoStartServer=True` plugin načíta, ale HTTP server sa nespustí (default v kóde je `false`).

## Ako to použiť

1. Otvor projekt v UE 5.8 (dvojklik na `AnimationsTest.uproject`). V logu musí byť `LogModelContextProtocol: Starting MCP server on port 8000`.
2. V Claude Code spusti session v priečinku projektu (načíta `.mcp.json`). Ak už bežala pred štartom editora, urob reconnect MCP servera `unreal` alebo ju reštartuj.
3. Nástroje: `list_toolsets`, `describe_toolset` (parameter `toolset_name`), `call_tool`.

Port 8000 je jeden na počítač: naraz nemôže bežať editor tohto projektu a editor HandballGame so zapnutým MCP. Port sa dá zmeniť parametrom editora `-ModelContextProtocolPort=N` (potom treba zmeniť aj `.mcp.json`).

## Test (2026-11-06)

- `initialize` -> OK (protocol 2025-11-25); server vyžaduje hlavičku `Mcp-Session-Id` pre ďalšie volania.
- `tools/list` -> `list_toolsets`, `describe_toolset`, `call_tool`.
- `list_toolsets` -> 20 toolsetov (EditorAppToolset, LogsToolset, AssetTools, BlueprintTools, SkeletalMeshTools, ProgrammaticToolset, ...).
- `call_tool` `EditorToolset.EditorAppToolset.IsPIERunning` -> `{"returnValue":false}`.
- Pluginy namountované v logu: MetaHuman, MetaHumanCoreTech, MetaHumanSDK, MetaHumanCalibrationDiagnostics, MetaHumanCalibrationProcessing, ModelContextProtocol, EditorToolset, PythonScriptPlugin.

## Odľahčenie CPU (2026-11-06)

Zmeny: z `.uproject` odstránené `AIAssistant`, `Landmass`, `InEditorDocumentation`, `MetaHumanCalibrationProcessing`, `MetaHumanCalibrationDiagnostics` (záloha pôvodného `.uproject` už neexistuje; `DefaultEngine.ini.bak` je len lokálne); v `DefaultEngine.ini` `r.ReflectionMethod=2`, `r.DynamicGlobalIlluminationMethod=0`, `r.Shadow.Virtual.Enable=0`, `r.Lumen.HardwareRayTracing=False`.
Výsledok merania: **efekt zmien nie je známy, lebo pred zmenami sa CPU nemeralo.** Všetky merania vznikli až po zmenách: editor ~50-55 % z 8 vlákien, bez shader workerov. `t.MaxFPS=30` hodnotu nezmenilo a bolo vrátené. V logu bolo okolo merania PIE; či sa s meraním časovo prekrývalo, nie je overené. `Landmass` sa naďalej načítava, dôvod nie je zistený.
