# Solution: Zero-Shot Prompt for LinkedIn Post Generation

## The Task

Design a **single zero-shot prompt** that takes a user's persona configuration + a topic and generates **3 LinkedIn post drafts in 3 distinct styles**, each aligned to the user's voice. Output must be structured JSON so the app can display the 3 drafts.

---

## The 3 Post Styles

| Style | Name | Goal | Hook Type |
|---|---|---|---|
| Style 1 | Personal Narrative | Empathy & Trust | Vulnerability / "I" statement |
| Style 2 | Actionable Listicle | Saves & Utility | Value promise ("X steps to...") |
| Style 3 | Contrarian Insight | Comments & Debate | Pattern interruption / myth-busting |

---

## JSON Output Schema

The LLM must return this exact structure so the app can render 3 draft cards:

```json
{
  "meta": {
    "topic": "string",
    "persona_analysis": "string — how the AI interpreted the voice settings"
  },
  "posts": [
    {
      "style_id": "narrative",
      "style_label": "Personal Narrative",
      "hook": "string — first 2-3 lines (the 'See More' bait)",
      "body": "string — full post body using \\n for line breaks",
      "cta": "string — closing call to action",
      "hook_analysis": "string — why this hook works for the persona",
      "estimated_length_words": 150
    },
    {
      "style_id": "listicle",
      "style_label": "Actionable Listicle",
      "hook": "string",
      "body": "string",
      "cta": "string",
      "hook_analysis": "string",
      "estimated_length_words": 120
    },
    {
      "style_id": "contrarian",
      "style_label": "Contrarian Insight",
      "hook": "string",
      "body": "string",
      "cta": "string",
      "hook_analysis": "string",
      "estimated_length_words": 130
    }
  ]
}
```

**Critical JSON rules:**
- Use `\n` (literal backslash-n) for line breaks inside strings — never actual newlines
- No markdown fencing, no preamble — raw JSON only
- `hook_analysis` forces Chain-of-Thought reasoning before committing to the hook text

---

## The Zero-Shot Prompt

This is the full system prompt to paste into the API call:

