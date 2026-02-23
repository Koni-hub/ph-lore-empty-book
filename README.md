# THE EMPTY BOOK

**Type:** Story-Driven First-Person Horror
**Engine:** Unreal Engine 5.7.3
**Platform:** PC (Windows)
**Development Status:** In Development — Chapter 1 Active
**Language:** Blueprint (C++ reserved for performance-critical systems)

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Game Concept and Narrative](#2-game-concept-and-narrative)
3. [Technical Stack](#3-technical-stack)
4. [Core Systems](#4-core-systems)s
5. [Project Folder Structure](#5-project-folder-structure)
6. [Cinematic Pipeline](#6-cinematic-pipeline)
7. [NPC Dialogue System](#7-npc-dialogue-system)
8. [Level Flow and Gameplay Structure](#8-level-flow-and-gameplay-structure)
9. [AI System — The Alan](#9-ai-system--the-alan)
10. [Chase System](#10-chase-system)
11. [Chapter Roadmap](#11-chapter-roadmap)
12. [Adding a New Chapter — Developer Guide](#12-adding-a-new-chapter--developer-guide)
13. [Recommended Plugins and Extensions](#13-recommended-plugins-and-extensions)
14. [Naming Conventions](#14-naming-conventions)
15. [Content Warning](#15-content-warning)

---

## 1. Project Overview

The Empty Book is a single-player, narrative-driven horror game built in Unreal Engine 5.7.3s. The game follows Marco, an investigative writer based in Manila, who receives an anonymous parcel containing a blank field journal. Each chapter sends Marco to a different Philippine province where he encounters a different creature from Philippine folklore.

Marco is the permanent protagonist across all chapters. Each chapter is a self-contained story — a new location, a new creature, a new set of villagers — but the journal, the field recorder, and the mystery of the parcel carry through from chapter to chapter.

The game contains no combat system. All player progression is driven by environmental investigation, object inspection, and story triggers. Tension is built through atmosphere, forced perspective, and encounter design rather than weapons or health systems.

All core systems — dialogue, investigation, journal, objectives, and cinematics — are designed to be reusable and data-asset-driven so that new chapters can be added without rebuilding infrastructure.

---

## 2. Game Concept and Narrative

### The Protagonist

Marco is an investigative writer. He does not fight. He documents. His tools are a field recorder, a camera, a journal that is no longer entirely his, and ritual notes left by his Lola that he is only beginning to understand.

### The Journal

The field journal is the central object of the game. It arrives blank. Pages fill themselves as Marco discovers clues, encounters creatures, and survives the night. The journal is persistent across all chapters and acts as both a progress tracker and a narrative device. New sketches, handwritten lines, and warnings appear without Marco writing them.

### Chapter Structure

Each chapter is a standalone encounter. Marco travels to a new province, investigates a disturbance connected to a specific Philippine mythological creature, and survives. The creature changes. The province changes. The villagers change. Marco and the journal do not.

### Prologue — Manila

Marco sits at his typewriter attempting to document Philippine folklore. A knock at the door. No one is there. A parcel is left in the hallway — wet, muddy, smelling faintly of soil and iron. Inside is the blank journal. Within moments the journal writes its first line on its own:

> IF YOU SEEK TRUTH, GO WHERE THE SKY TOUCHES THE BONES.

A black feather falls from the parcel. The typewriter begins typing the same line without anyone touching it. The radio plays a woman humming — from behind Marco, not from the speaker. Marco spins. Nothing. When he turns back, the journal now contains a rough sketch of something tall, winged, and wrong.

### Chapter 1 — Abra — The Alan

The Alan are winged hunters from Philippine folklore described as bone collectors who do not forgive the disturbance of shrines. Chapter 1 takes place in a mountain village in Abra where a cliff shrine has been disturbed. Villagers report dead livestock with no wounds. Marco investigates the shrine, photographs the altar, and encounters the Alan.

The Alan does not attack immediately. It observes. It sniffs. Then it follows.

### Chapter 2 — Batangas — The Aswang

Planned. Details to be documented when Chapter 2 enters active development. Marco travels to Batangas. A different creature. A different kind of wrong.

---

## 3. Technical Stack

| System             | Implementation                       |
| ------------------ | ------------------------------------ |
| Engine             | Unreal Engine 5.5.4                  |
| Rendering          | Lumen (Global Illumination) + Nanite |
| Input              | Enhanced Input System                |
| AI                 | Behavior Trees + Blackboard          |
| Cinematics         | Level Sequences + Sequencer          |
| Audio              | MetaSounds                           |
| UI Framework       | UMG + Common UI Plugin               |
| Save System        | Easy Multi Save (Fab.com, free)      |
| Physics            | Chaos Physics                        |
| Environment Assets | Quixel Megascans (bundled with UE5)  |

All plugins listed above are free except where noted. The dialogue system is custom-built in Blueprint — no paid dialogue plugin is used.

---

## 4. Core Systems

All core systems are data-asset-driven and chapter-agnostic. Adding a new chapter requires creating new data assets and levels — the underlying systems do not need to be modified.

### 4.1 Investigation System

All inspectable objects inherit from `BP_Interactable_Base` and implement the `BPI_Interactable` interface. When Marco interacts with an object, the interface calls `Interact()` which returns a `DA_InvestigationNode` data asset containing the display text, background lore, and a completion flag.

`BP_InvestigationManager` tracks which nodes have been collected and fires objective updates when a scene's required nodes are all complete.

### 4.2 Dialogue System

Dialogue is stored as `DA_DialogueLine` data assets. Each asset contains the speaker name, English dialogue text, Filipino subtitle text, voice audio reference, portrait texture, a Sequencer camera hint tag, and boolean flags for player-triggered lines and conversation end.

Conversations are chains of `DA_DialogueLine` assets grouped into `DA_DialogueConversation` assets. `BP_DialogueManager` is a singleton that handles conversation queuing, line advancement, and Sequencer event handoff.

NPCs have a `BP_DialogueComponent` attached which holds their main conversation and any optional player-triggered lines that unlock after the main scene.

### 4.3 Journal System

The journal persists across all levels and all chapters via `BP_GameInstance`. Each entry is a `DA_JournalEntry` data asset with text content, a sketch texture, and a `bSelfWrites` boolean that triggers the animated ink-bleed material reveal.

Journal entries are added via Sequencer Event Tracks. The journal widget `WBP_Journal` polls `BP_GameInstance` for new entries and animates them into the page layout.

### 4.4 Objective System

`BP_ObjectiveManager` maintains the current active objective and history of completed objectives. Objectives are defined as `DA_ObjectiveData` assets and listed in `DT_ObjectiveList`. The `WBP_ObjectiveTracker` HUD widget subscribes to `BP_ObjectiveManager` and updates when the active objective changes.

### 4.5 Field Recorder

`BP_FieldRecorder` handles the in-game recorder mechanic. It captures a reference to the ambient audio timeline during encounters. On playback at narrative beats, it replays the captured sequence — footsteps, creature sounds, Marco's own voice — as a diegetic story beat.

### 4.6 Jumpscare System

`BP_JumpscareManager` handles forced perspective events via a `ForcePOV(TargetActor, BlendTime)` function that overrides the player camera. The jar reflection scene in Chapter 1 uses a `SceneCaptureComponent2D` on the jar material to render what is behind Marco in real time before the forced cut.

### 4.7 Chase System

`BP_ChaseDirector` orchestrates creature encounter sequences. It activates via `BeginChase()` called from a Sequencer Event Track. It manages creature speed curves via Timeline, activates `BP_ChaseZone` trigger volumes that phase the encounter, and handles special events like the wind knockdown. There is no combat. The creature cannot be stopped.

The chase system is designed to be reusable across chapters. Each chapter's creature gets its own `BP_ChaseDirector` instance in its level, configured independently.

---

## 5. Project Folder Structure

The folder structure is organized to scale cleanly with new chapters. Every chapter follows the same internal layout under its own subfolder. Shared systems live outside all chapter folders and are never duplicated.

```
TheEmptyBook/
|
Content/
|
|-- _Core/
|   |-- GameModes/
|   |   `-- BP_GameMode_EmptyBook
|   |-- GameInstance/
|   |   `-- BP_GameInstance          # Persistent data: journal, flags, chapter state
|   `-- SaveGame/
|       `-- BP_SaveGame
|
|-- Characters/
|   |-- Marco/                       # Marco is shared across all chapters
|   |   |-- Blueprints/
|   |   |   `-- BP_Marco_Character
|   |   |-- Animations/
|   |   |   `-- ABP_Marco
|   |   |-- Mesh/
|   |   |   `-- SK_Marco
|   |   `-- Input/
|   |       |-- IA_Move
|   |       |-- IA_Look
|   |       |-- IA_Interact
|   |       |-- IA_Run
|   |       |-- IA_UseCamera
|   |       `-- IMC_Marco
|   |
|   `-- Creatures/                   # One subfolder per creature, one per chapter
|       |-- Alan/                    # Chapter 1
|       |   |-- Blueprints/
|       |   |   |-- BP_Alan_Base
|       |   |   `-- BP_Alan_BehaviorController
|       |   |-- AI/
|       |   |   |-- BT_Alan
|       |   |   |-- BB_Alan
|       |   |   |-- BTTask_Observe
|       |   |   |-- BTTask_BeginFollow
|       |   |   `-- BTService_UpdateMarcoLocation
|       |   |-- Animations/
|       |   |   `-- ABP_Alan
|       |   `-- Mesh/
|       |       `-- SK_Alan
|       |
|       `-- Aswang/                  # Chapter 2 — placeholder, do not populate yet
|           |-- Blueprints/
|           |-- AI/
|           |-- Animations/
|           `-- Mesh/
|
|-- Levels/
|   |-- Prologue/
|   |   `-- L_MarcoApartment
|   |-- Transitions/
|   |   |-- L_BusTransition_Prologue
|   |   |-- L_BusTransition_Ch1     # Each transition is chapter-specific
|   |   `-- L_BusTransition_Ch2     # Placeholder
|   |-- Chapter1/
|   |   |-- L_MountainVillage
|   |   |-- L_CliffShrine
|   |   `-- L_LodgingRoom
|   |-- Chapter2/                   # Placeholder — do not populate yet
|   `-- Menus/
|       `-- L_MainMenu
|
|-- Cinematics/
|   |-- Prologue/
|   |   |-- SEQ_P01_OpeningShot
|   |   |-- SEQ_P02_ObjectReveal
|   |   |-- SEQ_P03_TypewriterJam
|   |   |-- SEQ_P04_KnockAtDoor
|   |   |-- SEQ_P05_ParcelReveal
|   |   |-- SEQ_P06_NotebookOpen
|   |   |-- SEQ_P07_FeatherDrop
|   |   |-- SEQ_P08_TypewriterPossessed
|   |   |-- SEQ_P09_RadioWoman
|   |   |-- SEQ_P10_MarcoSpins
|   |   |-- SEQ_P11_SketchAppears
|   |   `-- SEQ_P12_MapDecision
|   |-- Transitions/
|   |   |-- SEQ_T01_BusRide
|   |   `-- SEQ_T02_BusArrival
|   |-- Chapter1/
|   |   |-- SEQ_C1_01_BusJournalWarm
|   |   |-- SEQ_C1_02_VillageArrival
|   |   |-- SEQ_C1_03_FarmerDialogue
|   |   |-- SEQ_C1_04_CliffApproach
|   |   |-- SEQ_C1_05_ShrineWrong
|   |   |-- SEQ_C1_06_CameraFlash
|   |   |-- SEQ_C1_07_AlanReveal
|   |   |-- SEQ_C1_08_MorningWakeup
|   |   |-- SEQ_C1_09_RecorderPlayback
|   |   |-- SEQ_C1_10_JournalNewDrawing
|   |   |-- SEQ_C1_11_NextDestination
|   |   `-- SEQ_C1_12_ShadowWindow
|   |-- Chapter2/                   # Placeholder — populate when Chapter 2 starts
|   `-- Shared/
|       |-- SEQ_SHARED_TextReveal
|       |-- SEQ_SHARED_LampFlicker
|       `-- SEQ_SHARED_FadeToBlack
|
|-- Dialogue/
|   |-- Prologue/
|   |   `-- Marco_VO/
|   |       `-- DIA_Marco_PrologueVO
|   |-- Chapter1/
|   |   |-- VillageNPCs/
|   |   |   |-- DIA_OldFarmer_Main
|   |   |   |-- DIA_OldFarmer_Optional
|   |   |   |-- DIA_YoungFarmer_Main
|   |   |   |-- DIA_YoungFarmer_Optional
|   |   |   |-- DIA_MiddleAgedFarmer_Main
|   |   |   |-- DIA_MiddleAgedFarmer_Optional
|   |   |   `-- DIA_VillagerAmbient
|   |   `-- Marco_VO/
|   |       |-- DIA_Marco_BusVO
|   |       |-- DIA_Marco_ShrineVO
|   |       `-- DIA_Marco_MorningVO
|   |-- Chapter2/                   # Placeholder — populate when Chapter 2 starts
|   `-- Shared/
|       `-- DIA_Marco_InspectLines   # Generic inspect reactions, reused every chapter
|
|-- UI/
|   |-- HUD/
|   |   |-- WBP_HUD_Main
|   |   |-- WBP_Crosshair
|   |   `-- WBP_ObjectiveTracker
|   |-- Journal/
|   |   |-- WBP_Journal
|   |   `-- WBP_JournalPage
|   |-- Dialogue/
|   |   |-- WBP_DialogueBox
|   |   `-- WBP_DialogueChoice
|   |-- Investigation/
|   |   |-- WBP_InspectObject
|   |   `-- WBP_InvestigationLog
|   `-- Menus/
|       |-- WBP_MainMenu
|       |-- WBP_PauseMenu
|       `-- WBP_LoadingScreen
|
|-- Interactables/
|   |-- Base/
|   |   |-- BP_Interactable_Base     # Parent class — never modify directly
|   |   `-- BPI_Interactable         # Interface — implement on every inspectable object
|   |-- Prologue/
|   |   |-- BP_SantoNino
|   |   |-- BP_Newspaper
|   |   |-- BP_Photograph
|   |   |-- BP_Typewriter
|   |   `-- BP_Notebook
|   |-- Chapter1/
|   |   |-- BP_JarWithBones
|   |   |-- BP_CliffsideShrine
|   |   `-- BP_Camera_Item
|   `-- Chapter2/                   # Placeholder — populate when Chapter 2 starts
|
|-- Systems/
|   |-- Investigation/
|   |   |-- BP_InvestigationManager
|   |   `-- DA_InvestigationNode
|   |-- Journal/
|   |   |-- BP_JournalSystem
|   |   `-- DA_JournalEntry
|   |-- Dialogue/
|   |   |-- BP_DialogueManager
|   |   |-- BP_DialogueComponent
|   |   `-- DA_DialogueLine
|   |-- Chase/
|   |   |-- BP_ChaseDirector         # Base class — extend per chapter creature
|   |   `-- BP_ChaseZone
|   |-- Recorder/
|   |   `-- BP_FieldRecorder
|   |-- Objectives/
|   |   |-- BP_ObjectiveManager
|   |   `-- DA_ObjectiveData
|   `-- Jumpscare/
|       `-- BP_JumpscareManager
|
|-- Audio/
|   |-- Music/
|   |   |-- Prologue/
|   |   |-- Chapter1/
|   |   `-- Chapter2/               # Placeholder
|   |-- Ambience/
|   |   |-- Apartment/
|   |   |-- Village/
|   |   `-- Chapter2/               # Placeholder
|   |-- SFX/
|   |   |-- Typewriter/
|   |   |-- Footsteps/
|   |   |-- Creatures/
|   |   |   |-- Alan/
|   |   |   `-- Aswang/             # Placeholder
|   |   `-- UI/
|   |-- VO/
|   |   |-- Marco/
|   |   `-- NPCs/
|   |       |-- Chapter1/
|   |       |   |-- OldFarmer/
|   |       |   |-- YoungFarmer/
|   |       |   `-- MiddleAgedFarmer/
|   |       `-- Chapter2/           # Placeholder
|   `-- MetaSounds/
|       |-- MS_RadioWarp
|       |-- MS_TypewriterStrike
|       `-- MS_AlanWings
|
|-- VFX/
|   |-- NS_RainWindow
|   |-- NS_DeskLampFlicker
|   |-- NS_FeatherFall
|   |-- NS_InkBleed
|   `-- NS_ShadowPass
|
|-- Environment/
|   |-- Prologue/
|   |   |-- Meshes/
|   |   |-- Materials/
|   |   `-- Textures/
|   |-- Chapter1/
|   |   |-- Meshes/
|   |   |-- Materials/
|   |   `-- Textures/
|   |-- Chapter2/                   # Placeholder
|   `-- Shared/
|       |-- Meshes/
|       |-- Materials/
|       |-- Textures/
|       `-- Decals/
|
`-- DataTables/
    |-- DT_DialogueLines
    |-- DT_ObjectiveList
    `-- DT_ClueDatabase
```

---

## 6. Cinematic Pipeline

All cutscenes are built as Level Sequences inside Sequencer. Each sequence is named with a prefix identifying its chapter and order. Shared sequences are prefixed `SEQ_SHARED`.

### Dialogue Integration with Sequencer

Each `DA_DialogueLine` asset contains a `CameraHint` tag. When `BP_DialogueManager` advances to a new line, it broadcasts the tag to Sequencer which blends to the camera actor marked with that tag in the level. This allows dialogue to direct its own cameras without hardcoding references.

Dialogue is triggered from Sequencer via Event Tracks calling `BP_DialogueManager → PlayConversation(DA_DialogueConversation)`.

### Jumpscare Handoff — SEQ_C1_07

The Alan reveal sequence ends with a Sequencer Event Track firing `BP_JumpscareManager → ForcePOV()` which snaps the camera into Marco's eye socket. The final frame fires `BP_ChaseDirector → BeginChase()`, handing control back to gameplay. There is no fade. The cut is intentionally hard.

### Camera Settings by Scene Type

| Scene Type                 | Focal Length | Aperture | Movement                |
| -------------------------- | ------------ | -------- | ----------------------- |
| Prologue object inspection | 35mm - 50mm  | f/1.8    | Slow dolly, no handheld |
| NPC dialogue               | 50mm - 85mm  | f/2.8    | Static or slow push     |
| Creature reveal — wide     | 18mm         | f/8      | Static, low angle       |
| Creature reveal — close    | 85mm         | f/1.4    | Static, extreme close   |
| Forced POV (jumpscare)     | N/A          | N/A      | Hard snap, no blend     |

### Cutscene Sequence List

**Prologue**

| Asset Name                  | Description                                                                 |
| --------------------------- | --------------------------------------------------------------------------- |
| SEQ_P01_OpeningShot         | Desk lamp, rain on window, radio — slow camera drift across apartment       |
| SEQ_P02_ObjectReveal        | Camera lingers on Santo Nino, newspaper, photograph — each held 3-5 seconds |
| SEQ_P03_TypewriterJam       | Marco types, typewriter jams, rubs his eyes                                 |
| SEQ_P04_KnockAtDoor         | Three slow knocks — Marco freezes                                           |
| SEQ_P05_ParcelReveal        | Marco opens door, finds wet parcel in empty hallway                         |
| SEQ_P06_NotebookOpen        | Marco opens notebook — blank — then one sentence appears with ink bleed VFX |
| SEQ_P07_FeatherDrop         | Black feather slides from parcel onto desk                                  |
| SEQ_P08_TypewriterPossessed | Typewriter types by itself — slow, each key strike heavier than normal      |
| SEQ_P09_RadioWoman          | Radio turns on — woman humming — audio spatialized to behind Marco          |
| SEQ_P10_MarcoSpins          | Marco spins toward the sound — nothing there                                |
| SEQ_P11_SketchAppears       | Notebook page has changed — rough sketch of the Alan — new line underneath  |
| SEQ_P12_MapDecision         | Marco spreads map, traces route to Abra — "That's where it starts"          |

**Transitions**

| Asset Name         | Description                                                      |
| ------------------ | ---------------------------------------------------------------- |
| SEQ_T01_BusRide    | Loading screen sequence — fog road, radio drama fading to static |
| SEQ_T02_BusArrival | Marco steps off the bus — mountain fog — village ahead           |

**Chapter 1 — Abra**

| Asset Name                  | Description                                                                          |
| --------------------------- | ------------------------------------------------------------------------------------ |
| SEQ_C1_01_BusJournalWarm    | Marco finds a warm page on the bus — text appears — V.O. about the Alan              |
| SEQ_C1_02_VillageArrival    | Village establishing shot — too quiet, dogs refuse to bark                           |
| SEQ_C1_03_FarmerDialogue    | Three-farmer dialogue — cow death, shadow overhead, no one looks up                  |
| SEQ_C1_04_CliffApproach     | Night — Marco walks toward the shrine — wrong feeling                                |
| SEQ_C1_05_ShrineWrong       | Camera reveals shrine — bones sorted too neatly, jars, mixed remains                 |
| SEQ_C1_06_CameraFlash       | Marco photographs altar — flash frame — face in jar glass                            |
| SEQ_C1_07_AlanReveal        | Forced POV — Alan on cliff face — backwards, hands wrong — wings — hard cut to chase |
| SEQ_C1_08_MorningWakeup     | Marco wakes — dried blood under fingernails — recorder still running                 |
| SEQ_C1_09_RecorderPlayback  | Recorder plays: breathing, footsteps, wings, Marco's voice — "Don't look up"         |
| SEQ_C1_10_JournalNewDrawing | Journal opens — new sketch, closer, more detailed                                    |
| SEQ_C1_11_NextDestination   | Final line writes itself — "NEXT: BATANGAS"                                          |
| SEQ_C1_12_ShadowWindow      | Shadow passes outside window — no sound — fade to black                              |

**Chapter 2 — Batangas — Planned, sequences not yet created**

---

## 7. NPC Dialogue System

### DA_DialogueLine Data Asset Fields

| Field              | Type       | Description                                                    |
| ------------------ | ---------- | -------------------------------------------------------------- |
| SpeakerName        | FText      | Display name shown in dialogue box                             |
| DialogueText       | FText      | English dialogue line                                          |
| FilipinoSubtitle   | FText      | Filipino translation shown as subtitle                         |
| VoiceAudio         | USoundBase | Audio asset for lip sync                                       |
| PortraitTexture    | UTexture2D | Speaker portrait shown in UI                                   |
| CameraHint         | FName      | Tag that tells Sequencer which camera angle to use             |
| bIsPlayerTriggered | bool       | If true, only plays when Marco approaches NPC after main scene |
| bEndConversation   | bool       | If true, closes dialogue box after this line                   |

### Chapter 1 Village Dialogue

Filipino is the primary dialogue. English appears as subtitle.

**Beat 1 — Marco Approaches the Group**

```
OLD FARMER
Ibang tao.
(A stranger.)

OLD FARMER
Anong hanap mo dito?
(What are you looking for here?)

MARCO
Nagtatanong lang po.
Narinig ko may nangyari sa bundok.
(Just asking around. I heard something happened on the mountain.)

OLD FARMER
Ang shrine ay nagulo.
(The shrine was disturbed.)

OLD FARMER
Ang buto ay hindi nagpapatawad kapag inilipat.
(Bones don't forgive being moved.)
```

**Beat 2 — Young Farmer Steps Forward**

```
YOUNG FARMER
Yung baka ko — patay na ngayong umaga.
(My cow — dead this morning.)

MARCO
Paano namatay?
(How did it die?)

YOUNG FARMER
Wala. Walang sugat. Walang dugo.
Nakatayo lang — nanigas.
Parang nagyelo kung saan nakatayo.
(Nothing. No wounds. No blood.
Just standing there — stiff.
Like it froze where it stood.)

YOUNG FARMER
Pagkatapos pumunta ng Alan dito, nangyari 'yan.
(That happened after the Alan came here.)
```

**Beat 3 — Middle-Aged Farmer**

```
MIDDLE-AGED FARMER
Namamatay ang mga hayop.
(Animals die.)

YOUNG FARMER
Hindi ganyan. Hindi sa isang gabi.
(Not like that. Not overnight.)

MIDDLE-AGED FARMER
Sinabi mo rin noong nakaraang linggo,
tumigil na kumain ang baka mo.
(You said yourself last week your cow stopped eating.)

YOUNG FARMER
Iba 'yun—
(That's different—)

MIDDLE-AGED FARMER
Ang takot ay gumagawa ng mga pattern kahit wala naman.
(Fear makes patterns where there are none.)
```

**Beat 4 — Old Farmer Final Line + Shadow Event**

```
OLD FARMER
Marahil.
(Maybe.)

[Shadow passes overhead. No sound. No one looks up.]
```

**Optional Lines — Player Triggered**

```
OLD FARMER (on approach)
Huwag mong itanong kung sino.
Itanong mo kung bakit bumalik.
(Don't ask who went. Ask why they came back.)

YOUNG FARMER (on approach)
Hindi ko siya hinahanap.
Hinarap na niya ako.
(I wasn't looking for it. It faced me first.)

Nasa mata pa rin niya ako. Pakiramdam ko.
(I'm still in its eyes. That's how it feels.)

MIDDLE-AGED FARMER (on approach)
Mas matanda pa ako sa takot sa mga kwento.
(I'm older than the fear of stories.)

Pero ang mga kwento — hindi sila natatanda.
(But the stories — they don't get old.)
```

**Marco V.O. — Bus Scene**

```
MARCO (V.O.)
Ang Alan.
Mga mangangaso ng langit. Mga tagakolekta ng buto.
(The Alan. Sky hunters. Bone collectors.)

Sinabi ni Lola — hindi sila pumapatay nang walang dahilan.
(Lola said they don't kill without reason.)

Hindi niya sinabi kung ano ang itinuturing nilang dahilan.
(She never said what they consider a reason.)
```

---

## 8. Level Flow and Gameplay Structure

### Prologue Flow

```
SEQ_P01 through SEQ_P02 (automatic chain)
    |
[GAMEPLAY] Investigation — inspect Santo Nino, Newspaper, Photograph
    |
SEQ_P03 through SEQ_P11 (automatic chain on completion)
    |
[GAMEPLAY] Collect equipment: Field Recorder, Journal, Camera, Ritual Notes
    |
[GAMEPLAY] Find the map in the apartment
    |
SEQ_P12 — Marco traces route to Abra
    |
CUT TO BLACK — Loading Screen
```

### Chapter 1 Flow

```
SEQ_T01 Bus Ride (loading screen)
    |
SEQ_T02 Arrival
    |
[GAMEPLAY] Walk to village
    |
SEQ_C1_01 Journal warm page (triggered when Marco opens journal on bus)
    |
SEQ_C1_02 Village establishing shot
    |
[GAMEPLAY] Approach NPCs
    |
SEQ_C1_03 Farmer dialogue — shadow overhead
    |
[GAMEPLAY] Optional: approach individual farmers for secondary lines
    |
[GAMEPLAY] Walk to cliff shrine
    |
SEQ_C1_04 Cliff approach
SEQ_C1_05 Shrine investigation
    |
[GAMEPLAY] Inspect jar with bones
    |
[GAMEPLAY] Photograph the altar
    |
SEQ_C1_06 Camera flash — face in jar glass (trigger)
    |
SEQ_C1_07 Alan reveal — JUMPSCARE — wings — hard cut
    |
[GAMEPLAY] Chase sequence — no combat, escape only
    |
SEQ_C1_08 Morning wakeup
SEQ_C1_09 Recorder playback
SEQ_C1_10 Journal new drawing
SEQ_C1_11 "NEXT: BATANGAS" writes itself
SEQ_C1_12 Shadow on window
    |
CUT TO BLACK — Chapter 1 Complete
```

---

## 9. AI System — The Alan

The Alan uses `BT_Alan` with `BB_Alan` for state management. Three non-combat phases.

| Phase   | Behavior Tree Task         | Description                                                                                        |
| ------- | -------------------------- | -------------------------------------------------------------------------------------------------- |
| Observe | BTTask_Observe             | Alan clings to cliff face, tilts head, sniffs. Does not move toward Marco. Waits a fixed duration. |
| Follow  | BTTask_BeginFollow         | Alan moves toward Marco at slow speed. Speed increases via a Blackboard float driven by Timeline.  |
| Retreat | Triggered by ChaseDirector | Alan breaks off after Marco crosses the safe zone trigger. Does not pursue past village boundary.  |

`BTService_UpdateMarcoLocation` runs on a tick interval to update the Blackboard with Marco's position without requiring a direct character reference.

---

## 10. Chase System

`BP_ChaseDirector` is placed in `L_CliffShrine` and activates via `BeginChase()` from the final Event Track of `SEQ_C1_07`.

The director manages Alan's speed curve via Timeline, activates `BP_ChaseZone` trigger volumes that phase the encounter, handles the wind knockdown physics impulse on Marco, and detects the chase end condition when Marco reaches the village edge safe zone.

There is no fail state. The Alan cannot kill Marco. If Marco stops moving, Alan's observation behavior resets and the player recovers. The game creates dread, not frustration.

For Chapter 2, a new `BP_ChaseDirector` instance will be placed in the Batangas level and configured independently for the Aswang encounter.

---

## 11. Chapter Roadmap

| Chapter   | Location               | Creature   | Status         |
| --------- | ---------------------- | ---------- | -------------- |
| Prologue  | Manila, Apartment      | None       | In Development |
| Chapter 1 | Abra, Mountain Village | The Alan   | In Development |
| Chapter 2 | Batangas               | The Aswang | Planned        |
| Chapter 3 | TBA                    | TBA        | Planned        |
| Chapter 4 | TBA                    | TBA        | Planned        |

---

## 12. Adding a New Chapter — Developer Guide

When Chapter 2 is ready to begin, follow this checklist. Do not skip steps or create chapter assets inside existing chapter folders.

**Step 1 — Create folder placeholders**

All placeholder folders already exist in the structure. Do not rename or move them. Begin populating these:

```
Characters/Creatures/Aswang/
Levels/Chapter2/
Cinematics/Chapter2/
Dialogue/Chapter2/
Interactables/Chapter2/
Audio/Music/Chapter2/
Audio/Ambience/Chapter2/
Audio/SFX/Creatures/Aswang/
Audio/VO/NPCs/Chapter2/
Environment/Chapter2/
```

**Step 2 — Create the creature AI**

Inside `Characters/Creatures/Aswang/`, create:

- `BP_Aswang_Base` inheriting from the base Pawn class, not from `BP_Alan_Base`
- `BT_Aswang` and `BB_Aswang` — new independent Behavior Tree and Blackboard
- `BTTask_` and `BTService_` assets specific to Aswang behavior
- `ABP_Aswang` and `SK_Aswang` when mesh is ready

**Step 3 — Create the chase director**

Inside `Systems/Chase/`, the base `BP_ChaseDirector` already exists. For Chapter 2, create a child Blueprint `BP_ChaseDirector_Aswang` that extends the base and overrides speed curves, zone behavior, and any Aswang-specific events. Place it in `L_Chapter2_MainLevel`.

**Step 4 — Create levels**

Create new levels inside `Levels/Chapter2/`. Follow the same naming pattern as Chapter 1:
`L_[LocationName]`, `L_[KeyScene]`, `L_[EndingRoom]`

**Step 5 — Create cinematics**

Create Level Sequences inside `Cinematics/Chapter2/`. Use the prefix `SEQ_C2_` followed by a two-digit order number and a descriptive name. Mirror the Chapter 1 sequence naming pattern.

**Step 6 — Create dialogue assets**

Inside `Dialogue/Chapter2/`, create NPC subfolders and `Marco_VO/`. Follow the `DIA_[NPC]_Main` and `DIA_[NPC]_Optional` naming convention. Every line needs both an English `DialogueText` and a `FilipinoSubtitle` field populated.

**Step 7 — Add journal entry**

Create a new `DA_JournalEntry` asset for the Chapter 2 journal page. Set `bSelfWrites` to true. Reference it in the Chapter 2 ending Sequencer Event Track.

**Step 8 — Register new objective list entries**

Add Chapter 2 objectives to `DT_ObjectiveList`. Do not modify existing Chapter 1 entries.

**Step 9 — Update save game**

If Chapter 2 requires new persistent flags (e.g., bChapter2Complete, bAswangEncountered), add them to `BP_SaveGame` and `BP_GameInstance`. Do not repurpose existing Chapter 1 flags.

---

## 13. Recommended Plugins and Extensions

### Built-in UE5 Plugins — All Free

| Plugin                      | Purpose                                                                  |
| --------------------------- | ------------------------------------------------------------------------ |
| Enhanced Input              | Input mapping for Marco's movement, interaction, camera use, and run     |
| MetaSounds                  | Reactive procedural audio for typewriter, radio warping, creature sounds |
| Sequencer / Level Sequences | All cinematic cutscenes across all chapters                              |
| Chaos Physics               | Object interaction and Marco wind knockdown physics                      |
| Common UI                   | Cross-platform UI framework for menus and HUD                            |

### Marketplace Plugins

| Plugin                               | Source  | Cost | Purpose                                                                   |
| ------------------------------------ | ------- | ---- | ------------------------------------------------------------------------- |
| Easy Multi Save                      | Fab.com | Free | Save and load system for chapter progress, journal state, clue collection |
| Footstep System                      | Fab.com | Paid | Surface-aware footstep audio for wood floors, stone shrines, grass, mud   |
| Advanced Locomotion System Community | GitHub  | Free | Weighted character movement for Marco's cautious walk versus panic run    |

### External Tools

| Tool                               | Cost                | Purpose                                                             |
| ---------------------------------- | ------------------- | ------------------------------------------------------------------- |
| Fmod Studio                        | Free tier available | Adaptive spatial audio — the woman humming that shifts position     |
| Rider or VS Code with UE extension | Free                | C++ development for performance-critical systems in later chapters  |
| Fab.com environment packs          | Varies              | Philippine and Southeast Asian environment assets, fog and rain VFX |

---

## 14. Naming Conventions

| Asset Type            | Prefix      | Example                       |
| --------------------- | ----------- | ----------------------------- |
| Blueprint             | BP\_        | BP_Marco_Character            |
| Blueprint Interface   | BPI\_       | BPI_Interactable              |
| Animation Blueprint   | ABP\_       | ABP_Marco                     |
| Skeletal Mesh         | SK\_        | SK_Alan                       |
| Widget Blueprint      | WBP\_       | WBP_DialogueBox               |
| Data Asset            | DA\_        | DA_JournalEntry               |
| Data Table            | DT\_        | DT_DialogueLines              |
| Level Sequence        | SEQ\_       | SEQ_C1_07_AlanReveal          |
| Level                 | L\_         | L_CliffShrine                 |
| Behavior Tree         | BT\_        | BT_Alan                       |
| Blackboard            | BB\_        | BB_Alan                       |
| Behavior Tree Task    | BTTask\_    | BTTask_Observe                |
| Behavior Tree Service | BTService\_ | BTService_UpdateMarcoLocation |
| Input Action          | IA\_        | IA_Interact                   |
| Input Mapping Context | IMC\_       | IMC_Marco                     |
| Niagara System        | NS\_        | NS_InkBleed                   |
| MetaSound             | MS\_        | MS_RadioWarp                  |
| Dialogue Asset        | DIA\_       | DIA_OldFarmer_Main            |

---

## 15. Content Warning

This project contains horror imagery, jump scare sequences, death themes, depictions of bones and remains, and references to Philippine supernatural mythology. Some content may be disturbing to sensitive viewers.

The game references real Philippine folklore creatures and provincial geography. All folklore content is used with cultural intent and is not intended to be exploitative or reductive of Philippine cultural heritage.

---

_Developed in the Philippines. Built on stories that were never meant to be written down._
