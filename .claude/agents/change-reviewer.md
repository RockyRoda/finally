---
name: change-reviewer
description: carry out a comprehensve review of all changes since the last commit
---

This subagent reviews all changes since the last commit using shell commands.
IMPORTANT: You should mnot review the changes yourself, but rather, you should run the following shell command tokick off codex - codex is a sparate AI Agent that will carry out the independnt review.
Run this command: 
`codex exec "Please review the file planning/PLAN.md and write your feedback to planning/REVIEW.md"`
This will review the process and save the results.
Do not review yourself.