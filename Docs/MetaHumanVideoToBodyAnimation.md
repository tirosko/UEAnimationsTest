# MetaHuman: video -> animácia tela (UE 5.8)

Stav overený: 2026-11-06. Engine: `E:\Epic_Games\UE_5.8` (verzia 5.8.4, `Engine/Build/Build.version`). Projekt: `AnimationsTest.uproject` (EngineAssociation 5.8).
Zdroje: inštalácia enginu (`.uplugin`, zdrojáky pluginov), `Saved/Logs/AnimationsTest_2.log` a oficiálna dokumentácia Epic (odkazy dole).

## 1. Zhrnutie

Video -> body animation je v UE 5.8 **experimentálna** funkcia „MetaHuman Animator Markerless Motion Capture“ (len Windows, len editor). Používa sa cez asset **MetaHuman Performance** (MonoFootage + zapnuté *Body Tracking*).

**Plugin je nainštalovaný a načítaný** (2026-11-06): *MetaHuman Animator Markerless Motion Capture*, názov pluginu `MetaHumanBodyTracker` v1.0.0 (Beta, Experimental), `E:\Epic_Games\UE_5.8\Engine\Plugins\Marketplace\MetaHumanBodyTracker_5.8` (~5,5 GB, obsahuje modely Detectron2, IK_SMPL a IK Retargeter pre MetaHuman). Či je to ten istý plugin, ktorý Epic v kroku 2 nazýva „Body Animation“, nie je potvrdené; v inštalácii sa medzi súbormi `.uplugin` nenašiel iný s „Body“ v názve, čo to len naznačuje. Bol nainštalovaný do **enginu** (Fab/Launcher), v projekte je zapnutý v `AnimationsTest.uproject`.

## 2. Pluginy – stav

| Plugin | Rola | V engine | V .uproject |
|---|---|---|---|
| MetaHuman (MetaHuman Animator, `MetaHuman.uplugin`) | Performance asset, Identity, Capture Source | áno | **zapnutý** |
| MetaHumanCoreTech | jadro, `MetaHumanBodyTrackerInterface` | áno | závislosť (log: modul sa načíta) |
| MetaHumanSDK | základ MetaHuman nástrojov | áno | závislosť (log: mountnutý) |
| CaptureData, CaptureManagerEditor, PerformanceCaptureCore, LiveLinkHub, NNERuntimeORT, IKRig... | závislosti MetaHuman pluginu (všetky `Enabled: true` v `MetaHuman.uplugin`) | áno | zapnú sa ako závislosti |
| MetaHumanCalibrationProcessing / MetaHumanCalibrationDiagnostics | kalibrácia stereo kamier | áno | **odstránené z .uproject** (pre mono video->telo netreba) |
| **MetaHumanCharacter** (MetaHuman Creator, Beta) | obsahuje `SKM_Body_DNA`, od ktorého závisí `/MetaHumanBodyTracker/SKM_Body` | áno | **zapnutý, potrebný** (bez neho log hlási `LoadErrors`) |
| MetaHumanLiveLink | live link zdroje (real-time) | áno | nezapnutý, pre offline workflow netreba |
| **MetaHuman Animator Markerless Motion Capture** (`MetaHumanBodyTracker`, Fab) | **telové sledovanie z videa** | **áno** (`Plugins/Marketplace/MetaHumanBodyTracker_5.8`) | **zapnutý** |

Poznámka: vzťah „Body Animation“ z Epic stránky a Markerless Motion Capture nie je potvrdený (pozri sekciu 1).
V `.uproject` je teraz zapnutý `MetaHumanBodyTracker`; `MetaHuman`, `MetaHumanCoreTech` a `IKRig` sú jeho závislosti.

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

- Namountované pluginy (pôvodný stav pred inštaláciou Body Trackera): `MetaHuman`, `MetaHumanCoreTech`, `MetaHumanSDK`, `MetaHumanCalibrationProcessing`, `MetaHumanCalibrationDiagnostics`, `CaptureData`, `CaptureManagerCore`, `CaptureManagerEditor`, `PerformanceCaptureCore`, `LiveLink`, `LiveLinkDevice`, `LiveLinkHub`, `NNERuntimeORT`, `NNERuntimeBasicCpu`, `IKRig`, `ControlRig`, `FullBodyIK`, `PythonScriptPlugin`. Všetky závislosti MetaHuman pluginu sa teda zapli automaticky.
- **MetaHumanBodyTracker** sa po inštalácii z Fab a zapnutí v `.uproject` namountoval; v logu sa načítali moduly `MetaHumanBodyTracker`, `BodyTracker`, `Segmentation`, `MetaHumanBodyOptimizer` a `MetaHumanBodyTrackerInterface`, bez warningov a errorov.
- `MetaHumanLiveLink` nie je namountovaný (pre offline workflow netreba). `MetaHumanCharacter` bol pôvodne vypnutý, čo spôsobilo `LoadErrors`; je zapnutý, pozri sekciu 5a.
- Default RHI: `DefaultGraphicsRHI=DefaultGraphicsRHI_DX12` v `DefaultEngine.ini`; log: `Using Default RHI: D3D12`, `Highest Feature Level of D3D12: SM6`. Krok 4 v sekcii 4 je splnený.
- V logu pri MetaHuman/Markerless nie sú žiadne warningy ani errory.
- Neoverené: vzhľad okna Plugins a Show Engine Content (nevidel som UI, len log).
## 4. Nastavenie editora krok za krokom

