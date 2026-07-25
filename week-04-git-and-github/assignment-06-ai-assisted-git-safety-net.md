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

![ss1](./screenshots/W4-SS-A6/W4-A6-SS-1.png).

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

A dedicated feature branch isolates this work from main, allowing you to test and iterate without affecting the primary codebase. It establishes a clear workflow and ensures main stays stable.

---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![ss1](./screenshots/W4-SS-A6/W4-A6-SS-1-1.png).




---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

Using a fake key avoids creating a real security vulnerability. An obviously fake key (AKIAABCDEFGHIJKLMNOP) demonstrates the security pattern without actual risk.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![ss2](./screenshots/W4-SS-A6/W4-A6-SS-2-hooks-pre-commit.png).


---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![ss3](./screenshots/W4-SS-A6/W4-A6-SS-3.png).

---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

.git/hooks/ is local and not tracked by Git, so hooks placed there won't be shared with teammates. By tracking hooks/pre-commit in the repository root and configuring core.hooksPath hooks, every developer who clones the repo automatically gets the same hook. This creates one canonical version that the whole team uses, ensuring consistent safety checks everywhere.

If it only lived in .git/hooks/, each person would have to manually copy it or set it up by hand—defeating the purpose of a shared safety gate.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

PreToolUse blocks Claude Code tool invocations before they run; pre-commit blocks Git commits before they're created. Both are "safety gates" using fixed rules.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![ss4](./screenshots/W4-SS-A6/W4-A6-SS-4.png).

---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

AKIA[0-9A-Z]{16} matched. AWS Access Key IDs always start with "AKIA" + 16 alphanumeric chars—the fake key fits exactly.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No, it wouldn't catch SECRET_KEY=abc123 without AKIA. This shows the core limit: fixed rules only block what they're told to look for. That's why you need both the hook and AI review.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![ss5](./screenshots/W4-SS-A6/W4-A6-SS-5.png).

---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![ss6](./screenshots/W4-SS-A6/W4-A6-SS-6.png).

---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

/pr-ready must only observe, never modify. Bash + Read let it gather facts; no Write prevents it from editing files, committing, or pushing.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

Yes, both flagged the AKIA secret and debug echo. The hook caught them via regex; the skill via semantic understanding. The skill would also catch mixed concerns or unclear intent the regex misses.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![ss7](./screenshots/W4-SS-A6/W4-A6-SS-7.png).

---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![ss8](./screenshots/W4-SS-A6/W4-A6-SS-8.png).
![ss8](./screenshots/W4-SS-A6/W4-A6-SS-8-1.png).

---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

Removed the hardcoded AWS key line (AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP) and the debug echo statement (echo "DEBUG: token is $AWS_ACCESS_KEY_ID")
---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![ss9](./screenshots/W4-SS-A6/W4-A6-SS-9.png).

---

#### PR Link

https://github.com/santosronnie26-sr/devops-micro-internship-pravinmishra/pull/1

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**

Yes. Clarify vague phrasing, add missing context, remove redundancy, ensure title/description align. Editing ensures the PR accurately represents your intent.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The AI's draft might have inaccuracies, omissions, or mischaracterizations that confuse reviewers. A bad description wastes review time and you become responsible for every word.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

The shared upstream is for cohort-wide submissions. Your fork keeps learning work separate and experimental.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

Gather: git status and git diff --cached (hook + skill inspect changes)

---

**2. Which step(s) represent Analyze?**

Analyze: Hook checks patterns; skill flags semantic risks and drafts PR description

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

Human Act: You fix issues, run git commit, git push, open PR. Only humans can decide and act on the shared codebase.

---

**4. Which step is Verify?**

Verify: Re-run hook (passes clean) and skill (clean report). Merge and post-PR testing.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

Both needed: Hook catches known risks (secrets, size) reliably; AI notices judgment calls (mixed concerns, unclear intent). Together: mechanical automation + reasoning.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

Add your LinkedIn post URL here...

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

-Pre-commit hooks are shareable safety gates: Tracking hooks/pre-commit in the repo and configuring core.hooksPath ensures every developer on the team runs the same fixed-rule checks before a commit is created—no manual setup, no excuses.

-Fixed rules + AI advice are complementary, not redundant: The hook catches known secret patterns reliably (AKIA regex, file size). The /pr-ready skill catches judgment calls (mixed concerns, missing context, unclear intent). Together they cover the mechanical and the reasoning.

-Read-only AI is safer and clearer: A Claude Code skill with Bash, Read, Grep but no Write can advise but never damage code. It drafts, you decide; it gathers facts, you act. This keeps the human in control of every change to the shared codebase.

-Always read and edit AI drafts before using them: Copy-pasting an AI-generated PR description without reviewing it risks inaccuracies, missing context, and wasted review time. You become responsible for every word, so treat the draft as a starting point, not a finished product.

-The Agentic Loop applies to Git workflows too: Gather (git status + diff) → Analyze (hook + skill check) → Human Act (you fix, commit, push, open PR) → Verify (re-run checks). A human must always own the actions that change the shared codebase.

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

https://github.com/santosronnie26-sr/devops-micro-internship-pravinmishra

---

# Completion Checklist

- [✓] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [✓] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [✓] `core.hooksPath` configured to point at `hooks/`
- [✓] Pre-commit hook shown blocking the risky commit
- [✓] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [✓] `/pr-ready` run against the risky diff and shown flagging issues
- [✓] Risky file fixed; `git commit` succeeds cleanly
- [✓] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [✓] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [✓] Agentic Loop mapping (Task 7) completed in your own words
- [] LinkedIn post published and URL submitted
- [✓] All required screenshots added
- [✓] GitHub repository URL provided

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
