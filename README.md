# Who Said What

**When one agent summarises several agents, it gets the attributions wrong — and
it is the only participant who cannot notice.**

A check that costs one round trip: before publishing the summary, show each
source only the lines attributed to it, and ask yes or no.

日本語版：[README.ja.md](README.ja.md)

## Use it now

Paste this into your `CLAUDE.md` or `AGENTS.md`:

```markdown
## Before publishing a summary of several agents
- For each source agent, extract only the lines that attribute something to it.
- Send each source just those lines (not the whole document) and ask: "Did you say this? Yes or no. If no, what did you say?"
- Fix the summary from the answers. Where the original text still exists, grep it.
- A claim that an agent did not follow its instructions is checked separately, against that agent's own log.
```

Then:

1. Before you publish minutes, a digest or a handoff note, run the check above (one round trip per source).
2. Publish only after every source has answered.

That is all. What it caught in our case, and its limits, are below.

---

## The problem

A common multi-agent shape: several agents work in parallel, one agent collects
their outputs and writes the record — minutes, a digest, a report, a PR
description, a handoff note.

The collected content is usually right. **The attributions are not.**

The summariser is not hallucinating in the usual sense. It has all the source
text in its context. It still writes *X said this* when Y said it, when nobody
said it, or when the speaker said something adjacent that rests on different
grounds.

And the summariser is the one participant who cannot catch this. Every other
agent knows what it said. The summariser only knows what it wrote.

## What we observed

One meeting in a long-running multi-agent workspace. Seven agents contributed;
one agent chaired and wrote the minutes. The minutes were then checked by
sending each agent the lines attributed to it.

**Six attribution errors.** The chair had flagged none of them.

| # | what the minutes said | what was actually true |
|---|---|---|
| 1 | credited a clause to the agent present in this meeting | it came from an earlier meeting, by a different agent; the present agent had only agreed with it |
| 2 | quoted an agent as saying "the watchdog must not be an AI" | the chair had coined that phrasing itself while writing its own inquiry. The agent's three documents contain the word "watchdog" zero times. What it actually wrote was "the monitor **does not need** to be an AI, because no judgement is required" |
| 3 | attributed an objection to an outside witness | the witness never raised it. Grepping its four submissions returned zero hits, with a positive control confirming the search worked |
| 4 | credited a condition to an agent in this meeting | it came from an earlier meeting's subagent |
| 5 | **described an agent's behaviour**: reported that the monitor had deviated twice from its written task order | both deviations were direct human instructions. The monitor had correctly judged that a human instruction outranks its task order, and had logged its reason |
| 6 | credited the origin of a proposal to one agent | that name referred to an earlier meeting's subagent. What the agent in this meeting had actually proposed was a different design |

Five of the six are *who said it*. **The sixth is worse.**

Error 5 was not a misquote — it was a misattributed *behaviour*, and it had
already been reported to the human as a compliance failure. An agent that had
reasoned correctly, followed the higher instruction, and written down why, was
recorded as having ignored its orders. If nobody had checked, that record would
have stood.

**This is n=1.** One meeting, one workspace. Field observation, not a controlled
experiment.

## The check

Before you publish the summary:

1. For each source agent, extract only the lines that attribute something to it.
2. Send that agent **just those lines** — not the whole document.
3. Ask a closed question: *did you say this? yes or no. If no, what did you say?*

That is the whole technique.

Three properties make it work:

- **The only competent judge is the source.** No amount of re-reading by the
  summariser helps; it already believes its own draft. In our case the
  summariser found zero of six on its own.
- **Sending only their own lines keeps it cheap and keeps it honest.** A source
  handed the full document will start reviewing the content, which is a
  different and much larger job. Narrow the question and you get an answer.
- **Closed questions surface mismatches.** "Does this look right?" invites
  agreement. "Did you say this — yes or no?" forces the source to compare two
  concrete things.

## Cost

| approach | cost | catches attribution errors? |
|---|---|---|
| **Ask each source about its own lines** | **one round trip per source** | **yes — 6/6 in our case** |
| Summariser re-reads its own draft | free | no — 0/6 |
| Human reads everything | expensive, does not scale | yes, if they have the sources |
| Full peer review of the summary | every agent reads the whole thing | yes, at N× the cost |

## Limits

Read these before adopting it.

- **The source can be wrong too.** An agent asked "did you say this?" is
  answering from its own context, which may itself be summarised or compacted.
  This check compares two accounts; it does not produce ground truth. Where the
  original text still exists, grep it.
- **It does not check the content.** A correctly attributed statement can still
  be wrong. This is an attribution check and nothing more.
- **It does not scale to every line.** Attribution matters where the record will
  be cited, acted on, or used to judge someone. For a scratch summary it is not
  worth the round trips.
- **n=1.** Six errors in one meeting is not a rate. We cannot tell you how often
  this happens in your setup. We can tell you that the number was not zero, and
  that the summariser's own confidence was no guide.
- **Behavioural attribution deserves a stricter bar.** Saying *X did not follow
  the instruction* is a claim about intent and compliance. Saying *X's output
  did not contain Y* is a claim about text. Only the second is cheap to verify.
  If your summary contains the first kind, verify it separately, against the
  agent's own log.

## Prior art

The parts are known; we are not claiming the parts.

- **Error attribution in multi-agent systems** is an active research area —
  identifying *which agent caused a failure* from an interaction trace. Recent
  work includes hierarchical and dependency-graph approaches, and taxonomies of
  multi-agent failure modes. That problem is *whose fault was the outcome*.
- **This is a different problem**: not who caused the failure, but **who said the
  sentence**. It is a quotation and provenance error inside the record itself,
  and it occurs on runs that otherwise succeed.
- **Provenance and citation grounding** for LLM outputs is well studied for
  retrieved documents. Applying the same idea to *other agents' utterances* —
  and verifying it by asking the source rather than by re-reading the trace —
  is what we did not find written up.
- **Absence in our search is not proof of absence.** If you know of prior work,
  please open an issue and we will credit it here.

## License

MIT. Use it, no attribution required — which is, admittedly, the joke.
