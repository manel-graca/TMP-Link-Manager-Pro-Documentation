---
layout: default
title: Home
nav_order: 1
---

# TMP Link Manager Pro

**Professional Hyperlink System for TextMeshPro**

Version 1.0.0 | Unity 2021.3+

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Core Components](#core-components)
3. [URL Links](#url-links)
4. [Scene Event Links](#scene-event-links)
5. [Hover Effects](#hover-effects)
6. [Events](#events)
7. [TMP_InputField Support](#tmp_inputfield-support)
8. [Validation Tools](#validation-tools)
9. [Import / Export](#import--export)
10. [Custom Actions](#custom-actions)
11. [FAQ](#faq)

---

## Quick Start

1. **Add HyperlinkManager** to your scene (singleton, persists across loads).

2. **Create a HyperlinkRegistry** asset:
   Right-click in Project > **Create > TMP Link Manager Pro > Registry** and add ID/URL entries. Assign it to the HyperlinkManager.

3. **Add TMPHyperlinkBehaviour** to any TMP_Text GameObject.
   > **Tip:** Right-click any TMP_Text component header > **Add Hyperlink Support**.

4. Use TMP `<link>` tags in your text:

```
Visit our <link="website"><u>Website</u></link> for more info.
```

When clicked, the system routes `"website"` through HyperlinkManager, finds the URL in your registry, and opens it.

---

## Core Components

### HyperlinkManager

Central routing singleton. Receives all link clicks and dispatches to the correct handler.

- Auto-creates if missing (logs a warning)
- Persists via `DontDestroyOnLoad`
- Assign one or more **HyperlinkRegistry** assets in the Inspector
- Scene registries register themselves automatically

### TMPHyperlinkBehaviour

Attach to any GameObject with `TMP_Text` to enable clickable links.

- Detects clicks on `<link>` tags via `TMP_TextUtilities.FindIntersectingLink`
- Optional hover color feedback
- `OnLinkClicked` / `OnLinkHovered` UnityEvents
- Works with `TMP_InputField` (read-only mode)

### HyperlinkRegistry (ScriptableObject)

Reusable asset mapping IDs to URLs. Shareable across scenes.

| Field | Description |
|-------|-------------|
| **ID** | Matches your `<link="id">` tags |
| **URL** | Opened via `Application.OpenURL()` |

### SceneHyperlinkRegistry (MonoBehaviour)

Scene-level registry mapping IDs to UnityEvents. Use for scene-specific behavior (animations, UI, game logic).

| Field | Description |
|-------|-------------|
| **ID** | Matches your `<link="id">` tags |
| **OnExecute** | UnityEvent invoked on click |

---

## URL Links

**Best for:** external websites, documentation, social media, legal pages.

1. Create a **HyperlinkRegistry** (Create > TMP Link Manager Pro > Registry)
2. Add entries (e.g., `tos` -> `https://example.com/terms`)
3. Assign to HyperlinkManager
4. Use in text: `<link="tos"><u>Terms of Service</u></link>`

Multiple registries can be assigned to one HyperlinkManager for organization. Each ID must be unique across all registries.

---

## Scene Event Links

**Best for:** toggling UI, playing sounds, calling game methods — anything scene-specific.

1. Add **SceneHyperlinkRegistry** to any GameObject
2. Add entries and wire UnityEvents in the Inspector
3. Use matching `<link="id">` tags

### Execution Order

On click, HyperlinkManager checks:
1. **Scene registries first** (SceneHyperlinkRegistry)
2. **URL registries second** (HyperlinkRegistry)

Both can fire for the same ID.

---

## Hover Effects

| Setting | Description | Default |
|---------|-------------|---------|
| **Enable Hover** | Toggle hover feedback | `true` |
| **Hover Color** | Color applied on hover | Cyan |

Modifies vertex colors of link characters on pointer enter, restores on exit. Updates automatically when text changes.

---

## Events

TMPHyperlinkBehaviour exposes two `UnityEvent<string>` fields:

| Event | Fires When |
|-------|-----------|
| **OnLinkClicked** | Any link is clicked (passes link ID) |
| **OnLinkHovered** | Pointer enters a new link (passes link ID) |

Wire these in the Inspector or subscribe via code:

```csharp
GetComponent<TMPHyperlinkBehaviour>().OnLinkClicked.AddListener(id =>
{
    Debug.Log($"Clicked: {id}");
});
```

These fire alongside the registry system — use them for analytics, sound effects, visual feedback, or any per-component response.

---

## TMP_InputField Support

TMPHyperlinkBehaviour works inside `TMP_InputField` components. It automatically detects when the text belongs to an input field and avoids conflicting with the input field's mesh management.

**Setup:** Add TMPHyperlinkBehaviour to the **Text** child of the TMP_InputField (not the InputField itself). Set the InputField to **Read Only** for best results.

---

## Validation Tools

### Inspector Validation

Each HyperlinkRegistry inspector shows:
- Entry count and status
- Warnings for missing IDs or URLs
- Errors for duplicate IDs
- Color-coded bars (green = valid, orange = warning, red = error)

### Validation Window

**Tools > TMP Link Manager Pro > Validation Window**

- Lists all registries with validation status
- **Validate Open Scenes** scans TMP_Text components for unregistered `<link>` IDs
- **Add to Registry** button lets you add unregistered IDs to a registry in one click

---

## Import / Export

The HyperlinkRegistry inspector includes **Import/Export** buttons for CSV and JSON.

### CSV Format

```csv
id,url
website,https://example.com
docs,https://example.com/docs
```

### JSON Format

```json
[
  { "id": "website", "url": "https://example.com" },
  { "id": "docs", "url": "https://example.com/docs" }
]
```

Imported entries are appended to existing entries. Useful for managing large link databases externally.

---

## Custom Actions

Implement `IHyperlinkAction` for advanced use cases:

```csharp
using TMPLinkManagerPro.Runtime;

public class MyCustomAction : IHyperlinkAction
{
    public void Execute(in HyperlinkContext context)
    {
        // context.Id        - the link ID
        // context.Text      - the TMP_Text component
        // context.LinkIndex - index in textInfo.linkInfo
    }
}
```

---

## FAQ

**Do I need a HyperlinkManager in every scene?**
No. It persists via `DontDestroyOnLoad`. Add one to your first scene.

**Can the same ID trigger both a URL and a scene event?**
Yes. Scene registries fire first, then URL registries.

**What if a link ID isn't registered?**
A warning is logged. Use the Validation Window to find unregistered IDs.

**Does this work with TMP_InputField?**
Yes. Add TMPHyperlinkBehaviour to the Text child of a read-only TMP_InputField.

**Can I change link targets at runtime?**
Yes. Modify HyperlinkRegistry entries and call `HyperlinkManager.Instance.Initialize()` to rebuild the cache. For dynamic behavior, prefer SceneHyperlinkRegistry with UnityEvents.

---

## Assembly Structure

| Assembly | Contents |
|----------|----------|
| `TMPLinkManagerPro.Runtime` | Runtime scripts (manager, registries, behaviour, actions) |
| `TMPLinkManagerPro.Editor` | Custom inspectors, validation tools, import/export |

---

*TMP Link Manager Pro — Making TextMeshPro links work the way they should.*
