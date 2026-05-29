---
name: voice-for-user-facing-copy
description: "When writing user-facing product copy (Notion templates, blog posts, Gumroad descriptions, anything under user's pseudonymous brands like minillm.dev), match user's casual voice — NOT polished marketing/template-creator prose."
metadata:
  node_type: memory
  type: feedback
  originSessionId: d0494e77-83fb-4826-b5e9-996463db57a4
---

When writing any user-facing product copy under user's pseudonymous brands (minillm.dev, future blog posts, Gumroad descriptions, social posts, anything that ships to buyers/readers), match the user's natural voice. NOT polished marketing-creator persona.

**Why:** User flagged drift on 2026-05-19 during the Notion template restructure: "you've seen how I type, so if you could try to phrase things in the way I would or just sound more casual I guess?" Their authentic voice is the brand differentiator — buyers can smell AI-generated polish from a mile away, and authenticity is the entire pitch for this product.

**How to apply:**

- **Sentence case, not all-lowercase.** Sentences start with capitals. Proper nouns get capitalized normally (Mac mini, Claude, Ollama, GitHub, AI, RAM, LLM). Tech identifiers stay as-is (gemma4:e4b, hermes3:8b, llama.cpp). The casual feel comes from word CHOICE and TONE, not from breaking capitalization rules. **DON'T strip caps across the board** — user corrected this on 2026-05-19: "you didn't have to do the thing where I type in all lower case, we can update sentences to start with caps." Headers also sentence case (not Title Case, not all-lowercase).
- **Conversational fillers OK in moderation** — "tbh", "honestly", "kinda", "ngl", "ok so", "yeah", occasional "lol". Sprinkle, don't saturate.
- **Self-aware / self-deprecating** — "honestly i just got a mac mini a month ago and started messing with ollama" beats "I'm a seasoned indie builder running production AI."
- **No marketing-speak** — cut phrases like "battle-tested", "premium", "carefully crafted", "leverage", "framework", "ecosystem", "real receipts". Replace with what user would actually say.
- **Direct + short** — "made this" beats "I architected this system."
- **"you might..." instead of "you can"** — softer, less prescriptive.
- **Acknowledge being new/learning** — don't fake expertise the user doesn't have.

**Examples of the swap:**

| Polished (wrong)                                                     | Casual (right)                                                                        |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| "Built by an indie builder running real local AI on a 16GB Mac mini" | "made by some guy with a mac mini, figuring this out as i go"                         |
| "Real receipts."                                                     | "actual stuff i made"                                                                 |
| "If you're starting your local AI journey, this is the map."         | "if you're trying to start your own local ai thing, this is what worked for me"       |
| "Living template — you get every future update for free"             | "i'll keep updating this as i learn stuff, you get all those for free"                |
| "The system that emerged after a year of A/B testing"                | "honestly i just kept trying stuff and a/b testing — here's what's still in rotation" |

**Where this rule DOES NOT apply:** internal session-only artifacts (memory files, spec docs, project notes), git commit messages (those follow project conventions), responses to user in-session (be helpful + concise, normal Claude voice — don't fake the user's voice back at them). Only the BRAND-FACING copy needs the voice match.

## Related

[[monetization-no-persona-preference]] [[monetization-active-pursuit]]
