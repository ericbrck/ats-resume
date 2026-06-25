# ats-resume

A skill that rewrites resumes and cover letters to clear automated hiring agent screening.

Built on interviewstreet/hiring-agent (HackerRank, MIT) and blader/humanizer (MIT).

---

## Trigger

Activate when the user drops a job posting URL, or says anything like:
- "rewrite my resume for this job"
- "help me apply for this"
- "can you tailor my resume"

---

## Two Modes

**First use:** Run onboarding. Before anything else, determine which environment the user is in:

- **Cowork:** Check if context.md exists in the user's connected folder. If it does, skip onboarding and go straight to the application flow. If it does not, run onboarding and save context.md to their folder.
- **Claude.ai free:** There is no persistent file storage. Check if the user has pasted context.md content into the conversation. If they have, use it. If they have not, run onboarding and at the end display the completed context.md and tell them to save it themselves for future sessions.

If it is unclear which environment the user is in, ask:

> "Are you using Claude Cowork, or Claude.ai free? This determines how I save your profile."

**Every use after that:**
- Cowork: load context.md from the connected folder automatically.
- Claude.ai free: prompt the user to paste their context.md at the start of each session before running the application flow.

---

## Onboarding Flow

Run this exactly once. The goal is to build context.md, a file Claude will reference on every future application.

Tell the user:

> "Before we start, I need to set up your profile. This takes about 5 minutes and you will never have to do it again."

Then collect the following, one question at a time using the multiple choice UI where possible:

### Step 1: Resume upload

> "Upload your current resume. Any format works: .docx, .pdf, or paste the text directly."

Store as the baseline resume. Extract and save to context.md:
- Full name
- Contact info
- All work history (company, title, dates, bullets)
- Education
- Skills and tools listed

### Step 2: Relocation

> "Where are you located, and are you open to relocating?"

Options:
- I am local to where I am applying
- I am relocating. Where and when? (free text)
- I am open to remote only
- I am open to hybrid or remote

Save answer to context.md under LOCATION.

### Step 3: Target roles

> "What job titles are you going for?"

Free text. Examples: Executive Assistant, Operations Coordinator, Office Manager.

Save to context.md under TARGET ROLES.

### Step 4: Openness to alternatives

> "If your resume is a weak match for a job you apply to, do you want suggestions for roles that might be a better fit based on your background?"

Options:
- Yes, always suggest alternatives
- Only if the match is weak
- No, just focus on the job I gave you

Save to context.md under ALTERNATIVES PREFERENCE.

### Step 5: Brain dump

> "This is the most important step. Think back across every job you have had. I want rough numbers, achievements, and anything you know you did but never wrote down. Do not worry about how it sounds. Examples:
>
> - Managed about 12 people
> - Reduced scheduling conflicts by roughly a third
> - Ran a budget of around $40K
> - Know PowerPoint but never listed it
> - Onboarded around 20 new hires in one year
>
> Type everything you can remember. More is better."

Free text, no limit. Save everything verbatim to context.md under BRAIN DUMP.

### Step 6: Confirm and save

Assemble context.md with this structure:

```
# context.md

## NAME
[full name]

## CONTACT
[email, phone, LinkedIn URL if present]

## LOCATION
[relocation status and details]

## TARGET ROLES
[job titles]

## ALTERNATIVES PREFERENCE
[user's choice]

## BASELINE RESUME
[full extracted work history, education, skills]

## BRAIN DUMP
[everything the user typed in step 5]
```

**Cowork users:** Write context.md to their connected folder.

Tell the user:
> "Done. Your profile is saved. From now on, just drop a job link and I will handle the rest."

**Claude.ai free users:** Do not attempt to save a file. Instead, display the completed context.md in full inside a code block and tell the user:

> "Done. Copy everything in the box above and save it somewhere, a notes app, Google Drive, anywhere. At the start of any future session where you want to use this skill, paste it into the chat before dropping your job link. That is how I will remember you."

---

## Application Flow

Run this on every application after onboarding is complete.

### Step 1: Intake

User drops a job posting URL.

Fetch the URL. If the fetch fails or returns a blank page, say:

> "I could not load that page. Paste the job description as text, or take a screenshot and drag it here."

Extract from the job posting:
- Job title (exact string as written)
- Company name
- Location and on-site/remote/hybrid requirement
- Required qualifications (list each one)
- Preferred qualifications (list each one)
- Every tool, software, or platform mentioned
- Every responsibility listed
- Any language or phrasing repeated more than once (these are priority keywords)

### Step 2: Supplemental input

