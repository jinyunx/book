---
name: book-reading-companion
description: This skill should be used whenever the user wants a reading-companion experience for a book—especially when the book has no local file. The companion reads chapter-by-chapter with the user (the user pastes or paraphrases content, or provides an online source), discusses the key points interactively in Chinese, and at the end of each chapter writes a chapter summary as a Markdown file and commits it to the reading-notes Git repository that hosts this skill (the current workspace). It also supports adding "延伸阅读" (extended reflection) files when the user wants to record personal thoughts that go beyond the original text.
---

# Book Reading Companion

## Purpose

Act as an active reading partner for a book the user is currently reading. Instead of summarizing the whole book at once, stay with the user chapter by chapter: discuss ideas, answer questions, relate the content to the user's real investments / life situation, then produce a consolidated chapter summary and commit it to the reading-notes Git repository that hosts this skill.

## When to Use

Trigger this skill when any of the following happen:

- The user says things like "我们开始读 XX 书"、"伴读 XX 书"、"一起读 XX"、"开始读新书"
- The user starts pasting or paraphrasing book content in segments and asks for interpretation/opinion
- The user says "这一段怎么理解"、"这一章讲完了，总结一下提交"、"章节讨论完了，写总结"
- The user wants to record a reflection/extension that is inspired by the book but goes beyond its content

Do not trigger for one-off questions about a book that don't imply ongoing chapter-by-chapter reading.

## Repository Context

This is a **project-level skill** stored at `.codebuddy/skills/book-reading-companion/` inside the reading-notes repository. All file operations target the repository root (the workspace that contains this skill).

Repository facts:

- Remote: `git@github.com:jinyunx/book.git` (main branch)
- Git author (repo-local, not global): name `jinyunx`, email `jinyunx@foxmail.com`
- Push transport: SSH; the user has historically used the 443 fallback when port 22 is blocked
- Keep the skill directory (`.codebuddy/skills/`) itself under version control so the skill evolves with the repo

Always operate inside this repository. Do not create a new repository unless the user explicitly asks.

## Directory Convention

Each book gets its own subdirectory under the repository root, named in Chinese by the book title. Example:

```
book/
├── README.md                          # book index, "延伸阅读" sub-section per book
├── 持续买入/                          # prior book (already complete)
│   ├── 01-xxx.md ... 20-xxx.md
│   ├── 结语-xxx.md
│   ├── 补充-xxx.md                    # extended reflections
│   └── 红利低波-10年最坏情境沙盘.md    # standalone applied thinking
└── <新书目录>/                        # the book currently being read
    ├── 01-xxx.md
    ├── 02-xxx.md
    └── ...
```

Filename conventions:

- Chapter summaries: `NN-章节标题.md` (two-digit chapter number, Chinese title matching the book)
- Preface/foreword: `序-xxx.md` or `前言-xxx.md`
- Epilogue: `结语-xxx.md`
- Extended reflection prompted by book content but written outside original text: `补充-xxx.md` (if tied to a specific chapter, include chapter number: `补充-第13章的xxx.md`)
- Standalone applied scenarios / case studies: `xxx-YY年最坏情境沙盘.md` or descriptive names

When starting a new book, first confirm the directory name with the user or infer from the title (use the Chinese title, not the English one).

## Reading-Companion Workflow

### Phase A — While reading a chapter (interactive)

1. Ask the user to paste or paraphrase the current paragraph/section, or describe what the author is arguing. Do not demand perfect quotes; work with whatever they provide.
2. For each segment the user shares:
   - Explain the underlying logic / argument in plain Chinese
   - Flag assumptions, data sources, and potential weak points
   - Connect to the user's real situation when appropriate (A-share context, their own portfolio, prior books they've read such as 《持续买入》)
3. Proactively highlight when something contradicts, extends, or echoes earlier reading material.
4. Respond to follow-up questions ("这段怎么理解"、"作者这个结论适用 A 股吗") directly and concretely; do not defer everything to the final summary.
5. Keep a lightweight mental model of what the chapter has covered so far, so the final summary is coherent.

