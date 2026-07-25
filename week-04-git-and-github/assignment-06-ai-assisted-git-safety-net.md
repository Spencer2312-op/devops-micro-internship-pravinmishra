# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch
[Assignment 06Screenshot Task0](screenshots/week-04-git-and-github-06-task0-1.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

I keep separate branches for a few reasons. First, it protects main from half finished or broken work, so if something goes wrong in this assignment it doesn't affect the stable version of the repo. Second, it keeps the changes for this specific task isolated, which makes the pull request cleaner since it only shows what actually belongs to this assignment. And third, since I'm regularly submitting different pieces of work for this course, having a dedicated branch per assignment just keeps things organized and easier to track.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

[Assignment 06Screenshot Task1](screenshots/week-04-git-and-github-06-task1-1.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

I used a fake key instead of a real one because the point of this exercise is to test that the pre commit hook and the AI skill can actually detect a secret pattern, not to expose a real credential. Using a real key would be a genuine security risk if it ever got pushed or leaked, even accidentally. The fake key still matches the same format real AWS keys use, so it's enough to trigger the detection without any real danger attached to it.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

[Assignment 06Screenshot Task2](screenshots/week-04-git-and-github-06-task2-2.png)

---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

[Assignment 06Screenshot Task2](screenshots/week-04-git-and-github-06-task2-3.png)

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

.git/hooks/ never gets committed or pushed, it's local to each person's machine, so a hook living only there would only protect me and nobody else on the project. By keeping the hook in a tracked hooks/ folder and pointing core.hooksPath at it, the script becomes part of the repo itself, so anyone who clones it and sets the same config gets the same protection. It turns a personal safety check into a shared one.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

PreToolUse intercepts a Claude Code action before it runs, like blocking a Write or Bash command before it touches the file system. The Git pre-commit hook intercepts a commit before it's created, checking the staged changes and stopping the commit if it finds a problem.

What they have in common is the timing: both act before the risky action is finalized, not after. Neither one asks permission first or reviews the result afterward, they just refuse to let the action complete if it fails the check. Both are also rule based rather than judgment based, they run the same fixed check every time regardless of context.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

[Assignment 06Screenshot Task3](screenshots/week-04-git-and-github-06-task3-4.png)

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**
The line that caught it was the grep pattern inside the secret detection check:
grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'
My key was AKIAIABCDEFGHIJKLMNOP, which starts with AKIA followed by 16 uppercase letters and numbers, so it matched the first part of that pattern exactly.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No, it wouldn't have caught it. The hook only looks for specific known formats like the AKIA prefix or a private key header. If I had stored a secret in a variable like myPassword = "hunter2" or apiToken = "xyz123", the hook has no way of knowing that's sensitive because it doesn't understand meaning, it just matches patterns. That's the core limitation of a fixed rule, it can only catch what it was explicitly told to look for, not anything that looks risky based on context or naming.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

[Assignment 06Screenshot Task4](screenshots/week-04-git-and-github-06-task4-5.png)

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

[Assignment 06Screenshot Task4](screenshots/week-04-git-and-github-06-task4-6.png)

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

It has Bash and Read because it needs to actually look at the repository, run things like git status and git diff to see what's staged, without ever being able to change anything. Leaving out Write means it physically can't edit files, create commits, or modify the repo, so even if it wanted to "fix" something, it's not capable of it. The restriction forces it to stay in an advisory role.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

They both caught the debug echo statement and the credential-like content, so there was overlap there. But /pr-ready went further than the hook, it questioned the actual purpose of the script, pointed out that it looked incomplete, and suggested what might be missing, like a README or real implementation. The hook can't do any of that, it just pattern matches and blocks. So the hook caught the same surface-level secret issue, but /pr-ready added judgment and context the hook has no way of producing.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

[Assignment 06Screenshot Task5](screenshots/week-04-git-and-github-06-task5-7.png)

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

[Assignment 06Screenshot Task5](screenshots/week-04-git-and-github-06-task5-8.png)

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**
I removed the hardcoded AWS style key and the debug echo line that printed it. The script went from setting a fake credential and echoing it to the terminal, down to just a comment explaining it's a placeholder and a simple echo saying the notification script is running. Once there was no more AKIA pattern anywhere in the staged diff, the hook had nothing to flag and let the commit through.
---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

[Assignment 06Screenshot Task6](screenshots/week-04-git-and-github-06-task6-9.png)

---

#### PR Link
(https://github.com/Spencer2312-op/devops-micro-internship-interviews/pull/1)

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

The description was mostly used as-is since it was accurate and covered the right things. The main adjustment was making sure the Testing section reflected what actually happened during the assignment rather than generic placeholder text, so it read like a real record of what was verified rather than something the AI assumed would be done.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The AI draft could contain inaccuracies, reference files or steps that didn't actually happen, or use overly generic language that doesn't reflect what was actually built. Blindly pasting it would mean submitting a description that might misrepresent the changes, which defeats the whole point of a PR description and could confuse a real reviewer.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This assignment's files are practice artifacts, not contributions meant for the shared class repo. Opening the PR against upstream would pollute Pravin's repository with everyone's assignment work, which isn't the intention. The PR targets your own fork because that's where your work lives and where you're in control of what gets merged.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Gather is represented by the pre-commit hook and /pr-ready both running git diff --cached and git status to collect information about what's staged. Neither is making decisions yet, they're just pulling together the raw data needed for the next step.

---

**2. Which step(s) represent Analyze?**

Analyze is represented by the pre-commit hook pattern-matching the staged diff against known secret formats and file size limits, and /pr-ready reading through the same diff and applying judgment to flag risks, assess context, and draft a PR description. Same data, two different kinds of analysis.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act is git commit, git push, and opening the PR — all three have to be done by a human because they change real, external state. A commit becomes part of the permanent history, a push changes what's on GitHub, and opening a PR notifies other people and starts a review process. These are irreversible or hard-to-reverse actions that require a human to take responsibility for them. Claude's role was to inform that decision, not make it.

---

**4. Which step is Verify?**

Verify is the second run of both the pre-commit hook and /pr-ready after fixing the issues. The hook passing without a BLOCKED message confirmed the secret and debug line were gone, and /pr-ready returning a clean report with no findings confirmed the changes were safe to proceed with.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

You need both because they cover different things. The hook is fast, deterministic, and always on — it will never miss an AKIA pattern because it never gets tired or skips a file. But it's blind to anything it wasn't programmed to catch. /pr-ready brings judgment and context — it can flag that a script looks incomplete, that a change lacks documentation, or that something feels off even if no pattern matched. One gives you reliability, the other gives you reasoning. Neither alone is enough.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

(https://www.linkedin.com/posts/spencerenchill_dmibypravinmishra-agenticai-devops-share-7486306499208675328-6X6O/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAGRQq6IBvhyFikdnuZmUnAgoctctbC0h3m4)

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

-I learned that Git pre-commit hooks and AI-assisted review tools like /pr-ready serve different but complementary roles — the hook catches known patterns with zero judgment, while the AI brings context and reasoning the hook can't replicate.
-I learned that restricting what an AI skill can do (removing Write access) is just as important as defining what it should do — keeping /pr-ready read-only meant it could never accidentally modify, commit, or push anything, which made it safe to run at any point in the workflow.
-I learned that the Agentic Loop applies just as naturally to Git workflows as it does to system triage — Gather, Analyze, Human Act, and Verify is a pattern that shows up wherever you need automated tools and human judgment working together without one overstepping the other.

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

https://github.com/Spencer2312-op/devops-micro-internship-pravinmishra.git
---

# Completion Checklist

- [✅ ] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [ ✅] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [ ✅] `core.hooksPath` configured to point at `hooks/`
- [ ✅] Pre-commit hook shown blocking the risky commit
- [✅ ] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [✅ ] `/pr-ready` run against the risky diff and shown flagging issues
- [✅ ] Risky file fixed; `git commit` succeeds cleanly
- [✅ ] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [✅ ] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [✅ ] Agentic Loop mapping (Task 7) completed in your own words
- [ ✅] LinkedIn post published and URL submitted
- [ ✅] All required screenshots added
- [✅ ] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
