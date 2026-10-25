# UCR-04 Corgi Customization — Use-Case Realization

| Author | Realizes |
| --- | --- |
| Tony | [UC-04 Corgi Customization](../use_cases/UC-04_Corgi_Customization.md) |

## 1 Introduction

This realization describes how the system loads, previews, validates and saves the selected Corgi's visual appearance and decorative items as specified in UC-04. It covers the basic flow (UC-04 steps 1–9) and alternative flows 3a, 5a, 3b/5b, 7a and 8a. It does not define functional equipment management (UC-07), purchasing/unlocking, exact locked-option presentation or save-failure recovery.

## 2 Flow of events — design

### 2.1 Participating objects

| Object | Stereotype | Responsibility |
| --- | --- | --- |
| `:CustomizationPopup` | «boundary» | Presents the shared customization pop-up, receives player requests, renders visual preview and offers Save / Discard / Continue editing when the player leaves with pending changes. |
| `:CustomizeCorgiVisuals` | «control» | Coordinates the UC-04 workflow, manages pending **visual** selections, checks availability and commits visual selections only on successful confirmation. Functional state is not modified by this control. |
| `:VisualOptionCatalog` | «entity» | Represents the visual options and their availability to the player. Availability rules and locked-option presentation remain undefined. |
| `:CorgiVisualProfile` | «entity» | Stores the selected Corgi's persisted body appearance and visual-item loadout, independently of functional equipment and stats. |

**Actor:** `Player` interacts with the boundary only. `:CustomizationPopup` is a **shared** boundary with UC-07; it must take independent pending changes from both areas into account when offering Save / Discard / Continue editing. The `:CustomizeCorgiVisuals` control itself checks and modifies only the visual draft. The following analysis models only the visual workflow of UC-04 and the cross-use-case confirmation/cancellation semantics explicitly stated in its requirements.

**Selection constraints:** The six appearance attributes are fur colour, fur pattern, paw colour, snout colour, eye colour and tail colour. Decorative items can be equipped, replaced or removed in applicable independent visual slots; at most one item occupies each visual slot. Pending changes are previewed, not immediately persisted. No visual selection changes character stats, abilities or functional equipment.

### 2.2 Sequence diagrams

#### Basic flow and visual-option alternatives

![UCR-04 refined sequence diagram](../diagrams/UCR-04_sequence.png)

#### Leaving with unsaved changes (UC-04 alternative 7a)

![UCR-04 exit alternative refined sequence diagram](../diagrams/UCR-04_sequence_exit.png)

The Task-03 diagram using frontend/backend/database remains unchanged in [UC-04, section 4.3](../use_cases/UC-04_Corgi_Customization.md). The diagrams above refine it using analysis-level **boundary**, **control** and **entity** objects. Names are analysis concepts, not prescribed implementation class names; concrete design classes belong to Task 05.

### 2.3 Walkthrough

1. **Open/load (UC-04 steps 1–2):** The Player asks `:CustomizationPopup` to open Visual Customization. `:CustomizeCorgiVisuals` loads the selected Corgi's stored visual profile and the available visual choices from the two entities. The boundary then displays the current appearance and the **Corgi Appearance** / **Visual Items** sub-tabs.
2. **Change and preview (steps 3–6, alternatives 3a/5a):** The Player proposes a body option or equips, replaces or removes a decorative item. The control checks whether the selection is allowed; if so, it updates only that pending visual slot and the boundary renders the live preview. A second selection for the same slot replaces the earlier pending selection; removing an item clears the visual slot. These operations do not affect any functional item, stat or ability.
3. **Unavailable choices (alternatives 3b/5b):** When the option is unavailable, the control rejects the proposal and the pending selection remains unchanged. The Player may choose another option. No specific locked-item presentation is assumed.
4. **Confirm/save (steps 7–9):** Upon confirmation, the control validates pending visual selections and asks `:CorgiVisualProfile` to persist the selected Corgi's appearance and decorative items. Success is reported only after a successful save. When functional changes are also pending, the **same shared confirmation** must apply to UC-07 independently; the functional workflow and the inter-workflow failure/atomicity rules are not specified here.
5. **Exit with unsaved changes (alternative 7a):** When the Player tries to close the shared pop-up, the system checks pending changes across **both** areas. If changes exist, it offers Save (use the confirmation flow), Discard (discard both pending drafts without modifying saved data) or Continue editing (retain both drafts in the pop-up). If nothing is pending, the pop-up can close normally.
6. **Failed save (alternative 8a):** The boundary does not show a successful save. The exact error display, retry procedure, draft retention and cross-use-case rollback policy remain undecided rather than being invented in the diagrams.

