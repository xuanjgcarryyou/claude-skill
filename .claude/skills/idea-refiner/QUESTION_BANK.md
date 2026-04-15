# Question Bank: Idea Refiner

A reference bank of high-impact clarifying questions, organized by category. Use these to drive the refinement dialogue — picking the ONE most important question at each turn based on what's still unknown.

---

## Category 1: Scope and Lifetime

Questions that determine whether this is a throwaway script or a product to maintain.

- Is this something you'll use once, or something that needs to keep running over time?
- Is this just for you, or will other people use it?
- Do you expect the requirements to change significantly after v1?
- Is there a hard deadline, or can we take time to do it right?
- What does "done" look like for the first version?

---

## Category 2: Users and Access

Questions that determine auth, multi-tenancy, and permission complexity.

- Is this single-user or multi-user?
- Do different users need different permissions or roles?
- Does it need authentication (login/session), or is it internal/trusted?
- Do you need to isolate data between users (multi-tenant), or is shared data OK?
- Will this be accessed by other services/APIs, or only by humans?

---

## Category 3: Data and Persistence

Questions that determine storage strategy, schema complexity, and retention.

- What data does this system need to store?
- How long does the data need to persist? (session only / days / forever)
- Where does the data come from? (user input / external API / file / database)
- How much data are we talking about? (a few rows / thousands / millions)
- Does data need to be searchable, or is direct lookup enough?
- Is any of the data sensitive (PII, credentials, financial)?

---

## Category 4: Real-time vs. Async

Questions that determine architecture (request/response vs. queue/worker).

- Does the user need an immediate result, or can it process in the background?
- How fast does a response need to be? (instant / seconds / minutes is OK)
- Does anything need to happen on a schedule or timer?
- Is there any event or trigger that kicks off processing (webhook, file upload, user action)?
- Do multiple things need to happen in parallel, or is sequential fine?

---

## Category 5: Integration and Environment

Questions that reveal constraints and external dependencies.

- Does this need to talk to any existing systems? (database, API, auth service)
- Is there an existing codebase this needs to fit into, or is it greenfield?
- What language/runtime/platform are you working in, or is that flexible?
- Does it need to run in a specific environment? (browser / server / CLI / mobile)
- Are there any third-party APIs or services involved?

---

## Category 6: Priorities and Trade-offs

Questions that reveal what the user actually cares about most.

- Are you optimizing for: fastest to build, easiest to maintain, or most scalable?
- What's the one thing that absolutely must work correctly in v1?
- What would you cut if you had to ship something smaller?
- Is performance critical, or is "good enough" fine for now?
- How much tech debt are you OK with in exchange for shipping faster?

---

## Category 7: Architecture Constraints

Questions that eliminate options before you propose them.

- Is there a preference for any specific architecture pattern? (monolith, services, serverless)
- Do you have infrastructure already in place (cloud provider, DB, hosting)?
- Are there team size or skill constraints? (solo dev / small team / specialists available)
- Is this something that needs to scale significantly, or is load predictable and small?
- Does it need to be open-source, self-hosted, or can it use managed services?

---

## Usage Notes

- Pick the ONE question that most changes the architecture given what you know so far
- After 2–4 questions, stop asking and start proposing — more questions rarely improve the outcome
- If the user gives a vague answer, ask a more concrete follow-up on that same topic before moving on
- Skip any question whose answer won't change the direction you'd recommend
