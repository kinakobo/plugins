# そのまま使えるプロンプト（Prompts worth copying）

パス、スキル、完了の確認は、実際のものに置き換える。くだけた言い回しでも通じる。

## 理解する（Understand）

- `/poteto-mode read <thread>. restate the underlying issue in your own words, in plain english.`
- `/in-your-own-words <long or messy ask>`
- `/sharpen <the prompt you're about to send>`
- `/poteto-mode investigate why <symptom>. give me what we know, what data you used, and your best hypotheses. don't change any code yet.`
- `use /how to understand <subsystem>. then use /why to find out why it broke recently.`
- `/recall my work on <topic> from last week, then read <issue>.`
- `/teach me why you implemented it this way and not <other way>. what did you trade off?`
- `/poteto-mode take over this branch. read the decision log, find what's done, and continue. don't redo finished work.`

## 作る（Build）

- バグ：`/poteto-mode <symptom>. repro first, then fix and verify.`
- アプリのバグ：`/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.`
- 安くテストできるバグ：`/poteto-mode repro <bug> first. if there's a cheap test path, /tdd it. then fix and rerun.`
- 機能：`/poteto-mode add <behavior>. <current output> stays byte-identical. verify both.`
- リファクタリング：`/poteto-mode move <code> into one module, zero behavior change. record the current output first and prove it's unchanged after.`
- 性能：`/poteto-mode <operation> takes <time> on <fixture>. trace it, fix the measured cause, show me before and after.`

## 設計と計画（Design and plan）

- `/poteto-mode prototype a few options for <feature>. take screenshots or videos for me to compare.`
- `/poteto-mode we need <feature>. /architect it first, and answer open questions with prototypes. let me review before proceeding.`
- `/poteto-mode write a tutorial for how i would use <new package> first. then /teach me why it beats the current one.`
- `ask /arena for a second opinion on this thread and our approach.`
- `/poteto-mode turn this design into a plan. small verifiable PRs, each with its own verification steps.`
- `/poteto-mode plan the migration of <library> to <target>. small verifiable PRs. the result must match the original exactly, bugs included.`

## レビューとリリース（Review and ship）

- `/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's a real bug or regression.` Dropped の一覧も読む。
- `/create-formal-verification-skill prove <property> holds in <module>, even when <steps> interleave or retry.` そのコードでマージやリファクタリングがあった後は `/maintain-formal-verification-skill`。
- `/swarm check every package under <dir> against its check script. one worker per package. one report.`
- `/poteto-mode open the pr. small ordered commits, evidence in the description.`
- `/poteto-mode babysit this pr. get it green.` 状況を知りたいだけなら `/poteto-mode check on pr <number>. anything outstanding?`
- `/poteto-mode land the stack.`

## 離席と復帰（Away and back）

- `/poteto-mode im going to bed. <goal> in a fresh worktree off <base>. done means <checks>. keep a decision log. don't ask me before committing. /loop until done. if you're truly stuck after a few hours, stop and write up why.`
- `/show-me-your-work catch me up on what you did last night.` まず Attention の節を読む。
- `/ketchup` は、前回のメッセージ以降のこのチャットの内容を伝え、ユーザーの判断を待っている事項をすべて挙げる。
- `/poteto-mode full autopilot on this queue. each item is independent.`
- `/poteto-mode autopilot these changes, but don't merge. i'll land them.`
- `/reflect capture what we learned so the next run doesn't repeat it.` 承認するのは、今後の判断を変える編集だけにする。
- `/bro` は、直前の返答を平易な言葉で言い直す。