### 2.4 State machine

Not applicable. The use case does not currently define a durable lifecycle state (such as `approved`/`cancelled`) for a visual profile; **saved vs. pending** refers to the working copy versus persistent data, not to a defined entity state machine. This can be revisited if explicit lifecycle states are added.

### 2.5 Design classes (Task 05)

**Status: proposed design.** The supplied package contains design documents but no application source code. Therefore the following class and method names are a **concrete implementation plan**, not a claim that these classes already exist. When implementation begins, keep the code and diagrams consistent. The [`UC-04 domain model contribution`](../domain_model_UC-04_draft.md) is **not yet the complete team-wide domain model**.

#### Mapping from Task-04 analysis objects to design classes

| Analysis object (section 2.1) | Design class(es) | Rationale |
| --- | --- | --- |
| `:CustomizationPopup` «boundary» | `CustomizationPopupView`, `CustomizationSession` | Rendering and player input remain in the view; the shared session coordinates Save / Discard across UC-04 and UC-07. |
| `:CustomizeCorgiVisuals` «control» | `CorgiVisualService`, `VisualDraft` | Service validates visual changes; draft holds the unsaved working copy, so the preview never directly changes the persisted state. |
| `:VisualOptionCatalog` «entity» | `VisualOptionCatalog` | Provides choices and checks their availability without specifying how unlocking or acquisition works. |
| `:CorgiVisualProfile` «entity» | `CorgiVisualProfile`, `VisualProfileRepository` | Domain profile contains saved visual selections; repository abstracts loading and storing them. |

`FunctionalCustomizationPort` is a **proposed integration interface for UC-07**, not another UC-04 analysis object. Its exact implementation belongs to UC-07. It allows the **shared** `CustomizationSession` to ask about pending functional changes and coordinate confirmation/discarding while `CorgiVisualService` remains visual-only. `Corgi` identifies the already selected character and is not created or selected by this use case.

#### Class diagram

![UCR-04 design class diagram](../diagrams/UCR-04_classes.png)

The arrows represent permanent references for view/session/service/repositories and dashed *dependencies* for shorter-lived uses. The diagram specifies **planned** operations that are called in the design-level sequences below. `VisualProfileRepository` and `FunctionalCustomizationPort` are interfaces/ports rather than confirmed implemented classes.

#### Redrawn sequence diagrams using the design classes

![UCR-04 design-level sequence diagram](../diagrams/UCR-04_sequence_design.png)

![UCR-04 design-level exit sequence diagram](../diagrams/UCR-04_sequence_exit_design.png)

The two **original Task-04 analysis diagrams remain above in section 2.2**. In contrast, these Task-05 diagrams use concrete planned class names and operations. All messages directed to those classes correspond to an operation shown in the class diagram. The `VisualDraft` remains unsaved until an explicit confirmation; changing a cosmetic option must not alter functional stats or equipment.

The domain draft models individual appearance and decorative-slot selections explicitly, whereas the proposed code-oriented `CorgiVisualProfile` may store the same choices in maps. Those are different views of the **same persisted information**, not extra game features.

**Open team decisions, unchanged from Task 04:** option-availability policy and presentation, failed-save recovery, and the transactional behavior when visual and functional changes are saved together. A combined success must not be shown if one required save failed; the proposed sequence **does not guarantee atomic rollback** if the other save already succeeded. The UC-07 interface and shared coordinator must be agreed with the UC-07 author before implementation.

## 3 Derived requirements

**None.** The realization makes explicit the existing UC-04 requirements (independent visual persistence, slot replacement, pending preview, shared confirmation and cancellation) without adding new design obligations. These points still need team decisions before implementation:

- How unavailable or locked visual choices are displayed and which ownership/unlock rules apply.
- What recovery action, if any, is offered after a failed save.
- Whether saving simultaneous UC-04 and UC-07 changes is atomic, or how partial failures are handled.

These are **open decisions**, not new confirmed requirements.
