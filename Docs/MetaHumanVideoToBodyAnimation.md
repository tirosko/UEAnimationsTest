# MetaHuman: video -> animácia tela (UE 5.8)

Stav overený: 2026-11-06. Engine: `E:\Epic_Games\UE_5.8` (verzia 5.8.4, `Engine/Build/Build.version`). Projekt: `AnimationsTest.uproject` (EngineAssociation 5.8).
Zdroje: inštalácia enginu (`.uplugin`, zdrojáky pluginov), `Saved/Logs/AnimationsTest_2.log` a oficiálna dokumentácia Epic (odkazy dole).

## 1. Zhrnutie

Video -> body animation je v UE 5.8 **experimentálna** funkcia „MetaHuman Animator Markerless Motion Capture“ (len Windows, len editor). Používa sa cez asset **MetaHuman Performance** (MonoFootage + zapnuté *Body Tracking*).

**Blokujúci nedostatok:** plugin *MetaHuman Animator Markerless Motion Capture* sa **nenachádza v inštalácii enginu**. Epic ho distribuuje cez Fab, treba ho nainštalovať. V engine je len rozhranie `MetaHumanBodyTrackerInterface` (modul v `MetaHumanCoreTech`) a kód hľadá plugin s názvom `MetaHumanBodyTracker` (`HyprsenseRealtimeNode.cpp`); implementácia v engine nie je. Prehľadanie `E:\` a `C:\ProgramData\Epic` žiadny takýto plugin nenašlo.

## 2. Pluginy – stav

| Plugin | Rola | V engine | V .uproject |
|---|---|---|---|
| MetaHuman (MetaHuman Animator, `MetaHuman.uplugin`) | Performance asset, Identity, Capture Source | áno | **zapnutý** |
| MetaHumanCoreTech | jadro, `MetaHumanBodyTrackerInterface` | áno | závislosť (log: modul sa načíta) |
| MetaHumanSDK | základ MetaHuman nástrojov | áno | závislosť (log: mountnutý) |
| CaptureData, CaptureManagerEditor, PerformanceCaptureCore, LiveLinkHub, NNERuntimeORT, IKRig... | závislosti MetaHuman pluginu (všetky `Enabled: true` v `MetaHuman.uplugin`) | áno | zapnú sa ako závislosti |
| MetaHumanCalibrationProcessing / MetaHumanCalibrationDiagnostics | kalibrácia stereo kamier | áno | zapnuté; **pre mono video->telo nie sú potrebné** |
| MetaHumanCharacter (MetaHuman Creator, Beta) | tvorba/assemblovanie postavy (voliteľná vizualizácia) | áno | nezapnutý (nie je povinný, ale užitočný) |
| MetaHumanLiveLink | live link zdroje (real-time) | áno | nezapnutý, pre offline workflow netreba |
| **MetaHuman Animator Markerless Motion Capture** (Fab) | **telové sledovanie z videa** | **NIE** | **chýba – nainštalovať z Fab** |

Poznámka: Epic stránka v kroku 2 spomína aj plugin „MetaHuman Animator Body Animation“ (Small Face Detector/Solver). Dokumentácia nehovorí, či je to ten istý plugin; po inštalácii z Fab to overiť v okne Plugins.
Pluginy v `.uproject` som **nemenil**: Markerless plugin sa tam bez inštalácie z Fab pridať nedá a ostatné povinné sú riešené závislosťami.

## 3. Zistenia o tomto PC (z logu a nástrojov)

- GPU: NVIDIA GeForce RTX 5060, ~7.9 GB VRAM, ovládač 621.14. Epic odporúča „at least RTX 3070 ... with 8GB VRAM“, takže VRAM je na hranici; výkon vs. RTX 3070 som neoveroval.
- D3D12 RHI beží, DirectX Agility SDK nájdené; NNERuntimeORT hlási DirectML: yes, RHI D3D12: yes.
- Python 3.11.8 (PythonScriptPlugin) beží.
- Epic prihlásenie: editor bol spustený cez launcher (`-EpicPortal`, užívateľ prihlásený). Hesla neriešiť.
- Editor bol neskôr spustený (2026-11-06) s Unreal MCP, pozri sekciu 3a a [UnrealMCPSetup.md](UnrealMCPSetup.md).
- RAM/CPU (zmerané): 64 GB RAM (spĺňa 32 GB); CPU Intel Xeon W-2125, **4 fyzické jadrá / 8 vlákien** - výrazne pod odporúčaním 16 fyzických jadier, spracovanie bude pomalšie.
- `Content/` obsahuje len šablónu (Characters/Mannequins, FirstPerson, LevelPrototyping, DemoTemplate); žiadne MetaHuman assety ani Capture Data.

## 3a. Overenie v bežiacom editore (2026-11-06)

Zdroj: `Saved/Logs/AnimationsTest.log` z editora s Unreal MCP (UE 5.8.4) a `Config/DefaultEngine.ini`.

- Namountované pluginy: `MetaHuman`, `MetaHumanCoreTech`, `MetaHumanSDK`, `MetaHumanCalibrationProcessing`, `MetaHumanCalibrationDiagnostics`, `CaptureData`, `CaptureManagerCore`, `CaptureManagerEditor`, `PerformanceCaptureCore`, `LiveLink`, `LiveLinkDevice`, `LiveLinkHub`, `NNERuntimeORT`, `NNERuntimeBasicCpu`, `IKRig`, `ControlRig`, `FullBodyIK`, `PythonScriptPlugin`. Všetky závislosti MetaHuman pluginu sa teda zapli automaticky.
- **Nenamountované:** žiadny plugin Markerless Motion Capture / `MetaHumanBodyTracker`; priečinok `Plugins` v projekte neexistuje. Blokujúci nedostatok z sekcie 1 **platí**.
- `MetaHumanCharacter` a `MetaHumanLiveLink` nie sú namountované (nie sú povinné).
- Default RHI: `DefaultGraphicsRHI=DefaultGraphicsRHI_DX12` v `DefaultEngine.ini`; log: `Using Default RHI: D3D12`, `Highest Feature Level of D3D12: SM6`. Krok 4 v sekcii 4 je splnený.
- V logu pri MetaHuman/Markerless nie sú žiadne warningy ani errory.
- Neoverené: vzhľad okna Plugins a Show Engine Content (nevidel som UI, len log).
## 4. Nastavenie editora krok za krokom

1. V Epic Games Launcheri/Fab (prihlásený účet) pridaj do knižnice a nainštaluj do UE 5.8 **MetaHuman Animator Markerless Motion Capture**: <https://www.fab.com/listings/4095b8e0-3eff-44f1-acb4-cb40b99228b9>
2. Otvor projekt. V **Edit > Plugins** over, že sú zapnuté: *MetaHuman Animator*, *Capture Manager*, *Live Link Hub*, *MetaHuman Animator Markerless Motion Capture*. Zapni, čo chýba, a reštartuj editor.
3. Voliteľne zapni *MetaHuman Creator* (MetaHumanCharacter, Beta) pre tvorbu postavy.
4. Over D3D12: **Project Settings > Platforms > Windows > Default RHI = DirectX 12**.
5. V Content Browseri zapni **Show Engine Content**, aby si videl modely Small Face Detector/Solver.
6. Voliteľne FFmpeg: v Capture Manageri *Settings > Use Third Party Encoder* a cesta k `ffmpeg.exe` (znižuje riziko neplatného FPS).

## 5. Workflow video -> animácia tela (podľa dokumentácie Epic)

1. **Ingest:** *Tools > Live Link Hub* > *Capture Manager* > Data Devices > Add > **Mono Video Ingest**; nastav cestu k priečinku s videami (Video Discovery Expression nechaj `<Auto>`), pridaj súbor do Queue, **Start**. Výstup: `Content/CaptureManager/Imports`.
2. **MetaHuman Performance:** Content Browser > right-click > *MetaHuman > MetaHuman Performance*; v Details nastav *Footage Capture Data*.
3. **Iba telo:** vypni *Facial Tracking*, zapni **Body Tracking**, *Head Movement Mode* = Disable.
   **Tvár + telo:** zapni obe a v *Facial Tracking > Advanced > Models* nastav Face Detector = **Small Face Detector** a Face Solver = **Small Face Solver**.
4. Nastavenia tela v assete (z `MetaHumanPerformance.h`): Auto Body Height / Body Height (mimo 145–190 cm môže klesnúť kvalita), Body Detection Confidence (0.7), Body Tracking Confidence (0.5), Enable Foot-Locking.
5. Orež timeline, **Process**, skontroluj výsledok, prípadne uprav a spusti znova.
6. **Export Animation:** *Existing Skeleton* (MetaHuman, automatický retarget) alebo *Performer Skeleton* (pre iné postavy, retarget cez IK Retargeter).
7. **Aplikácia:** Level Sequence > Add Actor Track (MetaHuman), zmaž ControlRig track, na Body track pridaj exportovanú Animation Sequence.

## 6. Požiadavky na video

Oficiálne stránky, ktoré som čítal, **nešpecifikujú** rozlíšenie, FPS, kodek, rámovanie, počet osôb ani statickú kameru. Epic uvádza len: funguje s webkamerou a podporovanými smartfónmi, alebo Live Link Face (iOS/Android); východiskový transcoder je Windows Media Framework, FFmpeg podporuje viac formátov.
Treba dohľadať: <https://dev.epicgames.com/documentation/metahuman/metahuman-animator-performance-capture-guidelines-in-unreal-engine> a <https://dev.epicgames.com/documentation/metahuman/metahuman-animator-capture-device-requirements>.

## 7. Obmedzenia

- Experimentálne, môže sa zmeniť alebo zaniknúť; len editor (nie runtime).
- Telové funkcie len Windows (Linux/macOS len tvárový MetaHuman Animator).
- Raw export tela je skrytý za CVar `mh.BodyTracker.EnableRawExport` (z kódu).
- Kvalita závisí od VRAM (8 GB je odporúčané minimum).

## 8. Čo ostáva overiť

- Inštalácia Markerless Motion Capture z Fab a presný názov pluginu v okne Plugins (rozdiel s „Body Animation“).
- Vzhľad okna Plugins v UI (z logu je stav známy, pozri 3a).
- Výkon a stabilita na 8 GB VRAM a 4 fyzických jadrách CPU.
- Presné požiadavky na video (rozlíšenie, FPS, rámovanie).
- Skúšobný beh na krátkom klipe.

## Zdroje

- <https://dev.epicgames.com/documentation/metahuman/metahuman-animation-from-mono-video-capture-in-unreal-engine>
- <https://dev.epicgames.com/documentation/metahuman/metahuman-5-8-release-notes-in-unreal-engine>
- <https://dev.epicgames.com/documentation/metahuman/metahuman-hardware-requirements-in-unreal-engine>
- Kroky 1–3: `metahuman-animator-01-import-your-footage-in-unreal-engine`, `-02-process-and-export-your-animation-in-unreal-engine`, `-03-apply-your-animation-to-a-metahuman-in-unreal-engine` (rovnaká doména).
