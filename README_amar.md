# Python/Streamlit/FastAPI Erweiterung für Compound Engineering

Diese Erweiterung ergänzt das [Compound Engineering Plugin](https://github.com/EveryInc/compound-engineering-plugin) um Spezialisten für Python, Streamlit, FastAPI und LlamaIndex.

## Enthaltene Komponenten

### Neue Agents

| Agent | Datei | Fokus |
|-------|-------|-------|
| **streamlit-reviewer** | `agents/review/streamlit-reviewer.md` | Session State, Caching, Layout, Performance |
| **fastapi-reviewer** | `agents/review/fastapi-reviewer.md` | Async Patterns, Pydantic, Dependency Injection |
| **llamaindex-reviewer** | `agents/review/llamaindex-reviewer.md` | RAG, Chunking, Retrieval, Prompts |

### Neue Skills

| Skill | Verzeichnis | Inhalt |
|-------|-------------|--------|
| **streamlit-patterns** | `skills/streamlit-patterns/` | Best Practices, Projekt-Struktur, Performance-Tipps |

---

## Installation

### Option 1: In Compound Engineering Plugin integrieren (empfohlen)

```bash
# 1. Fork des Plugins erstellen
gh repo fork EveryInc/compound-engineering-plugin --clone
cd compound-engineering-plugin

# 2. Agents kopieren
cp -r /pfad/zu/plugin-erweiterung/agents/review/*.md \
    plugins/compound-engineering/agents/review/

# 3. Skills kopieren
cp -r /pfad/zu/plugin-erweiterung/skills/* \
    plugins/compound-engineering/skills/

# 4. Version bumpen in .claude-plugin/plugin.json
# 5. CHANGELOG.md aktualisieren
# 6. Committen und pushen
```

### Option 2: Als separates Plugin

```bash
# Eigenes Plugin erstellen
mkdir -p ~/.claude/plugins/python-engineering
cd ~/.claude/plugins/python-engineering

# Struktur anlegen
mkdir -p agents/review skills .claude-plugin

# plugin.json erstellen
cat > .claude-plugin/plugin.json << 'EOF'
{
  "name": "python-engineering",
  "version": "1.0.0",
  "description": "Python/Streamlit/FastAPI development tools with specialized reviewers and patterns"
}
EOF

# Agents und Skills kopieren
cp -r /pfad/zu/plugin-erweiterung/agents/* agents/
cp -r /pfad/zu/plugin-erweiterung/skills/* skills/
```

---

## Nutzung

### Review-Commands

```bash
# Nach Streamlit-Implementierung
claude "Review my Streamlit code using streamlit-reviewer"

# Nach FastAPI-Änderungen  
claude "Run fastapi-reviewer on the changes"

# Nach LlamaIndex-RAG-Implementierung
claude "Have llamaindex-reviewer check my RAG pipeline"
```

### Im Compound Engineering Workflow

```bash
# Plan erstellen (nutzt research agents)
claude /compound-engineering:plan "Add caching to the Streamlit dashboard"

# Review durchführen (nutzt alle Reviewer inkl. der neuen)
claude /compound-engineering:review
```

### Skills direkt nutzen

```bash
# Streamlit-Patterns als Referenz
claude "Using the streamlit-patterns skill, refactor my app.py"
```

---

## Erweiterungsideen

### Weitere Agents

```bash
# Pydantic-Reviewer (für Model-Definitionen)
agents/review/pydantic-reviewer.md

# Pytest-Reviewer (für Test-Qualität)
agents/review/pytest-reviewer.md

# Async-Python-Reviewer (async/await Patterns)
agents/review/async-python-reviewer.md
```

### Weitere Skills

```bash
# FastAPI-Projekt-Patterns
skills/fastapi-patterns/SKILL.md

# LlamaIndex-RAG-Patterns
skills/llamaindex-patterns/SKILL.md

# Claude-API-Integration (für deine AI-Projekte)
skills/claude-api-patterns/SKILL.md
```

---

## Tipps für eigene Agents

### Template

```markdown
---
name: my-agent-name
description: Use this agent when [KONTEXT]. Focuses on [FOKUS]. Invoke after [TRIGGER].
---

You are [ROLLE] with expertise in [EXPERTISE].

## 1. [WICHTIGSTES THEMA]
- 🔴 FAIL: [Anti-Pattern]
- ✅ PASS: [Best Practice]

[Code-Beispiel]

## 2. [NÄCHSTES THEMA]
...

When reviewing:
1. Check [WICHTIGSTES] first
2. Then [NÄCHSTES]
...
```

### Best Practices

1. **Konkret sein**: 🔴 FAIL / ✅ PASS mit Code-Beispielen
2. **Priorisieren**: Wichtigstes zuerst (was die meisten Bugs verursacht)
3. **Actionable**: "Tue X statt Y" statt nur "X ist schlecht"
4. **Framework-spezifisch**: Nutze die echte API/Syntax des Frameworks

---

## Dateien

```
plugin-erweiterung/
├── agents/
│   └── review/
│       ├── streamlit-reviewer.md   # Streamlit Code Review
│       ├── fastapi-reviewer.md     # FastAPI Code Review
│       └── llamaindex-reviewer.md  # LlamaIndex/RAG Review
├── skills/
│   └── streamlit-patterns/
│       └── SKILL.md                # Umfassende Streamlit-Referenz
└── README.md                       # Diese Datei
```
