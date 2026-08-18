# Generative AI & Agentic AI — n8n Workflow Projects

![n8n](https://img.shields.io/badge/n8n-workflow%20automation-EA4B71)
![LangChain](https://img.shields.io/badge/LangChain-agents-1C3C3C)
![OpenAI](https://img.shields.io/badge/OpenAI-chat%20model-412991)
![Projects](https://img.shields.io/badge/projects-2-blue)

Two working **agentic AI workflows** built in n8n — an LLM that drafts and publishes LinkedIn posts under human approval, and an agent that turns a mood into a real Spotify playlist.

Both are exported as importable `.json` workflows with a write-up alongside them. These are not diagrams; drop the JSON into n8n and they run.

**Agentic AI** here means the model does not just answer — it decides, calls tools, waits on a human, and acts on external systems.

---

## Projects

| Project | Trigger | What the agent does | Ends at |
|---|---|---|---|
| **1 — Automated LinkedIn Post** | Daily schedule (16:00) or manual | Drafts a post, emails it for approval, publishes only on a yes | LinkedIn |
| **2 — Mood-Based Spotify Playlist Generator** | Web form | Turns a mood into song recommendations, creates a playlist, resolves and adds every track | Spotify |

---

## Project 1 — Automated LinkedIn Post

`Project-1-LinkedIn post/`

```
Schedule Trigger (16:00) ─┐
                          ├─→ AI Agent ─→ Gmail: send & wait for reply ─→ If approved ─→ Create LinkedIn post
Manual Trigger ───────────┘      ↑
                    OpenAI Chat Model + Simple Memory
```

| Node | Role |
|---|---|
| `AI Agent` (LangChain) | Writes the post |
| `OpenAI Chat Model` | The model behind the agent |
| `Simple Memory` (buffer window) | Keeps recent context so posts do not repeat themselves |
| `Gmail — send message and wait for response` | **Human in the loop.** The workflow pauses here until a reply arrives |
| `If` | Publishes only on approval; otherwise the run ends quietly |

The interesting part is the pause. Most "AI posts for you" demos publish straight to the platform. This one blocks on a human reply before anything reaches a public profile — the difference between automation you can leave running and automation you have to watch.

---

## Project 2 — AI-Driven Mood-Based Spotify Playlist Generator

`Project-2 AI-Driven Mood-Based Spotify Playlist Generator/`

```
Mood Input Form → Store responses → AI Agent (+ Structured Output Parser)
      → Create Spotify Playlist → Extract Playlist ID
      → Split songs → Search track on Spotify → Format URI → Aggregate → Add tracks
```

| Stage | Nodes | Detail |
|---|---|---|
| Capture | `Mood Input Form`, `Store Form Responses` | Public form asks for a mood and a song count; responses persist to a data table |
| Reason | `AI Agent`, `OpenAI Chat Model`, `Simple Memory`, `Structured Output Parser` | The parser pins the model to a fixed shape — `playlist_name`, `mood`, `songs[{title, artist}]` |
| Build | `Create Spotify Playlist`, `Extract Playlist ID` | Creates the empty playlist and captures its ID |
| Resolve | `Split Song Recommendations`, `Search for Track on Spotify`, `Format Track URI`, `Aggregate Track URIs` | Each suggestion is looked up individually, converted to a Spotify URI, then batched |
| Write | `Add Tracks to Playlist` | One call with all URIs |

Two files are included: `Mood-Based Spotify Playlist Generator.json` (13 nodes) and `...updated.json` (15 nodes, adds branching and validation). Start from the updated one.

**Structured output parsing** is what makes this work. A free-text list of songs cannot be looped over; a schema-validated JSON array can. That single node is the difference between a chatbot and a pipeline.

---

## Running these

1. **n8n** — self-hosted or cloud. Import each `.json` via *Workflows → Import from File*.
2. **Credentials** — set up your own in n8n; none are stored in these exports:
   - OpenAI API key (both projects)
   - LinkedIn OAuth (project 1)
   - Gmail OAuth (project 1)
   - Spotify OAuth (project 2)
3. **Re-point the nodes** at your credentials, then run manually once before enabling the schedule or publishing the form URL.
4. Project 1 will email *you* for approval — check the recipient on the Gmail node before the first run.

---

## What these two projects are actually demonstrating

- **Tools beat prompts.** The value is not the model's prose; it is that the model can create a playlist and search a catalogue.
- **Force the shape early.** Structured output at the agent boundary removes an entire class of downstream parsing bugs.
- **Put a human where the cost is irreversible.** A bad playlist can be deleted. A bad LinkedIn post has already been seen.

---

## Repository layout

```
Project-1-LinkedIn post/
  Automated LinkedIn Post.json          importable n8n workflow
  LinkedIn post.docx                    write-up
Project-2 AI-Driven Mood-Based Spotify Playlist Generator/
  Mood-Based Spotify Playlist Generator.json            first build (13 nodes)
  Mood-Based Spotify Playlist Generator updated.json    current build (15 nodes)
  AI-Driven Mood-Based Spotify Playlist Generator.docx  write-up
Documents/                              reserved
```
