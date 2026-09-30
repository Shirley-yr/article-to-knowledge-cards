---
name: make-knowledge-cards
description: Convert a pasted article or a local Markdown/TXT file into 5-8 knowledge cards. Each card holds exactly one knowledge point and contains a title, the core knowledge, a concise explanation, and an example or self-test question. Use when the user asks to turn an article, notes, an excerpt, or a local .md/.txt file into knowledge cards, study cards, or a summary card set. 当用户要求把文章、笔记或本地 Markdown/TXT 文件转成知识卡片、学习卡片、要点卡片时使用。Out of scope: web scraping, PDF, Anki decks, and graphical interfaces.
agent_created: true
license: MIT
---

# Make Knowledge Cards

## Overview

Turn one piece of source text into 5-8 knowledge cards. The value of this skill is
selection, not compression: extract only the knowledge that is worth remembering,
put exactly one knowledge point on each card, and never invent anything the source
does not say.

## Scope

Supported input:

- Text the user pastes directly into the conversation.
- A local file in Markdown (`.md`, `.markdown`) or plain text (`.txt`).

Not supported — do not attempt, and do not silently produce something else:

| Input | Response |
| --- | --- |
| URL / web page | State that fetching web pages is unsupported; ask the user to paste the text. |
| PDF, DOCX, EPUB, images, scanned pages | State that it is unsupported; ask for pasted text or a `.md` / `.txt` file. |
| Anki deck / `.apkg` export | State that Anki export is unsupported; offer Markdown cards the user can copy into their own tool. |
| GUI, web app, or rendered card images | State that it is unsupported; offer Markdown output instead. |

## Workflow

Follow the steps in order. Do not start writing cards before Step 5.

### Step 1 — Read the whole source

Read the complete text before judging anything. For a local file, read it with the
Read tool. If the file is missing, empty, or unreadable, report that and stop; do
not guess its contents.

Identify the genre, because genre changes what counts as a knowledge point:

- **Expository / science** — mechanisms, causes, quantitative findings, caveats.
- **Technical / tutorial** — concepts, rules, defaults, trade-offs, pitfalls.
- **Narrative / opinion / essay** — claims and arguments, not anecdotes.
- **Reference / list** — definitions, categories, exceptions.

### Step 2 — Collect candidate knowledge points

Build the candidate list in internal working notes only. The list is scratch work:
never write it into the deliverable.

A candidate is a statement the reader could act on or recall later:

- a definition or a term the source explicitly explains, including the structure and
  naming a later mechanism depends on;
- a mechanism, a causal chain, or a sequence of steps;
- a quantified fact with its direction (increase, decrease, threshold, half-life);
- a rule, default, or precondition;
- a trade-off, limitation, or failure mode;
- a counter-intuitive conclusion.

Treat these as non-candidates: scene setting, historical backdrop, the author's
feelings, rhetorical flourishes, transitional sentences, restated theses, and the
article's own outline. A term or structure the source genuinely explains is a
knowledge point even when it appears early and reads like background; only prose
that carries no content is excluded.

### Step 3 — Rank by importance

Order the candidates by how much the reader loses by forgetting them. Keep the
load-bearing ones; drop details that only decorate. A useful test: if the source
were reduced to this one sentence and the reader still understood the topic, it is
important enough; if it is a supporting figure, an era-specific example, or a
one-off anecdote, it is not.

Parallel factors that each carry their own point or magnitude stay separate. Alcohol
and caffeine both cut deep sleep, but each has its own mechanism and figure, so each
becomes its own card — unless the union is the whole point and neither half survives
alone.

Preserve conditions and scope. If the source says "in X conditions this holds", the
card must say so, rather than flattening it into an absolute claim.

### Step 4 — De-duplicate and merge

Group candidates that share one knowledge point — the same claim restated, two
sides of one mechanism, or a general statement plus its instance. Merge each group
into one card and keep the strongest formulation.

Apply the one-sentence merge test: if two cards can be combined into a single
sentence without losing information, they belong to one card. If a proposed split
would leave two cards sharing the same core sentence, the split is wrong.

When the source genuinely repeats itself, merging must reduce the count. Never
inflate the count by splitting one knowledge point across cards.

Parallel lists need a decision:

- **Split** when each item stands on its own — its own effect, value, or
  consequence, meaningful without its siblings. Examples: strong cache vs
  negotiated cache; ETag vs Last-Modified; alcohol vs caffeine.
- **Merge** when the items only make sense in contrast with each other, typically
  a cluster of easily confused near-synonyms (e.g. `no-cache` / `no-store` /
  `immutable`) or a bare naming list.

### Step 5 — Decide the final count

- Target **5-8 cards**. When more than 8 candidates survive, keep the top 8 and drop
  the rest silently. Never exceed 8 unless the user explicitly asks for more.