Ask:

> "Is there anything else you want me to consider for this job outside of the listing you provided here? Additional resumes, old files you have, or anything you think is relevant. If you think it adds context, you can drag and drop a file here."

If the user provides additional files, extract and fold their content into the working context for this application. Do not overwrite context.md.

### Step 3: Cover letter

Ask:

> "Do you need a cover letter as well?"

Note the answer. Do not produce the cover letter yet.

### Step 4: Score the original resume

Load context.md from the connected folder (Cowork) or from what the user pasted into the conversation (Claude.ai free). If neither is available, stop and say:

> "I need your profile before I can continue. If this is your first time, I will walk you through setup. If you have used this before on Claude.ai free, paste your saved context.md now."

Then run onboarding if they are new, or wait for the paste if they are returning.

Once context is loaded, using the baseline resume and the job posting data extracted in Step 1, run the scoring simulation below.

Produce Score A: original resume vs. this job.

### Step 5: Rewrite the resume

Using the scoring gaps identified in Step 4, rewrite the resume. Rules are in the Rewrite Rules section below.

### Step 6: Score the rewritten resume

Run the same scoring simulation against the rewritten resume.

Produce Score B: rewritten resume vs. this job.

### Step 7: Output

Deliver in this order:

**1. What the employer's screening agent would see**

Show Score A and Score B side by side as a table:

| Category | Original | Rewritten | Max |
|---|---|---|---|
| Keyword match | x | x | 25 |
| Quantification | x | x | 20 |
| Location alignment | x | x | 10 |
| Skills coverage | x | x | 20 |
| Domain language | x | x | 15 |
| Formatting / parseability | x | x | 10 |
| **Total** | **x** | **x** | **100** |

**2. Screening assessment**

One short paragraph. State plainly whether the rewritten resume would likely clear automated screening for this role. If yes, name the remaining human-side risks. If no, name what is still holding it back.

Example:
> "The rewritten resume would likely clear automated screening for this role. The remaining risk is domain mismatch: your background is in healthcare administration and TSMC is a semiconductor manufacturer. A human reviewer may flag that gap."

**3. resume.docx**

The rewritten resume as a native .docx file. Output specs are in the Output Format section below.

**4. cover letter.docx** (if requested)

The cover letter as a native .docx file. Cover letter rules are below.

**5. Alternative role suggestions** (only if score is weak or user preference is set to always)

If the rewritten resume scores below 55/100, or if the user's ALTERNATIVES PREFERENCE is set to always, add:

> "Based on your background, you may also be a stronger match for: [list 2-3 specific job titles with a one-line reason each]."

---

## Scoring Simulation

Adapted from interviewstreet/hiring-agent scoring criteria. Original was built for software engineering roles. This version is adapted for any role.

Score the resume against the job posting across these six categories. Every category must be scored. No category can be skipped.

### 1. Keyword match (0-25 points)

Extract every required skill, tool, qualification, and responsibility from the job posting.

Check how many appear in the resume, either as exact matches or clear equivalents.

- 20-25: Nearly all required keywords present
- 13-19: Most required keywords present, minor gaps
- 6-12: Significant keyword gaps against required qualifications
- 0-5: Major gaps, multiple required items completely absent

Deductions:
- -3 for each required tool or skill listed in the JD that is completely absent from the resume
- -5 if the job title or a direct equivalent never appears anywhere in the resume

### 2. Quantification (0-20 points)

Count how many bullet points include a specific number, percentage, dollar amount, team size, or time saving.

- 16-20: Most bullets have numbers. Clear, specific evidence throughout.
- 10-15: Some bullets have numbers. Inconsistent across roles.
- 4-9: Very few numbers. Mostly task descriptions with no outcomes.
- 0-3: No numbers anywhere in the resume.

Deductions:
- -2 for each role with zero quantified bullets
- -5 if the entire resume has no numbers at all

### 3. Location alignment (0-10 points)

Compare the candidate's listed location against the job's location requirement.

- 10: Location matches or remote role
- 7: Candidate lists a relocation statement with target city and timeline
- 3: Candidate is out of state with no relocation statement
- 0: Candidate location directly conflicts with an explicit on-site requirement and no statement is present

### 4. Skills coverage (0-20 points)

Check every tool, platform, and software mentioned in the job posting against the resume's skills section and work history.

- 16-20: All or nearly all tools present
- 10-15: Most tools present, one or two missing
- 4-9: Several tools missing
- 0-3: Most tools missing or skills section absent

Deductions:
- -3 for each tool listed as required in the JD that does not appear anywhere in the resume

