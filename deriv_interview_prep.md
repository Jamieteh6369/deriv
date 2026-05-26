# Deriv AI Engineer — 60-Minute Assessment Prep Sheet

You already have the engineering. This sheet is for the *performance* of it: getting from a blank file to a working agentic tool fast, while narrating and managing the clock.

---

## 1. Before you click "Join Interview" (pre-flight, ~5 min)

- [ ] AI tool open and tested with a real prompt (don't discover it's broken on the clock)
- [ ] API key ready and confirmed working — run one test call *before* joining
- [ ] Camera + mic working, screen share tested
- [ ] Empty GitHub repo created and set to **public** (you submit by pushing here)
- [ ] Editor + terminal open and arranged so screen share looks clean
- [ ] Python env ready: `pip install fastapi uvicorn openai anthropic requests`

> Note on the API key: it almost certainly means *your own* key for the LLM provider you'll drive (OpenAI or Anthropic). Never paste a key tied to sensitive billing into anything you don't control. Use a key with a low spend cap.

---

## 2. The time budget (don't skip this)

| Phase | Time | What you're doing |
|---|---|---|
| Read + scope out loud | ~8 min | Restate the problem, state your plan, name your assumptions |
| Build core path | ~30 min | Get ONE thing working end-to-end |
| Harden + handle edge cases | ~10 min | Error handling, the "what if the LLM returns garbage" moment |
| Clean up + push | ~10 min | README, commit, push to public repo |

**The rule that wins:** a small working thing beats an ambitious broken thing. Get the happy path working *first*, then improve. Don't gold-plate.

---

## 3. Starter skeleton — adapt this to whatever they ask

This is a minimal LLM-with-tool-calling loop in a stack you already know. It's deliberately simple so you can read every line out loud and extend it live. Swap OpenAI for Anthropic/Groq trivially — the shape is the same.

```python
# app.py — minimal agent with tool calling
import os, json
from fastapi import FastAPI
from pydantic import BaseModel
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
app = FastAPI()

# --- 1. Define a tool the model can call ---
def get_data(query: str) -> str:
    """Replace this with whatever the task needs:
    a lookup, a calculation, an API call, a DB query."""
    return f"Pretend result for: {query}"

TOOLS = [{
    "type": "function",
    "function": {
        "name": "get_data",
        "description": "Look up data for a given query",
        "parameters": {
            "type": "object",
            "properties": {"query": {"type": "string"}},
            "required": ["query"],
        },
    },
}]

# --- 2. The agent loop ---
def run_agent(user_message: str, max_steps: int = 5) -> str:
    messages = [{"role": "user", "content": user_message}]
    for _ in range(max_steps):  # cap steps so it can't loop forever
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=messages,
            tools=TOOLS,
        )
        msg = resp.choices[0].message
        if not msg.tool_calls:
            return msg.content  # model is done, return its answer
        messages.append(msg)
        for call in msg.tool_calls:
            args = json.loads(call.function.arguments)
            result = get_data(**args)        # actually run the tool
            messages.append({
                "role": "tool",
                "tool_call_id": call.id,
                "content": result,
            })
    return "Stopped: hit max steps."  # safety fallback

# --- 3. Expose it ---
class Ask(BaseModel):
    message: str

@app.post("/ask")
def ask(body: Ask):
    return {"answer": run_agent(body.message)}
```

Run it: `uvicorn app:app --reload` then POST to `/ask`.

**Why this skeleton scores well:** it has a tool, a bounded loop (`max_steps`), and a fallback. That bounded loop + fallback is the "production reliability" signal Deriv keeps asking for — most candidates write an unbounded `while True`.

### If it's a RAG / "answer questions about this doc" task
Strip the tools, and instead: load the doc → chunk it → embed → retrieve top-k → stuff into the prompt as context. Even a dead-simple keyword match for retrieval is fine if you're short on time and you *say* "I'd swap this for embeddings given more time."

### If it's "fix/extend this repo"
Spend the first 5 min reading it out loud before touching anything. Narrating your read of unfamiliar code is itself a scored skill.

---

## 4. The "narrate out loud" cheat sheet

You go silent exactly when they most want to hear you (when it's hard). Fight that. Keep talking. Templates:

**When scoping:**
> "Okay, so the task is X. My plan is to get a minimal version working first — just the happy path — then add error handling. I'm assuming Y; I'll flag if that's wrong."

**When you reach for the AI (do this openly — it's encouraged):**
> "I'll have the AI scaffold this part, but I specifically want to check how it handles [empty input / a failed API call], because that's where these break."

**The money moment — catching the AI being wrong:**
> "The AI gave me this, but it's not handling the case where the model returns no tool call — let me fix that." 

This single move is what gets an *AI engineer* hired. They pay you to be the human who makes AI output reliable. Manufacture at least one of these moments on purpose.

**When stuck (don't go quiet):**
> "This isn't working as expected. Let me think about why — I'll check the request payload first, then the response shape."

**When you make a tradeoff:**
> "I'm choosing the simple approach here to stay in time. In production I'd do [embeddings / retries with backoff / a queue], but for now this proves the concept."

**Lean on your actual background — say these things, they're true for you:**
- "I've shipped a chatbot with SSO and guardrails before, so I'm wary of X here."
- "I've done red-teaming on LLM apps, so let me make sure this input can't be abused."

That red-teaming line is rare and credible. Use it.

---

## 5. Your mock run (do this once before the real thing)

1. Set a 60-min timer.
2. Camera on, screen recording on (so you feel watched — record yourself).
3. Give yourself a fake task, e.g.: *"Build an agent that takes a question, decides whether to call a 'search' tool, and returns an answer. Handle the case where the tool fails."*
4. Build it from the skeleton above **while talking the entire time.**
5. Push to a public GitHub repo before the timer ends.
6. Watch 2 minutes of your recording. The only thing you're checking: **did you ever go silent?** If yes, that's the habit to fix.

---

## 6. Common failure modes to avoid

- **Passive AI use** — pasting output and errors back and forth without understanding. Disqualifying for this role.
- **Going silent under pressure** — the #1 scoring killer.
- **Chasing perfection** — running out of time with nothing pushed beats a modest thing that's committed.
- **Unbounded loops / no error handling** — the opposite of the "last mile" reliability they want.
- **Not pushing in time** — commit early, commit often; don't leave submission to the final minute.

---

*You're not under-prepared on capability. Rehearse the narration and one timed run, and you're ready.*