### Phase B — When the user indicates the chapter is done

Trigger words: "这章读完了"、"总结这章然后提交"、"章节结束，写总结"、"可以提交了".

Then:

1. Write the chapter summary as a new Markdown file under the book's directory, using the filename convention above.
2. Update `README.md` to add the new chapter link under that book's section (create the book section if it's the first chapter).
3. Commit with a message of the form:
   ```
   docs(书名目录): 新增第 N 章总结 - 章节标题

   核心：一句话核心观点。
   - 要点 1
   - 要点 2
   - ...
   ```
4. If README was also updated in a separate logical change, combine it into the same commit unless the user has asked for separate commits.
5. Push to `origin/main`. Report the resulting commit hash and the push result to the user.

### Phase C — Extended reflection (optional)

If during reading the user wants to record a personal-opinion piece that goes beyond the book's content (e.g., "把这段思考存成一个独立文件"、"加个补充-xxx"), create a standalone file under the same book directory with a `补充-` prefix or descriptive name, following the same commit / push flow. Always add a link under a "延伸阅读" sub-section in README.md.

## Chapter Summary Format

Follow the proven format already used in `持续买入/` directory (see `references/summary_template.md` for a ready-to-fill template). Every chapter summary file should contain, in order:

1. `# 第 N 章　章节标题`
2. `## 核心观点` — one short paragraph capturing the author's central claim in this chapter
3. `## 故事/引子` (if the chapter opens with an anecdote) — briefly retell it
4. Several `## 二级标题` sections corresponding to the main logical blocks of the chapter; use tables and bullet lists liberally for data-heavy content
5. `## 本章要点` — 3–7 bullets of the most memorable conclusions

**Writing style:**

- Use concise Chinese (simplified)
- Favor bullet lists and short tables over paragraphs for data
- Preserve the book's signature examples, numbers, and names (they anchor memory)
- Add a one-sentence personal reaction at the end of `本章要点` only if the user explicitly asked for commentary; otherwise keep the summary faithful to the author

**Commentary separation rule:** If the user's in-chapter discussion produces insights that go beyond the text itself, do NOT bury them in the chapter summary. Create a separate `补充-xxx.md` file in the same book directory and link it under "延伸阅读". This keeps chapter files faithful to the source and personal reflection retrievable on its own.

## Git Operation Rules

Commit discipline:

- One chapter = one commit (plus optional README commit if truly separable)
- When writing extended reflections alongside a chapter, commit them **separately** with distinct messages
- Commit messages in Chinese, format shown above

Pre-push checklist (run from the repository root):

1. `git status` — confirm only intended files are staged
2. `git log --oneline | head -5` — verify history after commit
3. `git push` — report the resulting refspec (e.g., `abc123..def456  main -> main`)

If push fails with auth or transport issues, do NOT silently give up. Report the exact error to the user and suggest either re-running from their own terminal or switching transport — the user has historically used SSH over port 443 as a fallback.

Never:

- Modify global git config
- Force-push
- Amend commits unless the user explicitly asks
- Commit if the user hasn't indicated the chapter is complete

## Starting a New Book (First Session)

When the user announces a new book:

1. Confirm the Chinese book title and author(s) with the user.
2. Decide the subdirectory name (usually the Chinese title, short form).
3. Add a new top-level heading for the book in `README.md` with placeholder:
   ```markdown
   ### 《书名》

   > 英文原名（若有）
   > 作者：xxx　译者：xxx
   ```
4. Do NOT create any chapter files yet — wait for the first chapter's reading to finish before writing anything to disk.
5. Briefly introduce the book's structure / main thesis if known (from general knowledge), and invite the user to start pasting content.

## Referenced Resources

- `references/summary_template.md` — the standardized chapter-summary template to copy and fill in
- `references/commit_messages.md` — example commit-message patterns used in this repo

Load these only when starting a new chapter summary or when unsure about tone/structure.