1. **Hotovo:** plugin *MetaHuman Animator Markerless Motion Capture* je nainštalovaný do UE 5.8 (Fab Library > Install to Engine): <https://www.fab.com/listings/4095b8e0-3eff-44f1-acb4-cb40b99228b9>
2. V `AnimationsTest.uproject` je zapnutý `MetaHumanBodyTracker`. V **Edit > Plugins** (kategória MetaHuman) over aj *MetaHuman Animator*, *Capture Manager*, *Live Link Hub*; po každej zmene reštartuj editor.
3. Zapni *MetaHuman Creator* (`MetaHumanCharacter`, Beta). Je potrebný: asset `/MetaHumanBodyTracker/SKM_Body` závisí od jeho obsahu. V tomto projekte je zapnutý.
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

## 5a. Overený test (2026-11-06)

Klip: mono video 1920x1080, 30 fps, 38 s (1148 snímok). Spracovaných bolo prvých 150 snímok, len telo (Facial Tracking vypnutý). Video a výsledné assety sú lokálne a nie sú v repozitári.

| Krok | Výsledok |
|---|---|
| Ingest (Mono Video Ingest cez Live Link Hub) | OK, 46 s; assety v `/Game/CaptureManager/Imports/`, snímky v `%LOCALAPPDATA%\CaptureManager\Media\AnimationsTest` |
| MetaHuman Performance (Process) | OK, `StartPipelineErrorType.NONE`, 641 s na 150 snímok (~4,3 s/snímka), `BODY` dáta áno |
| Export Animation (Performer Skeleton) | OK, `AnimSequence`, 149 snímok, 4,97 s, 342 kostí |
| Kontrola pohybu | rotácie sa menia na všetkých 11 kontrolovaných kostiach (najviac `upperarm_r` ~20°), panva sa posúva o ~8 cm |

Odhad pre celý klip je lineárne ~80 minút, nebol skúšaný. Počas jednej vzorky v stage 4 bolo GPU na 100 % (7 GB z 8 GB VRAM) a CPU takmer nečinné; ostatné stage neboli merané, takže nie je potvrdené, že CPU nie je úzke hrdlo inde.

**Výsledok vizuálnej kontroly: neúspech.** Používateľ animáciu pozrel v editore a nevyzerala dobre. Čísla ukázali len to, že sa kosti hýbu, nie že pohyb zodpovedá videu. Príčina nie je zistená; klip bol vybraný bez kontroly obsahu, takže nemusel spĺňať podmienky (jeden človek, celá postava v zábere). Klip, snímky a animácia boli z projektu odstránené (presunuté do Koša).

Poznámky k skriptovaniu:
- Epic skripty: `CaptureManagerApp/Content/Python/examples/mono_video_ingest.py` (spúšťa sa v `LiveLinkHub.exe`, editor musí bežať) a `MetaHumanAnimator/Content/Python/process_monocular_performance.py`.
- `-ExecutePythonScript=` po dobehnutí skriptu aplikáciu zavrie. Priečinok s videom musí obsahovať len klipy na import.
- Blokujúce spracovanie hlási v správcovi úloh „Not Responding“, čo je očakávané.
- `get_number_of_processed_frames()` počíta len dáta tváre, pri spracovaní iba tela vráti 0. Na kontrolu slúži `contains_animation_data_type(BODY)`.
- Warning `Unspecified device class, assuming iPhone 13` sa v logu objavil; vplyv na výsledok nie je známy.

### Druhý test (2026-11-06)

Klip: 1024x576, 30 fps, 3 s, 93 snímok, celý klip, len telo. Pipeline OK za 257 s (~2,8 s/snímka; stage 1-5: 92, 50, 24, 69, 21 s). Export `AnimSequence`: 92 snímok, 3,07 s, 342 kostí. Pohyb je výrazne väčší než pri prvom teste (`lowerarm_r` yaw ~116°, `upperarm_r` ~60°). Hodnota `thigh_r` yaw ~356° je pravdepodobne len prechod uhla cez ±180°. Vizuálne posúdenie zatiaľ chýba.