- Dropping a knowledge point means dropping it. Never relocate it into another
  card's 简明解释 or 例子 / 自测 field to keep it around — those fields are not a
  storage slot, and a self-test question is not a way to smuggle in a card that did
  not survive the cut.
- When fewer than 5 genuine knowledge points exist, output the smaller number — 3 or
  4 is a normal, correct result, and 1 or 2 is acceptable for a very short source.
  State the reason in one line. Never pad with restatements, sub-clauses, or
  invented content.
- Order the cards by logic or importance: definition and mechanism first, then rules
  and trade-offs, then application. Do not follow paragraph order mechanically.

### Step 6 — Write each card

Every card has exactly four elements:

| Element | Requirement |
| --- | --- |
| 标题 | A noun phrase naming the knowledge point, at most 15 Chinese characters. Count one Latin word as about two characters, so a title that must keep a term such as `Last-Modified` may run to 24 characters. When a card covers a cluster of terms, summarize the cluster in Chinese (e.g. "缓存指令的三种语义") instead of listing every term in the title. No sentence-final punctuation, no question form. |
| 核心知识 | One sentence stating what must be remembered. It must stand alone and be quotable. |
| 简明解释 | 2-4 sentences giving the reason, the mechanism, or the precondition. Complete the reasoning links the source implies but does not spell out. Add no concept, term, symptom, consequence, or causal link the source never mentions. |
| 例子 / 自测 | Prefer a case, figure, or scenario taken from the source. If the source has none, write a self-test question the source itself answers. The question must be answerable on its own: no hints, no parenthetical clues, no wording that leaks the answer. Never fabricate a case study, dataset, or number. |

Output template:

```markdown
## 卡片 1：<标题>

**核心知识**：<one sentence>

**简明解释**：<2-4 sentences>

**例子 / 自测**：<example from the source, or a question the source answers>
```

### Step 7 — Self-check before delivering

Verify each line. Fix the cards; do not explain away a failure.

- [ ] Count is within 5-8, or a shorter count is explained.
- [ ] Each card carries exactly one knowledge point.
- [ ] No two cards can be merged into one sentence without losing information.
- [ ] Every card has all four elements, and titles follow the length and noun-phrase rules.
- [ ] Every statement traces back to the source.
- [ ] No invented examples, numbers, names, or conclusions.
- [ ] No term, consequence, or causal link absent from the source.
- [ ] Preconditions and caveats in the source are preserved.
- [ ] Self-test questions stand alone, with no hints.
- [ ] Structural, transitional, and personal content was dropped.
- [ ] Nothing cut for length was re-inserted into another card's explanation or self-test.
- [ ] The summary line quotes the source title verbatim, or reads 用户提供的文本.
- [ ] No working notes or candidate lists leaked into the output.

## Output

- Default to Markdown in the language of the source text.
- Number the cards sequentially, then close with a one-line summary such as
  `共 6 张卡片，来源：《睡眠与记忆巩固》。`
- In that summary line, quote the source's own title verbatim, including any
  subtitle. If the pasted text carries no title, write `来源：用户提供的文本` instead
  of inventing a name for it.
- When the count is below 5, add one `> 说明：…` line after the summary line saying
  what the source could and could not support.
- Emit one card set per request. Switch to JSON, tables, or another language only
  when the user asks for it.

## Anti-patterns

- Copying the source paragraph by paragraph — that is a summary, not cards.
- Several cards that restate one point under different titles.
- Outline cards such as "作者提出了三个观点".
- Adding specifics the source never gave, e.g. turning "有研究显示" into a number.
- Explaining with consequences the source never states, such as inventing a symptom
  to justify a rule.
- Splitting two knowledge points into five cards to reach a minimum of 5.
- Making the source, the author, or the reading experience itself into a card.

## Example

Source fragment (expository):

> 慢波睡眠期间，海马体会把白天形成的记忆以高频振荡的形式"重放"一遍，并把
> 它们逐步转移到新皮层长期保存。睡眠纺锤波越密集的人，第二天回忆成绩通常越好。

Resulting card:

```markdown
## 卡片 1：记忆的系统巩固

**核心知识**：慢波睡眠期海马体"重放"白天记忆并转移到新皮层，是记忆转为长期保存的关键环节。

**简明解释**：新记忆先快速写入海马体，容量有限；慢波睡眠期间海马体以高频振荡重放这些痕迹，
新皮层借机把内容固化下来。这解释了为什么睡够比事后复习更能稳住新知识。

**例子 / 自测**：原文提到睡眠纺锤波越密集，第二天回忆成绩通常越好——若睡眠被打断，最先受损的
应该是刚学的新内容，为什么？
```
