# Learning Journal — Authoring Skill

This file defines the conventions, rules, and quality standards for maintaining this learning journal. Apply these when adding notes and Q&A for any new course or module.

---

## 1. Repository Structure

```
courses/
  <course-slug>/
    README.md                          ← course overview, modules list, status
    modules/
      module-NN-topic-name/
        notes.md                       ← study notes
        qa-review.md                   ← Q&A for self-testing and exam prep
        images/                        ← screenshots from slides/videos
lab-notes/
  lab-NN-lab-name.md                   ← hands-on lab write-ups
shared-resources/
  gcp-services-overview.md             ← cross-module service reference
  key-terms.md                         ← glossary
README.md                              ← repo overview
```

**Module folder naming:** `module-NN-kebab-case-topic` — e.g. `module-03-vpc-and-compute-engine`

---

## 2. notes.md Structure

### Required Sections (in order)

```markdown
# Module N: Topic Name

## Status: ✅ Completed (Day N · YYYY.MM.DD)

## 🔗 Quick Navigation
- Q&A Review: [qa-review.md](qa-review.md)

---

## 📝 Learning Objectives
By the end of this module, you will understand:
- [x] Objective one
- [x] Objective two

---

## 📚 Key Concepts

### 1. First Topic
### 2. Second Topic
...

---

## 🔗 References & Links
| Resource | Description |
|---|---|
| [Link text](url) | What it covers |

---

## ❓ Key Questions to Review
- Bullet list of conceptual questions without answers

---

## 📌 Summary
| Concept | Key Point |
|---|---|
| Term | One-line takeaway |
```

### Notes Content Rules

- Use `> **Official slide wording:**` blockquotes for verbatim slide content
- Use `> **Exam Tip:**` or `> **Exam Key Point:**` blockquotes for exam-relevant callouts
- Embed images inline: `![Alt text](images/filename.png)`
- All image files must exist in the `images/` subfolder — no broken references
- Tables: use padded column separators for readability (align dashes to column width)
- No 3+ consecutive blank lines anywhere in the file

---

## 3. qa-review.md Structure

### File Header

```markdown
# Module N: Topic Name - Q&A Review

**Purpose:** Quick reference guide for reviewing all Q&A from Module N — <Topic>. Use this for self-testing and exam prep.

**How to Use:**
1. Read each question and try to answer without looking
2. Check your answer against the provided explanation
3. Mark ✅ if correct, ⏳ if it needs more review
4. Focus on ⏳ questions for final study

**Reference Notes:** [notes.md](notes.md)

---
```

### Topic Sections

Group questions by topic using `##` headings:

```markdown
## Topic 1: Topic Name

### Q1: Question Title
...

## Topic 2: Another Topic

### Q4: Question Title
...
```

### Question Format (MCQ — mandatory for all questions)

```markdown
### QN: Short Descriptive Title

**Question:**
<Full question text — scenario-based where possible>

**Options:**
- A) Option text
- B) Option text
- C) Option text
- D) Option text

**Answer:** **B) Exact option text repeated here**

**Explanation:**
<Detailed explanation of why the correct answer is right. Reference official docs/slide wording where applicable.>

**Why not the others:**
- A) Why this is wrong
- C) Why this is wrong
- D) Why this is wrong

---
```

### Q&A Rules

| Rule | Detail |
|---|---|
| **All MCQ** | Every question must have A/B/C/D options — no open-ended questions |
| **Numbering** | Sequential Q1, Q2, Q3… with no gaps; no OQ labels |
| **Heading format** | `# Module N: Topic Name - Q&A Review` |
| **No section split** | Do NOT create a separate section for "Official Quiz Questions" — all questions flow in one list under topic sections |
| **Answer format** | `**Answer:** **B) <exact option text>**` |
| **Why not the others** | Always present; covers every incorrect option; label is always `**Why not the others:**` |
| **Multi-select questions** | For "select 3" style questions, list all 5 options; answer shows correct set; Why not the others covers the 2 wrong ones |
| **Scenario-based** | Prefer scenario/context in the question stem rather than pure definition recall |
| **Blank lines** | No 3+ consecutive blank lines |
| **Encoding** | UTF-8; no replacement characters (�) |

---

## 4. Question Design Guidelines