### 5. Domain language (0-15 points)

Assess whether the resume uses language that signals familiarity with the industry or environment of the role.

- 12-15: Resume language maps closely to the JD's industry context
- 7-11: Partial match. Some relevant language, some mismatch.
- 3-6: Noticeable domain mismatch. Different industry background with no bridging language.
- 0-2: Resume language is entirely from a different domain with no connection to the role

### 6. Formatting and parseability (0-10 points)

Assess whether the resume structure would parse cleanly through an automated system.

- 9-10: Clean single-column layout, standard section headings, no tables or text boxes
- 6-8: Mostly clean with minor formatting risks
- 3-5: Contains elements that may cause parsing issues (columns, tables, graphics, headers/footers)
- 0-2: Heavily formatted template that is likely to produce malformed XML output

---

## Rewrite Rules

Rewrite the resume to improve the score in every category where points were lost. Follow these rules exactly.

**Never invent facts.** Every bullet, number, and claim must come from either the baseline resume or the BRAIN DUMP in context.md. If a number is an estimate from the brain dump, include it. Do not soften it or remove it.

**Keyword mirroring.** For every required keyword, tool, or skill that is present in context.md but absent from the resume, add it. Use the exact phrasing from the job posting where honest to do so.

**Quantify every bullet where possible.** Pull numbers from the BRAIN DUMP. If a bullet has no number available, rewrite it to include scope instead: team size, number of locations, frequency, or scale. A bullet with scope scores higher than a bullet with neither scope nor number.

**Add relocation statement if needed.** If the candidate's location does not match the job and their LOCATION in context.md includes a relocation plan, add this line directly under their name and contact info:

> Relocating to [city], [month year] | Available for on-site roles

**Mirror domain language.** Where the candidate's experience maps to the JD's responsibilities, rewrite the bullet to use the JD's phrasing. Do not change the meaning. Change the language.

**Job title clarification.** If the candidate's past titles do not match the target role but the work was equivalent, add a parenthetical after the title that reflects the actual function performed. Match the parenthetical to the role being applied for, not a generic label.

Examples:
> Scheduling and Operations Coordinator (EA-level support to executive leadership)
> Marketing Associate (full project and client account management)
> Team Lead (people management and cross-functional coordination)

**Clean formatting.** Output must be single-column. No tables in the body. No text boxes. No graphics. Standard section headings: Summary, Experience, Education, Skills. Dates right-aligned or inline. Contact info in plain text, not a header/footer.

---

## Cover Letter Rules

Produce only if the user said yes in Step 3.

**Structure:**

1. Opening line: state the exact job title and company name. One sentence.
2. Body paragraph 1: top 2-3 reasons the candidate is a match, drawn directly from the scoring gaps the rewrite addressed. Use specific language from the JD.
3. Body paragraph 2: if there is a location mismatch, address it here directly and confidently. If no mismatch, use this paragraph for one additional relevant qualification.
4. Closing: one sentence, direct, no desperation.

**Length:** Under 350 words.

**Keyword pass:** Every required skill and tool from the JD that appears in the resume must also appear naturally in the cover letter at least once.

**Humanizer pass:** After writing the cover letter, review it against these patterns and remove every instance:

- Em dashes or en dashes. Replace with periods, commas, or colons.
- Boldface on individual words or phrases.
- Emojis.
- Filler openers: "I am excited to...", "I am thrilled...", "I would love to..."
- Significance inflation: "passionate about", "deeply committed to", "uniquely positioned"
- Rule of three lists that sound like marketing copy
- Any sentence that starts with "I" three or more times in a row
- Generic closings: "I look forward to hearing from you", "Thank you for your consideration"

Replace the generic closing with something direct:

> "Happy to connect if you want to talk through my background."

or

> "Available to start [month]. Happy to answer any questions."

**Formatting:** Single-column plain .docx. No header/footer. No graphics. Candidate name and contact info at the top in plain text.

---

## Output Format (.docx)

Both resume and cover letter must be output as native .docx files, not exported from Google Docs or converted from another format.

Use the docx skill to generate the file programmatically. This ensures clean XML that ATS parsers can read reliably.

Resume filename: `[lastname]_[firstname]_[jobtitle]_resume.docx`
Cover letter filename: `[lastname]_[firstname]_[jobtitle]_coverletter.docx`

Job title in filename should match the exact title from the posting, lowercase, spaces replaced with underscores.

---

## Sources

- interviewstreet/hiring-agent, HackerRank, MIT License
- blader/humanizer, Siqi Chen, MIT License
