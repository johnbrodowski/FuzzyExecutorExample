# Fuzzy Executor

This C# example demonstrates how to execute natural language commands on UI elements using fuzzy matching. Designed for my computer uses framework and open sourced as a portfolio example of intelligent automation, it demonstrates parsing, approximate string matching, and command execution.

Key point: This method can be paired with any high-quality speech-to-text (STT) engine to control applications entirely via voice, without requiring AI—commands are interpreted and executed purely through deterministic logic.

---

## Features

* 🔹 **Natural language commands** – “calculator 5 click” or “notepad file save click”
* 🔹 **Fuzzy window & element matching** – works even if names are imprecise
* 🔹 **Supports multiple actions** – click, double-click, right-click, select, type, scroll, drag, etc.
* 🔹 **Full logging** – trace every step of the fuzzy execution
* 🔹 **Standalone classes** – easy to include in any C# project

---

## Quick Start

### Example Usage

```csharp
var fuzzyResult = await _bridge.ExecuteCommand("FUZZY calculator 5 click");

// Example command breakdown:
// "calculator" → window hint
// "5" → element hint
// "click" → action
```

---

## 1. Parsing Commands

Commands are parsed into **window, element, and action** parts:

```csharp
private (string windowHint, string elementHint, string action)? ParseFuzzyCommand(string fuzzyCommand)
{
    var words = fuzzyCommand.Split(' ', StringSplitOptions.RemoveEmptyEntries);
    if (words.Length < 3) return null;

    var knownActions = new[] { "click", "double-click", "doubleclick", "right-click", "rightclick",
                               "type", "select", "scroll", "drag", "set", "minimize", "maximize", "close", "open" };

    string action = "";
    int actionIndex = -1;

    for (int i = words.Length - 1; i >= 0; i--)
    {
        if (knownActions.Contains(words[i].ToLowerInvariant()))
        {
            action = words[i].ToLowerInvariant();
            actionIndex = i;
            break;
        }
    }

    if (string.IsNullOrEmpty(action) || actionIndex == -1)
    {
        action = words[^1].ToLowerInvariant();
        actionIndex = words.Length - 1;
    }

    if (actionIndex < 2) return null;

    string windowHint = words[0];
    string elementHint = string.Join(" ", words.Skip(1).Take(actionIndex - 1));

    return (windowHint, elementHint, action);
}
```

---

## 2. Window & Element Matching (Fuzzy Logic)

```csharp
private string? FindBestMatch(string query, List<string> options)
{
    if (options == null || options.Count == 0) return null;

    string? bestMatch = null;
    double bestScore = 0;

    foreach (var option in options)
    {
        var score = CalculateSimilarity(query, option);
        if (score > bestScore)
        {
            bestScore = score;
            bestMatch = option;
        }
    }

    return bestScore >= 0.3 ? bestMatch : null;
}

private double CalculateSimilarity(string source, string target)
{
    if (string.IsNullOrEmpty(source) || string.IsNullOrEmpty(target)) return 0;

    source = source.ToLowerInvariant();
    target = target.ToLowerInvariant();

    if (source == target) return 1.0;
    if (target.Contains(source)) return 0.9;
    if (source.Contains(target)) return 0.85;

    int distance = LevenshteinDistance(source, target);
    int maxLength = Math.Max(source.Length, target.Length);
    return Math.Max(0, 1.0 - ((double)distance / maxLength));
}

private int LevenshteinDistance(string source, string target)
{
    if (string.IsNullOrEmpty(source)) return target?.Length ?? 0;
    if (string.IsNullOrEmpty(target)) return source.Length;

    int[,] distance = new int[source.Length + 1, target.Length + 1];

    for (int i = 0; i <= source.Length; i++) distance[i, 0] = i;
    for (int j = 0; j <= target.Length; j++) distance[0, j] = j;

    for (int i = 1; i <= source.Length; i++)
    {
        for (int j = 1; j <= target.Length; j++)
        {
            int cost = (source[i - 1] == target[j - 1]) ? 0 : 1;
            distance[i, j] = Math.Min(
                Math.Min(distance[i - 1, j] + 1, distance[i, j - 1] + 1),
                distance[i - 1, j - 1] + cost);
        }
    }

    return distance[source.Length, target.Length];
}
```

> Matches below 30% similarity are ignored.

---

## 3. Mapping Actions

```csharp
private string MapActionToCommand(string action, int elementId)
{
    return action.ToLowerInvariant() switch
    {
        "click" => $"CLICK {elementId}",
        "double-click" or "doubleclick" => $"DOUBLE_CLICK {elementId}",
        "right-click" or "rightclick" => $"RIGHT_CLICK {elementId}",
        "select" => $"SELECT {elementId}",
        _ => $"CLICK {elementId}"
    };
}
```

---

## 4. Execute Fuzzy Command

```csharp
var fuzzyResult = await ExecuteFuzzyCommand("calculator 5 click");
Console.WriteLine(fuzzyResult.Data);
```

* Finds the closest window match.
* Searches all elements in the window.
* Selects the element with the highest similarity.
* Maps the action to an executable command.
* Returns detailed logs for inspection.

---

## 5. Example Commands

| Command                   | Description                         |
| ------------------------- | ----------------------------------- |
| `calculator 5 click`      | Click the button “5” in Calculator  |
| `notepad file save click` | Click “Save” under the File menu    |
| `browser search bar type` | Type in the browser search bar      |
| `calculator + click`      | Click the plus button in Calculator |

---

## 6. Notes

* Uses **fuzzy string matching** with substring and Levenshtein distance.
* Fully self-contained – just include the classes in your project.
* Logs every step for debugging and demonstration purposes.