### Writing Good MCQ Options

- One clearly correct answer; three plausible distractors
- Distractors should represent common misconceptions or related-but-wrong concepts
- Avoid "all of the above" / "none of the above"
- Keep option length roughly consistent
- For "which is INCORRECT?" questions — make 3 options correct, 1 subtly wrong (e.g. wrong standard/number/unit)

### Scenario Patterns That Work Well

| Pattern | Example |
|---|---|
| Cost optimization | "A company runs VMs 24/7 and wants to minimize cost..." |
| Compliance/residency | "Data must stay in Germany and survive one datacenter failure..." |
| Troubleshooting | "Pipeline fails with quota error, resolves after 3 minutes..." |
| Architecture choice | "15 projects, central security team needs to control firewall rules..." |
| Migration | "Move 2 PB of on-prem data to GCS; 1 Gbps link..." |
| Wrong answer trap | "Which statement is INCORRECT about Google's sustainability..." |

### Explanation Quality Bar

- Correct answer: explain *why* it's correct, not just restate it
- Reference official terms (e.g. "Titan chip", "edge PoP", "Shared VPC host project")
- Include a comparison table in the explanation when two similar options are close (e.g. Firestore vs Bigtable)
- **Why not the others:** each distractor gets a specific, technical reason — not "it's wrong"

---

## 5. Pre-Commit Quality Checklist

Run before every commit:

```python
import re, os

base = r'<path-to-modules>'

for mod in sorted(os.listdir(base)):
    mod_path = os.path.join(base, mod)
    issues = []

    for fname in ['notes.md', 'qa-review.md']:
        path = os.path.join(mod_path, fname)
        if not os.path.exists(path):
            issues.append(f'MISSING: {fname}')
            continue
        with open(path, 'r', encoding='utf-8') as f:
            content = f.read()

        if '\ufffd' in content:
            issues.append(f'{fname}: encoding artifacts')
        if re.search(r'\n{4,}', content):
            issues.append(f'{fname}: 3+ consecutive blank lines')

        if fname == 'qa-review.md':
            first_line = content.split('\n')[0]
            if not re.match(r'^# Module \d+: .+ - Q&A Review', first_line):
                issues.append(f'{fname}: heading missing topic name')

            oq = re.findall(r'^### OQ\d+:', content, re.M)
            if oq: issues.append(f'{fname}: OQ labels remain: {oq}')

            qs = [int(m) for m in re.findall(r'^### Q(\d+):', content, re.M)]
            for j in range(len(qs)-1):
                if qs[j+1] != qs[j]+1:
                    issues.append(f'{fname}: numbering gap Q{qs[j]}->Q{qs[j+1]}')

            for b in re.split(r'(?=^### Q\d+:)', content, flags=re.M):
                if not b.startswith('### Q'): continue
                qnum = re.match(r'### (Q\d+):', b).group(1)
                for section in ['**Options:**', '**Answer:**', '**Why not the others:**']:
                    if section not in b:
                        issues.append(f'{fname}: {qnum} missing {section}')

            if 'Official Google Module Quiz Questions' in content:
                issues.append(f'{fname}: old OQ section header present')

        if fname == 'notes.md':
            for img in re.findall(r'!\[.*?\]\((images/[^)]+)\)', content):
                if not os.path.exists(os.path.join(mod_path, img)):
                    issues.append(f'notes.md: broken image: {img}')

    if issues:
        print(f'\n=== {mod} ===')
        for issue in issues: print(f'  FAIL: {issue}')
    else:
        print(f'PASS: {mod}')
```

---

## 6. Content Sources (Priority Order)

| Priority | Source | Usage |
|---|---|---|
| 1 | Official ILT slide deck | Exact wording for key definitions; quiz questions |
| 2 | Session/course video | Explanations, examples, analogies |
| 3 | Official GCP documentation | Deep dives, limits, exact specs |
| 4 | Cert prep questions | Additional MCQ scenarios |

When using slide content verbatim, wrap it in a blockquote:
```markdown
> **Official slide wording:** "..."
```

---

## 7. Commit Convention

```
feat: add <course> module <N> — <topic>
fix: correct Q&A answers in module <N>
chore: convert open-ended questions to MCQ in all modules
```
