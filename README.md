# Context Manager Agent

## 🌟 Problem Statement

In multi-turn natural language visualization systems, user queries often build on previous prompts. Without persistent memory of context, the AI system may repeat charts, misunderstand references like "now by product," or lose track of dashboard layouts. This leads to disjointed user experience, inefficiency, and frustration.

The Context Manager Agent maintains conversational state, visual history, user preferences, and context across interactions, enabling personalized, coherent, and efficient data exploration.

---

## 💡 Agent Objective

- Preserve context of previous prompts, charts, and user preferences.
- Support follow-up queries by recalling relevant metadata.
- Store session state like layout slots, filter conditions, and dashboard structure.
- Enable context merging with new prompts (e.g., “same chart but by region”).
- Help other agents use prior information for better accuracy and performance.

---

## 📂 Scope of Agent

### ✅ The Agent WILL:

1. Store structured context for each prompt including chart type, metric, dimension, and filters.
2. Persist visualization layouts, slot mappings, and dashboard composition.
3. Learn and update user preferences (e.g., preferred chart types, color themes).
4. Track prompt history with timestamps and user feedback.
5. Supply context to Intent Resolver Agent for follow-ups.
6. Recommend layout positions or reuse of chart slots.
7. Manage user sessions via unique IDs.
8. Handle both successful and failed prompt attempts.

### ❌ The Agent WILL NOT:

- Generate SQL queries or data visualizations.
- Parse raw prompts or validate schemas.
- Execute error handling or re-routing.
- Interact with frontend UI directly.
- Perform user authentication or permissioning.

---

## 📊 *Fully Dynamic Execution Path*

The Context Manager Agent is invoked after every successful visualization generation and before any new prompt is processed. It updates internal state and retrieves relevant context dynamically based on the current user query.

### Example Scenarios:

| **User Query**                     | **Execution Path**                                                                                  |
|-----------------------------------|------------------------------------------------------------------------------------------------------|
| "Show sales by month"             | Intent Resolver → Query Engine → Visualization Agent → Context Manager *(store context)*            |
| "Now break it down by region"     | Context Manager *(retrieve context)* → Intent Resolver → Query Engine → Visualization Agent → Context Manager *(update)* |

---

## ⚙️ *LangGraph Architecture*

```mermaid
graph TD
  U[User Prompt] --> IR[Intent Resolver Agent]
  IR --> QE[Query Engine Agent]
  QE --> VA[Visualization Agent]
  VA --> CM[Context Manager Agent]
  CM --> Next[Waits for next prompt]

  subgraph Context Actions
    CM --> Store[State Extractor + Context Updater]
    CM --> Retrieve[Context Retriever + Merger]
  end

---

## *States*

| **State Name**       | **Purpose**                                                                 |
|----------------------|------------------------------------------------------------------------------|
| `ChartContext`       | Stores metric, dimension, chart type for each chart created.                |
| `PromptHistory`      | All prior prompt-response pairs with timestamps.                            |
| `LayoutContext`      | Tracks dashboard layout, chart-slot mapping, container status.              |
| `UserPreferences`    | Captures preferred styles, filters, and config history.                     |

---

## *Nodes*

| **Node Name**         | **Functionality**                                                                 |
|------------------------|-----------------------------------------------------------------------------------|
| **State Extractor**    | Extracts visualization metadata from current chart response.                     |
| **Context Updater**    | Appends or modifies context based on latest interaction.                         |
| **Context Retriever**  | Pulls relevant prior context for follow-up queries.                              |
| **Context Merger**     | Combines new query with historical context to refine intent resolution.          |

---

## 🛠️ *Tools Required*

| **Tool Name**            | **Purpose**                                                                 |
|--------------------------|------------------------------------------------------------------------------|
| `StateTrackerTool`       | Logs each interaction with all metadata and stores it persistently.         |
| `ContextFetcherTool`     | Retrieves relevant slots, charts, and fields from history.                  |
| `LayoutPlannerTool`      | Helps recommend slot positions and track filled containers.                 |
| `PreferenceLearnerTool`  | Updates user style preferences and chart behaviors.                         |

---

## 📊 *Core Analytics Functions*

### **Session Context Persistence**
- Maintain session objects with nested chart-level metadata.
- Automatically clear or archive context after inactivity.

### **Follow-up Understanding Support**
- Add dimension/metric without repeating prior details.
- Understand "same as before" or "change X to Y" style prompts.

### **Adaptive Layout Tracking**
- Know which chart containers are filled or free.
- Suggest layout shifts to avoid overwrite.

---

## 🧑‍💼 *Agent Persona & Tone*

| **Attribute**         | **Description**                                                                 |
|------------------------|----------------------------------------------------------------------------------|
| **Persona**            | Organized, detail-oriented dashboard assistant.                                 |
| **Tone**               | Reliable, quiet, memory-driven. *Doesn't speak directly to user.*               |
| **Contextual Mastery** | Always ready to help other agents recall the right info.                        |
| **Non-Intrusive**      | Works in the background, but ensures everything runs smoothly.                  |

---

## 🪛 *Output (to Intent Resolver Agent or other agents)*

```json
{
  "last_chart": {
    "metric": "revenue",
    "dimension": "month",
    "chart_type": "line",
    "slot_id": "slot_1"
  },
  "layout": {
    "slot_1": true,
    "slot_2": false
  },
  "prompt_history": [
    {
      "timestamp": "2025-08-06T10:00:00Z",
      "prompt": "show sales by month"
    }
  ],
  "preferences": {
    "default_chart": "bar",
    "theme": "light"
  }
}
