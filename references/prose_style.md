# Prose Style Rules (Reference)

> Companion to SKILL.md rule #7 (read-aloud test).
>
> When writing deep notes, the prose itself matters as much as the structure. Bad rhythm makes a technically-correct note unreadable. These rules apply to **all reader-facing prose** in the deep note — section bodies, panel-by-panel readings, callouts, summaries. They do NOT apply to claim ledger entries, hyperparameter tables, or code blocks (those are spec-like by design).
>
> Rules are language-aware: §1 covers issues universal to technical prose; §2 is Chinese-specific; §3 is English-specific. Apply whichever matches `--lang`.

---

## 1. Universal rules (any language)

### 1.1 One claim per sentence, with subject + verb + object

**Bad**: telegraphic comma-spliced fragments where 3 things share one sentence with no verb.

> §1.6 讲过, 低层 WM 跑 16 步 AR 误差 ≈ 12× 单步.

Three claims smushed: (i) cross-reference (§1.6); (ii) setup ("low-level WM runs 16 steps"); (iii) quantitative result ("error ≈ 12× single-step"). The result claim has no verb.

**Good**: each claim a sentence with a verb.

> Recall the toy demo in §1.6: when the low-level WM rolled out autoregressively for 16 steps, the error **accumulated to about 12× the single-step error**.

### 1.2 Verbalize symbols & math when they're a parameter, not the claim

When a symbol-cluster is being *mentioned* (a parameter value), read it out. When the formula *is* the claim (a derivation step, a loss definition), keep it as math.

**Bad** (symbol passed off as prose):
> HWM 让高层只走 H=2-5 步, 低层只走 h=2-5 步.

**Good** (verbalized, symbol in parens as supplement):
> HWM plans only two to five steps at the high level (`H = 2–5`) and two to five steps at the low level (`h = 2–5`).

**Still good** (math is the claim itself):
> Loss: $\mathcal L_{tf} = \frac{1}{T}\sum_k \|\hat z_{k+1} - z_{k+1}\|_1$.

### 1.3 Complete every comparison

A bare "shorter than X" / "more than Y" with no "than X's *what*" is a dangling comparison. Always finish it.