Poznámka k ingestu: prvé spustenie Live Link Hub po štarte editora skončilo `TimeoutError: Timed out waiting for endpoint`, lebo editor sa pripojil až po uvoľnení hlavného vlákna Hubu. Druhé spustenie prešlo. Stalo sa to pri oboch testoch.

### Prenos na postavu Manny (2026-11-06)

Export na *Performer Skeleton* má dva problémy: sieť je holé telo bez hlavy a v oboch testoch vyšlo telo ležiace pozdĺž osi X (hlava X≈-50, nohy X≈+78, výška len 6-25 cm). Príčina ležania nie je zistená.

Postup, ktorý zabral (Python cez `-ExecutePythonScript`):
1. IK Rig pre `SKM_Manny_Simple`: `IKRigController.apply_auto_generated_retarget_definition()` -> `/Game/VideoTest/IK_Manny_Video`.
2. Kópia Epic retargetera `/MetaHumanBodyTracker/RTG_MH_IKRig` (zdroj `IK_Metahuman`), cieľ prepnutý na Manny, `assign_ik_rig_to_all_ops`, `auto_map_chains(FUZZY)`, `auto_align_all_bones(TARGET)` -> `RTG_Video_To_Manny`.
3. `IKRetargetBatchOperation.duplicate_and_retarget` zo `SK_A_hrac_Body` na Manny -> `A_hrac_Body_Manny`. Ležanie sa prenesie tiež.
4. Korekcia: konštantná rotácia a posun kosti `root` cez `set_bone_track_keys`, vypočítané z priemernej osi nohy-hlava -> `A_hrac_Manny_Upright` (panva ~99 cm, hlava ~163 cm, nohy 10-20 cm).

Priamy export na Manny cez *Existing Skeleton* zo skriptu nejde: `SourceSkeletalMesh` nastavuje len editorové UI a pre tento režim sa sám nevytvorí. V UI dialógu Export Animation nebol skúšaný a môže byť čistejší.

Obmedzenia: korekcia je jedna konštantná hodnota na celý klip, nie oprava príčiny. Výška hlavy sa počas klipu mení len o ~3 cm a nohy skoro stoja, takže animácia neukazuje výskok.

## 6. Požiadavky na video

Oficiálne stránky, ktoré som čítal, **nešpecifikujú** rozlíšenie, FPS, kodek, rámovanie, počet osôb ani statickú kameru. Epic uvádza len: funguje s webkamerou a podporovanými smartfónmi, alebo Live Link Face (iOS/Android); východiskový transcoder je Windows Media Framework, FFmpeg podporuje viac formátov.
Treba dohľadať: <https://dev.epicgames.com/documentation/metahuman/metahuman-animator-performance-capture-guidelines-in-unreal-engine> a <https://dev.epicgames.com/documentation/metahuman/metahuman-animator-capture-device-requirements>.

## 7. Obmedzenia

- Experimentálne, môže sa zmeniť alebo zaniknúť; len editor (nie runtime).
- Telové funkcie len Windows (Linux/macOS len tvárový MetaHuman Animator).
- Raw export tela je skrytý za CVar `mh.BodyTracker.EnableRawExport` (z kódu).
- Kvalita závisí od VRAM (8 GB je odporúčané minimum).

## 8. Čo ostáva overiť

- Zobrazenie pluginu v okne Plugins v UI (z logu je načítaný).
- Vzhľad okna Plugins v UI (z logu je stav známy, pozri 3a).
- Stabilita na dlhších klipoch pri 8 GB VRAM (5 s klip využil ~7 GB).
- Presné požiadavky na video (rozlíšenie, FPS, rámovanie).
- Dobrá vizuálna kvalita zatiaľ nebola dosiahnutá; ďalší test s vhodným klipom (jeden hráč, celá postava, statická kamera). Export na MetaHuman cez *Existing Skeleton*.

## Zdroje

- <https://dev.epicgames.com/documentation/metahuman/metahuman-animation-from-mono-video-capture-in-unreal-engine>
- <https://dev.epicgames.com/documentation/metahuman/metahuman-5-8-release-notes-in-unreal-engine>
- <https://dev.epicgames.com/documentation/metahuman/metahuman-hardware-requirements-in-unreal-engine>
- Kroky 1–3: `metahuman-animator-01-import-your-footage-in-unreal-engine`, `-02-process-and-export-your-animation-in-unreal-engine`, `-03-apply-your-animation-to-a-metahuman-in-unreal-engine` (rovnaká doména).
