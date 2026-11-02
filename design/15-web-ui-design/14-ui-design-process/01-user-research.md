# User Research: Who We're Building For

> This is our compass. When designing any screen, ask: "Would Will find this intuitive? Would Ashley be proud to build with this? Would Valerie get the insight she needs? Would Edward understand it at a glance?"

---

## Personas

### Will, the Test Engineer

**His world**: Will runs tests - automated and manual - day in, day out. He keeps a personal arsenal of ready-to-go commands: get logs, restart pods, start traffic, generate Helm values. He learned early that typing commands from memory is asking for mistakes; a tested command saved in his notes is always the winner.

**What makes him curse**:
- "Where's that test result from three months ago?" - Digging through ancient shared drives while a manager waits on a call
- "How do I even connect to this interface?" - R&D designed it, R&D didn't document it
- "I ran this test yesterday, why am I setting it up again?" - No good way to save and reuse test configurations
- "The report is 47 pages of... what exactly?" - Results exist but aren't actionable

**What would make him recommend WAVE**: "I found the issue in 10 minutes instead of 2 hours. The tool just... worked. I could focus on actually testing instead of fighting the tooling."

**His goal**: Find issues. Not fight infrastructure.

---

### Ashley, the Lead Engineer

**Her world**: Ashley knows the product cold - every microservice, every lab environment, every production quirk. She's run so many regressions she probably sees test plans in her sleep. She's the one everyone asks when something doesn't make sense.

But Ashley is frustrated. She's capable of building sophisticated automation, yet she keeps getting pulled into tasks anyone could do. "Check the logs." "Run that suite again." "What was the result?" She wants to work on *hard* problems.

**What makes her curse**:
- "I already solved this. Why is someone solving it again?" - Her scripts exist but nobody can find them
- "I'm not a status board" - Constant interruptions asking how tests went
- "This should take one command, not twenty" - Packaging complexity is harder than it should be
- "I built this tool but I can't share it properly" - No good way to version, document, or distribute automation

**What would make her recommend WAVE**: "I finally have a workshop, not a sweatshop. I build automation that the whole team actually uses. I work on hard problems. And nobody asks me for status anymore - they just look at the dashboard."

**Her goal**: Package expertise. Multiply her impact. Stop being a bottleneck.

---

### Valerie, the System Architect

**Her world**: Valerie juggles parallel streams of change - new interface support, revamp of a legacy feature, fixing memory issues in microservices - while trying to understand how all of it impacts the product. She needs to spot problems early, before they compound.

Currently, her main tool is asking people. "How did the tests go?" "Did you notice anything slower?" She gets HTML reports that look like they were designed in 1997 and contain everything except what she actually needs.

**What makes her curse**:
- "I need to compare this run to last month's" - No easy way to see trends or deltas
- "Is the system slower or is the test flaky?" - Can't distinguish signal from noise
- "Just show me what changed" - Wading through complete reports for specific insights
- "I shouldn't have to ask for this data" - Information exists but requires human intermediaries

**What would make her recommend WAVE**: "I can actually see the impact of my changes. I spotted a 15% latency regression before it hit production - from my desk, without asking anyone."

**Her goal**: Understand impact. Spot anomalies. Make informed decisions.

---

### Edward, the Release Planner

**His world**: Edward coordinates releases. He needs to know: What tests will run? How long will it take? What resources do we need? He's not technical - he's operational. And he's visual. Juggling multiple documents to understand a single test is his personal hell.

**What makes him curse**:
- "How long will this regression take?" - Nobody knows until it's running
- "What does this test actually do?" - Has to read the acceptance plan doc, cross-reference the suite list, then ask someone anyway
- "Is it done yet?" - No visibility without interrupting engineers
- "Can I get a professional summary?" - The reports look like spreadsheet accidents

**What would make him recommend WAVE**: "I can plan releases properly now. I see what's being tested, watch progress in real-time, and get clean reports I'm not embarrassed to share. I stopped bothering the engineers - everything I need is right there."

**His goal**: Plan accurately. Track visibly. Report professionally.

---

## Jobs to Be Done

| Job | Who | Current Pain | WAVE's Answer |
|-----|-----|--------------|---------------|
| **Create a test** | Will | Setup takes hours. Hard to reuse configurations. | Drag-and-drop builder. Save and share configurations. |
| **Build automation** | Ashley | Scripts exist but aren't discoverable or shareable. | Compose complex from simple. |
| **Run a regression** | Will | No feedback until done. Results scattered. | Real-time progress. Everything in one place. |
| **Debug a failure** | Will, Ashley | Scroll through logs. "What was the value at failure time?" | Breakpoints, variable watcher, visual failure location. Re-execute from any point. |
| **Plan a release** | Edward | No visibility into scope, duration, or progress. | Test inventory. Real-time tracking. Professional reports. |
| **Find past results** | Everyone | Dig through SharePoint. Ask people. Waste time. | Searchable history. Filters. Instant access. |

---

## Design Principles (Derived)

1. **Show, don't tell** - Visual representation over text walls
2. **Fail forward** - When something breaks, show exactly where and why
3. **Respect expertise** - Power users need depth; occasional users need clarity
4. **One source of truth** - Everything in WAVE, not scattered across drives and docs
5. **No interruptions needed** - Self-service over asking people

---

## The Ecosystem

```
Ashley builds tools  →  Will runs tests  →  Valerie analyzes impact  →  Edward plans releases
    (automation)          (execution)           (insight)                (coordination)
```

When everyone succeeds:
- Ashley works on hard problems, not status updates
- Will finds issues, not fights tooling
- Valerie spots problems before production
- Edward plans with confidence, reports with pride

**Releases ship with confidence.**
