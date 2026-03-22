# How to Use SKILL.md in Claude Code (or other AI tools)

## 1. Treat it as a “System Prompt” for your codebase

Claude Code works best when it understands your **standards + thinking style**.

Put your `SKILL.md` in your repo root:

```
/project
  ├── SKILL.md
  ├── README.md
  ├── cmd/
  ├── internal/
```

Then explicitly tell Claude:

```
Follow the guidelines defined in SKILL.md while generating code.
```

---

## 2. Use it as Context for Every Task

### Example: API Implementation

Instead of:

```
Create a user API in Go
```

Do this:

```
Using SKILL.md:
- Implement a user service in Go
- Follow cursor pagination
- Add proper error handling structure
- Include observability (logs + metrics)
- Use context propagation everywhere
```
---

## 3. Use It for Code Review (VERY POWERFUL)

Paste code and say:

```
Review this code using SKILL.md standards:
- Check scalability issues
- Check concurrency issues
- Check observability gaps
- Suggest improvements
```

---

## 4. Use It for System Design

```
Using SKILL.md:
Design a URL shortener service

Include:
- API design
- DB schema
- caching strategy
- failure scenarios
- scaling plan
```

---

## 5. Turn It Into Constraints (Best Trick)

Add constraints like:

```
Strictly follow:
- No offset pagination
- Must include idempotency
- Must include request_id logging
- Must handle failure scenarios
```
---

# Extra Setup (Optional)

Create a file:

```
/.claude/context.md
```

Put:

```
Always follow SKILL.md for:
- API design
- error handling
- concurrency
- observability
```

---

# Real Insight (Important)

Without SKILL.md:

> AI tools writes **correct code**

With SKILL.md:

> AI tools writes **senior-level production systems**
