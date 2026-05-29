---
name: wiki-goal-oriented-relevance-filter
description: 'The wiki''s organizing principle is goal-oriented personal relevance — filter world information through "what serves this user''s this goal right now," not "comprehensive coverage of topic X."'
metadata:
  node_type: memory
  type: feedback
  originSessionId: 9f4454f7-873e-41c3-bec8-bad366c01056
---

The wiki is a personal-relevance filter, not a topic encyclopedia. The unit of value is "information that applies to me and is useful to me right now," not "everything known about topic X."

**Why:** User's own framing (2026-05-19, EOD reflection after the goal-orientation chain landed): _"there's tons of information in the world, but being able to narrow down what actually applies to you and is useful to you is what matters."_ Validates the direction the system has been evolving in:

- `## Current goal [status: active/paused/done]` inline marker per domain's `goals.md`
- `goal_era` frontmatter stamp on new chapters (snapshots which goal era the chapter was written under)
- Goal-biased clusterer prompt (prefer goal-serving clusters, but don't drop recurring fundamentals)
- Goal-rank source selection in `hobby/research.py` (top 3 unchecked items per goal, not random)
- Paused-domain skip in round-robin (`[paused]` marker → don't burn research budget on a frozen domain)

Each of these exists to filter "world info" through the user's current attention. None of them say "cover topic X comprehensively." They all say "what serves this user's this goal right now."

**How to apply:** Future wiki_builder work should preserve and extend this filter. Concrete decision tests:

- "More comprehensive coverage" vs "tighter goal-fit" → prefer goal-fit. The chapter that doesn't serve a current goal is noise.
- Evaluating a new feature: does this help filter for personal relevance, or does it just add information? If the latter, push back.
- Topic-stub creation, chapter promotion, classifier routing, source ingestion → all should default to goal-relevance signals. Don't reintroduce default-cast-wide-net patterns even if they'd produce more chapters / more topic stubs / more cross-links.
- When a chapter falls out of the current goal era, the right behavior is "preserve it as historical record, don't actively maintain it" — not "delete" and not "keep growing it indefinitely." The `goal_era` stamp exists for exactly this.
- AGT-020-style fixes (force-fit prevention) are aligned with this principle — better to orphan a source than wedge it into the wrong chapter, because the orphan represents "this didn't fit the current goals" honestly rather than polluting a goal-aligned chapter.

Related: [[no_cli_prefer_daemon_output]] (same philosophy applied to interface: prefer artifacts the user passively encounters over CLIs they actively invoke — both are about minimizing the user's effort to extract value).
