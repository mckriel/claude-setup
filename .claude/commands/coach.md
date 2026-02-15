---
description: Get guided through implementing a feature yourself — interfaces, contracts, test cases, and conceptual coaching without code handouts
allowed-tools: Agent
---

🚨 CRITICAL: You MUST delegate this work to the coach agent. DO NOT provide implementation guidance yourself.

Guide the developer through implementing this themselves: $ARGUMENTS

## MANDATORY Workflow

**STEP 1: Invoke the coach agent**

Use the Task tool to delegate to the coach agent:
```
Task(subagent_type='coach', prompt='Guide the developer through implementing this feature themselves. Provide interfaces, contracts, test cases, and conceptual explanations. NEVER write implementation code. Feature: $ARGUMENTS')
```

**What you MUST do:**
- ✅ Use the Task tool with subagent_type='coach'
- ✅ Pass the feature description and any HLD context to the agent
- ✅ Wait for the agent to complete its analysis
- ✅ Present the agent's coaching output (contracts, test cases, concepts)

**What you MUST NOT do:**
- ❌ Write implementation code for the developer
- ❌ Provide step-by-step coding instructions
- ❌ Skip the agent and coach directly
- ❌ Supplement the agent's output with your own code examples

**Your role:** You are DELEGATING to the coach agent. Present its output and let the developer ask follow-up questions, which you route back to the coach.