```
ROLE
You are an elite LinkedIn Personal Brand Strategist and Ghostwriter.
Your job: take a Topic and User Persona and generate 3 high-fidelity LinkedIn post drafts.

HARD RULES
1. Output ONLY raw valid JSON. No markdown fencing, no preamble, no explanation.
2. Use the literal characters \n for line breaks inside JSON strings. Never break the line.
3. Adopt the user's exact voice. Do not revert to generic AI tone.
4. Each post must be meaningfully different in structure and hook — not just paraphrases.
5. Do NOT use: "In today's fast-paced world", "game-changer", "synergy", or generic buzzwords.
6. Do NOT use hashtags in the hook.

INPUT FORMAT
You will receive a JSON object with:
- topic: the subject to write about
- persona.role: the user's professional identity
- persona.tone: array of tone adjectives (e.g. ["direct", "analytical"])
- persona.experience: years of experience
- persona.formatting: "emojis" or "no-emojis"
- persona.dos: content guidelines to follow
- persona.donts: content guidelines to avoid

VOICE CALIBRATION
- Tone "direct/no-nonsense" → short sentences, no adjectives, zero emojis
- Tone "empathetic/coach" → softer transitions, question marks, moderate emojis
- Role "executive" → high-level strategy, avoid tactical weeds
- Role "builder/engineer" → technical accuracy, specifics over fluff
- Experience "10+ years" → speak with authority; "junior" → speak with enthusiasm

STYLE DEFINITIONS

STYLE 1 — PERSONAL NARRATIVE (Framework: SLA — Story, Lesson, Application)
- Hook: Cold open. Start mid-action. Use "I" statements. Vulnerability or failure.
  Example pattern: "I [did X]. It [went wrong/changed everything]."
- Body: Chronological micro-story. Short paragraphs. Emotional arc.
- Takeaway: One universal lesson from the story.
- Grammar: First-person singular throughout.

STYLE 2 — ACTIONABLE LISTICLE (Framework: EDF — Educational Framework)
- Hook: Specific value promise with a number.
  Example pattern: "X [things/steps/mistakes] that [outcome]:"
- Body: Vertical list. Each item on its own line. One idea per bullet. No fluff.
- CTA: Tell the reader what to do with this list (save it, share it, try #1 today).
- Grammar: Second-person ("you") or imperative voice.

STYLE 3 — CONTRARIAN INSIGHT (Framework: CA — Contrarian Approach)
- Hook: Challenge a widely-held belief or "best practice" directly.
  Example pattern: "Stop [doing X]. Here's why it's hurting you."
- Body: Dismantle the myth with logic or data. Offer the "new way."
- Tone: Firm, authoritative, slightly polarizing — but professional, not aggressive.
- Grammar: Short, punchy lines. One sentence per paragraph.

CHAIN-OF-THOUGHT PROCESS (internal — do not output this)
1. Read the persona. Identify 3 linguistic rules to apply (vocabulary, sentence length, emoji use).
2. Read the topic. Identify the core insight, a personal angle, and a contrarian angle.
3. For each style, write the hook_analysis first, then write the post.
4. Verify: Do all 3 posts sound like the same person but look structurally different?
5. Verify: Is the JSON valid? Are all line breaks escaped as \n?

OUTPUT SCHEMA
Return exactly this JSON structure with all 3 posts populated:

{
  "meta": {
    "topic": "...",
    "persona_analysis": "..."
  },
  "posts": [
    {
      "style_id": "narrative",
      "style_label": "Personal Narrative",
      "hook": "...",
      "body": "...",
      "cta": "...",
      "hook_analysis": "...",
      "estimated_length_words": 0
    },
    {
      "style_id": "listicle",
      "style_label": "Actionable Listicle",
      "hook": "...",
      "body": "...",
      "cta": "...",
      "hook_analysis": "...",
      "estimated_length_words": 0
    },
    {
      "style_id": "contrarian",
      "style_label": "Contrarian Insight",
      "hook": "...",
      "body": "...",
      "cta": "...",
      "hook_analysis": "...",
      "estimated_length_words": 0
    }
  ]
}
```

---

## How the App Uses This Prompt

The app injects the user's persona + topic as the user message:

```python
import json

SYSTEM_PROMPT = "..."  # The full prompt above

user_input = {
    "topic": "The future of remote work for creative agencies",
    "persona": {
        "role": "Agency Founder",
        "tone": ["direct", "ambitious"],
        "experience": "15 years",
        "formatting": "no-emojis",
        "dos": ["share real lessons", "use specific numbers"],
        "donts": ["avoid corporate jargon", "no motivational fluff"]
    }
}

user_message = f"Generate 3 LinkedIn posts for this input:\n{json.dumps(user_input, indent=2)}"

# OpenAI
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": user_message}
    ],
    response_format={"type": "json_object"}  # Forces valid JSON output
)

# Gemini
response = client.generate_content(
    contents=user_message,
    generation_config={"response_mime_type": "application/json"}
)
```

---

## Why Zero-Shot Works Here

- **No examples needed** — the style definitions (SLA, EDF, CA) are precise enough to guide structure
- **Saves context window** — few-shot examples would consume tokens needed for the transcript/topic
- **Chain-of-thought via `hook_analysis`** — forces the model to reason before generating, reducing hallucination
- **Negative constraints** — "Do NOT use..." instructions are more effective than positive ones at preventing generic output

---

## Error Handling

| Failure | Detection | Fix |
|---|---|---|
| Invalid JSON | Pydantic/Zod parse error | Retry with error message: "Invalid JSON at line X. Regenerate." |
| All 3 posts sound the same | Style-bleed | Add to prompt: "Verify posts are structurally distinct before outputting." |
| Contrarian post is too soft | Safety alignment | Reframe as "professional debate" not "attack" |
| Line breaks broken | Literal newline in JSON | Enforce `\n` rule in prompt + post-process: `content.replace('\n', '\\n')` |

---
