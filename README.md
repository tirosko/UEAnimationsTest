# UEAnimationsTest

Testovací projekt pre Unreal Engine 5.8 (5.8.4, Windows): prevod videa na animáciu tela cez MetaHuman Animator.

## Obsah repozitára

| Cesta | Čo obsahuje |
|---|---|
| `AnimationsTest.uproject` | projekt (EngineAssociation 5.8) |
| `Config/` | nastavenia projektu (1080p, Unreal MCP, odľahčené renderovanie) |
| `Docs/MetaHumanVideoToBodyAnimation.md` | potrebné pluginy, nastavenie editora a workflow video -> animácia tela |
| `Docs/UnrealMCPSetup.md` | nastavenie a test Unreal MCP, poznámky k odľahčeniu CPU |
| `.mcp.json` | server `unreal` na `http://127.0.0.1:8000/mcp` (lokálny editor) |

`Content/` (šablóna od Epicu) v repozitári **nie je**, je v `.gitignore`. Projekt sa po otvorení v editore doplní o vlastné assety.

## Stav

- Zapnuté pluginy: `MetaHuman`, `MetaHumanBodyTracker` (MetaHuman Animator Markerless Motion Capture, beta, nainštalovaný z Fab do enginu), `ModelingToolsEditorMode`, `PythonScriptPlugin`, `ModelContextProtocol`, `EditorToolset`.
- Plugin pre sledovanie tela z videa je nainštalovaný a načítaný (moduly `MetaHumanBodyTracker`, `BodyTracker`, `Segmentation`, `MetaHumanBodyOptimizer`, bez warningov v logu). Detaily v [Docs/MetaHumanVideoToBodyAnimation.md](Docs/MetaHumanVideoToBodyAnimation.md).
- Unreal MCP otestovaný: server beží na porte 8000, `list_toolsets` a `call_tool` fungujú.
- Neotestované: samotné spracovanie videa na animáciu (MetaHuman Performance), výkon na 8 GB VRAM a 4 fyzických jadrách CPU.

## Použitie

1. Otvor `AnimationsTest.uproject` v UE 5.8.
2. Plugin `MetaHumanBodyTracker` musí byť nainštalovaný v engine (Fab Library > Install to Engine, UE 5.8); v projekte je zapnutý v `.uproject`.
3. Pre prácu cez Claude Code spusti session v tomto priečinku a over, že editor beží (MCP na porte 8000).