**Bad**: "整体 AR chain 远短于 flat." (shorter than flat's what?)
**Good**: "整体 AR chain 比 flat planner 的 AR chain 短一个数量级."

### 1.4 "(a) + (b)" inside a sentence is a hidden bullet list

Patterns like "by (a) … and (b) …" or "uses (a) … and (b) … simultaneously" are bullet lists pretending to be prose. Pick one:
- (i) **Promote to a real bulleted list** if the items are independent and >2.
- (ii) **Verbalize fully** with proper enumeration words ("first … second …" / "一是 … 二是 …"), no parenthetical letters.

**Bad**: "HWM 用 (a) 搜索分层 + (b) 长 horizon 用高层单步 同时解决"
**Good**: "HWM addresses both problems at once — on the search side, hierarchy collapses one deep loop into two shallow ones; on the error side, the long-horizon segment is handled by single-step prediction in the high-level WM, which sidesteps the long AR chain entirely."

### 1.5 Active voice over Anglo-style passive

In both English and Chinese, prefer active when the subject is clear. Passive ("被 X 打击" / "is hit by") often comes from translating English passive constructions and sounds stiff.

**Bad**: "flat planner 被 (a) 搜索爆炸 + (b) 误差累积同时打击"
**Good**: "在长 horizon 任务里, flat planner 同时被两件事 KO: 一是 …; 二是 …"  (lighter; or rewrite further: "搜索爆炸和误差累积同时把 flat planner 干翻")

### 1.6 Connective tissue between sentences

Adjacent sentences should signal their relationship. Ask: is sentence 2 a consequence / restatement / contrast / specialization of sentence 1? If yes, add an explicit connective (因此 / 这意味着 / 换句话说 / 也就是说 / 这导致 / 但是 / 反过来 / Therefore / In other words / This means / By contrast).

Strings of unmarked sentences read as scattered claims. The reader has to infer the logic.

### 1.7 Soften cross-references

A bare "§X.Y said: …" is a lookup instruction. Turn it into a shared-memory recall: "Recall the toy in §X.Y where …" / "如同 §X.Y 里那个例子 …" / "前面 §X.Y 给过 …".

This treats the reader as a fellow traveler who remembers, not a database doing lookups.

### 1.8 Read-aloud test (the catch-all)

After writing a paragraph, read it aloud (in your head). If it sounds like:
- reading a table cell ("X 32 步 误差 12× 单步") → fail
- bullet-points-with-the-bullets-removed ("A + B + C; therefore X + Y") → fail
- "if I were giving a slow verbal explanation, this is not how the sentence would come out" → fail

…then rewrite. Spoken explanation is the goal; written-down notes is the failure mode.

---

## 2. Chinese-specific (when `--lang chinese`)

### 2.1 中英 code-switching 要有节奏, 不要每个 clause 都切

If every clause has an English noun ("AR chain 远短于 flat planner 的 AR chain"), the reader's brain switches context every 3 characters. Patterns that work:

- **English noun as named entity** (kept English throughout the note for consistency): `HWM`, `CEM`, `MPC`, `Transformer`, `latent action`.
- **Switch boundaries align with phrase boundaries**: "HWM 的 latent action 空间" works; "AR chain 远短于 flat" fails because the predicate boundary cuts the language.
- **Avoid English subject ending the sentence**: "比 flat planner 的 AR chain 短" is OK because the Chinese predicate carries weight; "AR chain 远短于 flat" leaves English dangling and reads like an unfinished sentence in either language.

### 2.2 中文连词比英文密

中文 prose 比英文更依赖"因此 / 也就是说 / 换言之 / 即 / 结果就是 / 反过来 / 但是" 等连接词。英文 prose 经常省略 ("This means …" 可以省成只剩 "."); 中文 prose 不这样 — 省连词的中文读起来碎。Don't directly translate English-rhythm prose into Chinese; add the connectives.

### 2.3 数字一般不用 "× 单步" 这种写法

"≈ 12× 单步" 是英文 spec sheet 写法。中文 prose 写"约为单步精度的 12 倍" / "比单步大约高 12 倍" / "累积到了单步的十几倍"。

数字写中文还是阿拉伯, 跟语境配:
- 精确量化 + 表格上下文: "12 倍" / "12×"
- 行文叙述: "约十二倍" / "十几倍" (中文数字让 prose 更有节奏)

### 2.4 避免"AR 链 / AR chain 远 短于 flat"这种半英半中名词短语

英文复合名词嵌中文里时, 要么整体用英文 ("AR rollout chain"), 要么用中文译名 ("AR 累积链") + 首次出现给括号注释 ("自回归累积链 (AR rollout chain)")。混拼 "AR 链" 没有英文译过来 nor 完整中文术语, 读起来卡。

---

## 3. English-specific (when `--lang english`)

### 3.1 No "demonstrates" / "showcases" / "leverages" / "delves" / "robust" filler

Standard AI-tell vocabulary. Replace with concrete verbs ("uses", "runs", "outperforms by 3pp", "is stable under …"). Same energy as rule #4 in SKILL.md but applied to prose, not claims.

### 3.2 Em-dash discipline

Em-dashes are fine for parenthetical asides, but more than 2 em-dashes in a paragraph signals AI prose. Convert at least half to commas, semicolons, or actual subordinate clauses.

### 3.3 No "It is important to note that" / "Note that" sentence starters

Either the next sentence IS important (in which case "Importantly, …" is barely better, "Note that" much worse), or it isn't (delete the meta-comment).

---

## 4. When to use the polish skills

Optional post-pass tools:
- **`writing-anti-ai`** catches the AI-tell vocabulary issues (rule 3.1) and helps remove AI stylistic fingerprints. Run after the main draft if you suspect AI tells are leaking in. **Not a substitute for rules 1.1–1.8 / 2.1–2.4** — those are about prose mechanics, not AI tells.
- **`nature-polishing`** is for Nature-leaning English. Overkill for deep notes (which aim at tutor-style, not publication-formal), but useful when the deep note's `suggested-survey-paragraph` is going to be lifted into a survey.

These are tools, not crutches. The rules above must be applied during writing, not delegated to a post-pass.

---

## 5. Self-iteration friction (cross-link to SKILL.md Step 6.5)

When running the self-iteration loop (`--iter ≥ 2`) or the final self-check (always), add these friction categories:

- *"This sentence has 3 commas and no verb"* → rule 1.1 violation
- *"I had to mentally translate `H=2-5` to 'two to five' to parse"* → rule 1.2 violation
- *"Shorter than X — wait, X's what?"* → rule 1.3 violation
- *"(a) + (b) inside one sentence reads like a bullet list crammed in"* → rule 1.4 violation
- *"Why is this sentence right after the previous one? What's the link?"* → rule 1.6 violation
- *"This paragraph reads like a table cell, not speech"* → rule 1.8 violation
- *"中英切换太频繁, 我大脑卡了"* → rule 2.1 violation
