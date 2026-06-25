# ats-resume

Claude skill that rewrites your resume for a specific job posting. Built on actual hiring agent source code. The scoring prompts, evaluation criteria, and deduction logic are all public.

It does not invent anything. I am simply making this available to automate the workflow for myself and my friends.

---

## Why

- Resumes are rejected before a human sees them. Automated agents parse (extract words from your resume) and score first.
- These agents are built to be cheap. They extract keywords from your resume rather than read it, because fewer tokens means fewer dollars. You are being filtered by a cost-cutting measure.
- A Google Docs export to a Microsoft Word file produces messy XML. Fields go missing. Sections get misread. You get an instant denied email from the employer.
- This skill rewrites your resume to score against the actual criteria, uses that feedback to output a native .docx so the parser gets clean input.

---

## How it works

- Drop a resume and a job posting URL.
- The skill fetches the posting, extracts what the screener is scoring for.
- Claude rewrites the resume to address: keywords, quantification, location, skills gaps, domain language.
- To make it more effective: on first use, fill out a context file with rough numbers, achievements, and tools you know but never listed. After that, every application resume and cover letter you submit is automatic.

---

## Install

- Download `ats-resume.skill` and go to Claude Settings > Skills > Upload.
- Or open Cowork on Claude and paste this GitHub link.

**Claude.ai (free or Pro, no desktop)**

- Open a new conversation.
- Go to the repo and open `SKILL.md`.
- Copy the entire file.
- Paste it at the start of your Claude conversation before uploading your resume.
- On first use, Claude will run a short onboarding and generate a `context.md` profile. Copy and save that yourself (Notes, Google Drive, anywhere).
- On every future use, paste your saved `context.md` into the conversation before dropping a job link.

---

## Usage

Drop in a job link and your resume. It exports a resume. If you want a cover letter too, it will ask you first.

---

## Based on

[interviewstreet/hiring-agent](https://github.com/interviewstreet/hiring-agent) (HackerRank, MIT License)

- Open source resume scoring that extracts structured data from a resume, scores it across categories, and produces an explainable evaluation with evidence. The scoring prompts and evaluation criteria are public.
- This skill reads those criteria and uses them to simulate what the agent would score before rewriting. The rewrite targets the specific gaps and deductions the agent would fire, not general best practices.
- The original repo was built for software engineering candidates. The scoring logic here is adapted for non-technical roles.

---

## License

MIT
