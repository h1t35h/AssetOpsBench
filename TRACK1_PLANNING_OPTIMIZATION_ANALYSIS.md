# Track 1 Planning Optimization Analysis

## Executive Summary

This document provides a comprehensive analysis of `track1_planning.py` and identifies optimization approaches to achieve better planning results in the AssetOpsBench Track 1 competition.

**File Location:** `src/agent_hive/workflows/track1_planning.py`

**Competition Goal:** Improve prompt engineering for task planning while keeping agents, ReAct framework, and execution components fixed.

---

## Table of Contents

1. [Current State Analysis](#current-state-analysis)
2. [Evaluation Criteria](#evaluation-criteria)
3. [Editable Sections Overview](#editable-sections-overview)
4. [Optimization Approaches](#optimization-approaches)
5. [Implementation Strategies](#implementation-strategies)
6. [Examples & Best Practices](#examples--best-practices)
7. [Common Pitfalls to Avoid](#common-pitfalls-to-avoid)
8. [Testing & Validation](#testing--validation)

---

## Current State Analysis

### Architecture Overview

```
NewPlanningWorkflow
├── generate_steps()           # Main planning logic
│   ├── Agent Description Formatting (EDITABLE)
│   ├── get_prompt()          # Prompt template (EDITABLE)
│   ├── LLM Call              # Fixed
│   └── Response Parsing      # Partially EDITABLE
└── run()                     # Fixed workflow execution
```

### Current Prompt Structure

**Current template at lines 187-214:**
- Simple instruction-based format
- Basic constraint listing
- Minimal guidance on agent selection
- Generic output format specification

### Current Agent Description Format (lines 79-86)

```python
agent_descriptions += f"\n({ii + 1}) Agent name: {aagent.name}"
agent_descriptions += f"\nAgent description: {aagent.description}"
if "task_examples" in aagent.__dict__ and aagent.task_examples:
    agent_descriptions += f"\nTasks that agent can solve:"
    for idx, task_example in enumerate(aagent.task_examples, start=1):
        agent_descriptions += f"\n{idx}. {task_example}"
```

**Issues:**
- Flat structure with minimal visual hierarchy
- Task examples may be too verbose
- No categorization or capability grouping
- Limited agent differentiation

---

## Evaluation Criteria

Based on `src/agent_hive/agents/plan_reviewer_prompt.py` and `src/evaluation/analyze.py`, plans are evaluated on:

### 1. **Completeness** (Critical)
- All necessary steps to accomplish the task are included
- Each step contains all relevant information and required parameters
- No crucial actions are missed

### 2. **Relevance** (Critical)
- Each step directly contributes to solving the task
- No unnecessary or superfluous steps

### 3. **Correctness** (Critical)
- Steps are logically consistent and correctly ordered
- Dependencies between steps are valid
- Proper sequence is maintained

### 4. **Expertise Alignment** (Critical)
- Steps fall within agent capabilities
- Agent selection matches the task requirements
- No tasks assigned beyond agent expertise

### 5. **Efficiency**
- No redundant actions
- Minimal complexity while maintaining completeness
- Optimal number of steps (typically < 5)

### 6. **Clarity**
- Tasks are clear, unambiguous, and actionable
- Plan is easy to understand and logically structured
- Task descriptions are grounded in concrete data

### Additional Metrics (from analyze.py):
- `task_completion`: Did the plan complete the overall task?
- `data_retrieval_accuracy`: Was data retrieved correctly?
- `agent_sequence_correct`: Were agents used in the right order?
- `clarity_and_justification`: Is the reasoning clear?
- `hallucinations`: Any fabricated information?

---

## Editable Sections Overview

### Section 1: Variable Declarations (Lines 16-19)
```python
# =========================================================
# TODO: Participants can edit this section ONLY
# Add variable, dict. no more any import just any inline code
# =========================================================
```

**Purpose:** Add helper variables, dictionaries, or inline utility code
**Constraints:** No imports allowed

### Section 2: Agent Description Formatting (Lines 64-89)
```python
for ii, aagent in enumerate(task.agents):
    agent_descriptions += f"\n({ii + 1}) Agent name: {aagent.name}"
    # ... format agent information
```

**Purpose:** Customize how agent information is collected and presented
**Allowed Changes:**
- Numbering style or bullet points
- Additional metadata (capabilities, tags)
- Example formatting
- Emojis for clarity
- Additional context or grouping

### Section 3: LLM Response Post-Processing (Lines 104-174)
```python
# Post-processing logic
task_pattern = r"#Task\d+: (.+)"
agent_pattern = r"#Agent\d+: (.+)"
# ... regex parsing
```

**Purpose:** Customize how LLM responses are parsed and processed
**Allowed Changes:**
- Response parsing logic
- Error handling
- Task construction logic
**NOT Allowed:**
- Changing workflow execution
- Modifying Task object structure

### Section 4: Prompt Template (Lines 180-214)
```python
def get_prompt(self, task_description, agent_descriptions):
    prompt = f"""
    🚀 You are an AI assistant...
    """
```

**Purpose:** Craft the prompt that guides the LLM in generating plans
**Allowed Changes:**
- Wording and phrasing
- Structure and formatting
- Examples and demonstrations
- Emojis for emphasis
**NOT Allowed:**
- Changing function signature
- Modifying workflow logic

---

## Optimization Approaches

### 🎯 Approach 1: Enhanced Agent Description Formatting

**Problem:** Current flat format doesn't highlight agent capabilities effectively.

**Solutions:**

#### 1.1 Structured Capability Breakdown
```python
for ii, aagent in enumerate(task.agents):
    agent_descriptions += f"\n{'='*60}\n"
    agent_descriptions += f"🤖 AGENT {ii + 1}: {aagent.name}\n"
    agent_descriptions += f"{'='*60}\n"
    agent_descriptions += f"📋 Description: {aagent.description}\n"

    if "task_examples" in aagent.__dict__ and aagent.task_examples:
        agent_descriptions += f"\n✅ Capabilities (Example Tasks):\n"
        for idx, task_example in enumerate(aagent.task_examples[:5], start=1):
            # Truncate long examples
            truncated = task_example[:100] + "..." if len(task_example) > 100 else task_example
            agent_descriptions += f"   {idx}. {truncated}\n"
    agent_descriptions += "\n"
```

**Benefits:**
- Clear visual hierarchy
- Better agent differentiation
- Easier to scan and understand

#### 1.2 Capability Categorization
```python
# In Section 1: Variable Declarations
AGENT_CAPABILITIES = {
    "IoT Data Download": ["data_retrieval", "sensor_query", "historical_data"],
    "Failure Mode and Sensor Relevancy Expert": ["failure_analysis", "sensor_mapping"],
    "Time-Series Foundation Model": ["forecasting", "anomaly_detection"],
    "Work Order Generator": ["maintenance_planning", "work_order_creation"]
}

# In Section 2: Agent Description Formatting
for ii, aagent in enumerate(task.agents):
    agent_descriptions += f"\n🤖 Agent {ii + 1}: {aagent.name}\n"
    agent_descriptions += f"Description: {aagent.description}\n"

    # Add capability tags
    if aagent.name in AGENT_CAPABILITIES:
        agent_descriptions += f"Key Capabilities: {', '.join(AGENT_CAPABILITIES[aagent.name])}\n"

    # Show selective examples
    if "task_examples" in aagent.__dict__ and aagent.task_examples:
        agent_descriptions += f"\nExample Use Cases:\n"
        for idx, task_example in enumerate(aagent.task_examples[:3], start=1):
            agent_descriptions += f"{idx}. {task_example}\n"
```

**Benefits:**
- Quick capability identification
- Reduced token usage
- Better agent selection by LLM

---

### 🎯 Approach 2: Advanced Prompt Engineering

**Problem:** Current prompt lacks specificity and strategic guidance.

#### 2.1 Chain-of-Thought Planning
```python
prompt = f"""
You are an expert task planning assistant for industrial asset operations.

🎯 YOUR MISSION: Break down the complex problem into a clear, executable plan using available agents.

📊 PLANNING PROCESS (Think step-by-step):
1. **Understand**: What is the end goal? What information is needed?
2. **Identify**: Which agents can help? What are their capabilities?
3. **Sequence**: What order makes logical sense? What depends on what?
4. **Validate**: Does each step have clear inputs/outputs? Are all steps necessary?

⚠️ CRITICAL RULES:
- Use ONLY the agents listed below (no exceptions)
- Keep plans under 5 steps
- Each step must be clear, specific, and actionable
- Tasks must include ALL necessary details (asset names, sensor names, time ranges, etc.)
- Dependencies must be explicitly stated using #S1, #S2, etc.
- Agent selection must match the task requirements exactly

📝 OUTPUT FORMAT (Follow exactly):

## Step 1
#Task1: <Clear, specific description with all necessary parameters>
#Agent1: <exact agent name from list>
#Dependency1: None
#ExpectedOutput1: <Concrete, measurable output>

## Step 2
#Task2: <Next specific task>
#Agent2: <exact agent name>
#Dependency2: #S1 (or more dependencies like #S1, #S2)
#ExpectedOutput2: <Concrete output>

[Continue as needed...]

## AVAILABLE AGENTS ##
{agent_descriptions}

## PROBLEM TO SOLVE ##
{task_description}

⚡ Generate your plan now (plan only, no explanation):
"""
```

**Benefits:**
- Structured thinking process
- Clearer constraints
- Better adherence to format

#### 2.2 Few-Shot Enhanced Prompt
```python
# In Section 1: Add example plans
EXAMPLE_PLANS = """
EXAMPLE 1:
Problem: "Get sensor data for Chiller 6 and detect anomalies"

## Step 1
#Task1: Download historical sensor data for Chiller 6 at the main site, including all temperature and pressure sensors for the last 30 days
#Agent1: IoT Data Download
#Dependency1: None
#ExpectedOutput1: JSON file containing historical sensor readings

## Step 2
#Task2: Analyze the downloaded sensor data from #S1 to detect anomalies in temperature patterns
#Agent2: Time-Series Foundation Model
#Dependency2: #S1
#ExpectedOutput2: Anomaly detection results with timestamps and severity scores

EXAMPLE 2:
Problem: "List failure modes for Chiller 9 that can be detected by installed sensors"

## Step 1
#Task1: Retrieve all failure modes associated with Chiller 9
#Agent1: Failure Mode and Sensor Relevancy Expert
#Dependency1: None
#ExpectedOutput1: List of failure modes for Chiller 9

## Step 2
#Task2: For each failure mode from #S1, identify which sensors are installed on Chiller 9 that can detect these failures
#Agent2: Failure Mode and Sensor Relevancy Expert
#Dependency2: #S1
#ExpectedOutput2: Mapping of failure modes to installed sensors
"""

# In Section 4: Prompt Template
prompt = f"""
You are an AI task planner for industrial asset operations.

First, study these EXAMPLE PLANS to understand the format and level of detail required:

{EXAMPLE_PLANS}

Now, create a similar plan following the exact same format.

## AVAILABLE AGENTS ##
{agent_descriptions}

## PROBLEM TO SOLVE ##
{task_description}

## YOUR PLAN (following the example format) ##
"""
```

**Benefits:**
- Concrete examples improve format adherence
- Shows appropriate level of detail
- Demonstrates proper dependency usage

#### 2.3 Agent-Matching Guidance
```python
prompt = f"""
You are an expert planning assistant specializing in industrial asset operations.

## STEP-BY-STEP PLANNING GUIDE ##

1️⃣ ANALYZE THE PROBLEM
Read the problem carefully and identify:
- What data needs to be retrieved?
- What analysis needs to be performed?
- What output format is required?

2️⃣ MATCH TASKS TO AGENTS
Use this decision tree:
- Need IoT/sensor data or asset information? → Use "IoT Data Download"
- Need failure mode analysis or sensor-failure mapping? → Use "Failure Mode and Sensor Relevancy Expert"
- Need time-series forecasting or anomaly detection? → Use "Time-Series Foundation Model"
- Need to create maintenance work orders? → Use "Work Order Generator"

3️⃣ BUILD THE SEQUENCE
- Start with data retrieval (usually IoT agent)
- Then analysis (FMSR or TSFM agent)
- Finally action (Work Order if needed)
- Each step should output something concrete for the next step

4️⃣ SPECIFY DETAILS
Each task MUST include:
✓ Exact asset names/IDs from the problem
✓ Specific sensor names if mentioned
✓ Time ranges if relevant
✓ File references if provided
✓ Clear expected output

## CONSTRAINTS ##
❌ Do NOT add steps that aren't necessary
❌ Do NOT use agents for tasks outside their expertise
❌ Do NOT create vague tasks like "analyze the data" (be specific!)
❌ Do NOT exceed 5 steps
❌ Do NOT skip dependencies between steps

## OUTPUT FORMAT ##
## Step N
#TaskN: <Specific task with all details>
#AgentN: <Exact agent name>
#DependencyN: <None or #S1, #S2, etc.>
#ExpectedOutputN: <Concrete, measurable output>

## AVAILABLE AGENTS ##
{agent_descriptions}

## PROBLEM TO SOLVE ##
{task_description}

## YOUR OPTIMIZED PLAN ##
"""
```

**Benefits:**
- Decision tree helps with agent selection
- Explicit detail requirements
- Clear do's and don'ts

---

### 🎯 Approach 3: Improved Response Parsing

**Problem:** Regex parsing may miss edge cases or multi-line content.

#### 3.1 Robust Multi-line Parsing
```python
# In Section 3: Post-processing

# More robust patterns that handle multi-line and edge cases
task_pattern = r"#Task(\d+):\s*(.+?)(?=\n#Agent|\n#|\Z)"
agent_pattern = r"#Agent(\d+):\s*(.+?)(?=\n#Dependency|\n#|\Z)"
dependency_pattern = r"#Dependency(\d+):\s*(.+?)(?=\n#ExpectedOutput|\n#|\Z)"
output_pattern = r"#ExpectedOutput(\d+):\s*(.+?)(?=\n##|\n#Task|\Z)"

# Use DOTALL flag for multi-line matching
tasks = re.findall(task_pattern, final_plan, re.DOTALL | re.MULTILINE)
agents = re.findall(agent_pattern, final_plan, re.DOTALL | re.MULTILINE)
dependencies = re.findall(dependency_pattern, final_plan, re.DOTALL | re.MULTILINE)
outputs = re.findall(output_pattern, final_plan, re.DOTALL | re.MULTILINE)

# Clean extracted content
tasks = [t[1].strip() if isinstance(t, tuple) else t.strip() for t in tasks]
agents = [a[1].strip() if isinstance(a, tuple) else a.strip() for a in agents]
dependencies = [d[1].strip() if isinstance(d, tuple) else d.strip() for d in dependencies]
outputs = [o[1].strip() if isinstance(o, tuple) else o.strip() for o in outputs]
```

#### 3.2 Agent Name Fuzzy Matching
```python
# In Section 1: Helper function
def find_best_agent_match(agent_name_from_plan, available_agents):
    """Find the best matching agent using fuzzy matching"""
    agent_name_lower = agent_name_from_plan.lower().strip()

    # Exact match
    for agent in available_agents:
        if agent.name.lower() == agent_name_lower:
            return agent

    # Partial match (contains)
    for agent in available_agents:
        if agent_name_lower in agent.name.lower() or agent.name.lower() in agent_name_lower:
            return agent

    # Keyword matching
    keywords = {
        "iot": ["iot", "data", "download", "sensor"],
        "fmsr": ["failure", "mode", "sensor", "relevancy"],
        "tsfm": ["time", "series", "forecast", "anomaly"],
        "wo": ["work", "order", "generator", "maintenance"]
    }

    for keyword_set, terms in keywords.items():
        if any(term in agent_name_lower for term in terms):
            for agent in available_agents:
                if any(term in agent.name.lower() for term in terms):
                    return agent

    # Fallback
    return available_agents[0]

# In Section 3: Use the helper
selected_agent = find_best_agent_match(agent_name, task.agents)
```

**Benefits:**
- Handles variations in agent name formatting
- More robust against LLM output variations
- Reduces plan failures due to name mismatches

---

### 🎯 Approach 4: Context-Aware Planning

#### 4.1 Problem Type Detection
```python
# In Section 1: Add problem categorization
def detect_problem_type(description):
    """Categorize the problem to guide planning"""
    desc_lower = description.lower()

    problem_types = []

    if any(word in desc_lower for word in ["list", "get", "download", "retrieve", "show"]):
        problem_types.append("data_retrieval")

    if any(word in desc_lower for word in ["failure", "fault", "malfunction"]):
        problem_types.append("failure_analysis")

    if any(word in desc_lower for word in ["forecast", "predict", "anomaly", "detect"]):
        problem_types.append("analysis")

    if any(word in desc_lower for word in ["work order", "maintenance", "repair"]):
        problem_types.append("action")

    return problem_types

# In Section 4: Use in prompt
problem_types = detect_problem_type(task_description)
context_hint = ""
if problem_types:
    context_hint = f"\n🎯 Problem Type Detected: {', '.join(problem_types)}\n"
    context_hint += "Consider this when selecting agents and ordering steps.\n"

prompt = f"""
{context_hint}
...rest of prompt...
"""
```

#### 4.2 Entity Extraction for Better Grounding
```python
# In Section 1: Entity extraction
import re

def extract_entities(description):
    """Extract key entities from the problem description"""
    entities = {
        "assets": re.findall(r'(Chiller|Pump|Boiler|HVAC|CU\d+)\s*\d*', description, re.IGNORECASE),
        "sensors": re.findall(r'(temperature|pressure|flow|current|voltage)\s*sensor', description, re.IGNORECASE),
        "sites": re.findall(r'(site|location|facility)\s*\w+', description, re.IGNORECASE),
        "time_ranges": re.findall(r'(last|past|previous)\s+\d+\s+(days|weeks|months|hours)', description, re.IGNORECASE)
    }
    return {k: list(set(v)) for k, v in entities.items() if v}

# In Section 4: Add to prompt
entities = extract_entities(task_description)
entity_reminder = ""
if entities:
    entity_reminder = "\n⚠️ IMPORTANT: Your plan MUST reference these specific entities:\n"
    for ent_type, ent_list in entities.items():
        entity_reminder += f"- {ent_type.title()}: {', '.join(ent_list)}\n"

prompt = f"""
...
## PROBLEM TO SOLVE ##
{task_description}

{entity_reminder}
...
"""
```

**Benefits:**
- Ensures plan references specific entities from problem
- Reduces hallucination
- Improves grounding

---

### 🎯 Approach 5: Leveraging PlanReviewerAgent Insights

**Insight:** The `PlanReviewerAgent` exists but isn't used in track1_planning.py. However, we can learn from its evaluation criteria.

#### 5.1 Self-Critique Prompting
```python
prompt = f"""
You are an AI planning expert. Create a plan that will pass the following review criteria:

✅ REVIEW CHECKLIST (Your plan will be evaluated on):
1. Completeness: All necessary steps included, no crucial actions missed
2. Relevance: Every step directly contributes to the goal
3. Correctness: Steps are logically ordered with valid dependencies
4. Expertise Alignment: Each agent is used within its capabilities
5. Efficiency: No redundant steps, minimal complexity
6. Clarity: Tasks are specific, unambiguous, and actionable

🎯 ANTI-PATTERNS TO AVOID:
❌ Abstract tasks like "analyze data" without specifics
❌ Missing dependencies between related steps
❌ Using wrong agent for a task
❌ Vague expected outputs like "results"
❌ Including information not in the original problem
❌ Skipping necessary intermediate steps

✅ GOOD PATTERNS:
✓ Tasks reference specific assets/sensors from the problem
✓ Each step has concrete, measurable output
✓ Dependencies are explicitly stated
✓ Agent selection matches task requirements
✓ Step sequence follows logical flow

{agent_descriptions}

{task_description}

Now create a plan that follows all good patterns and avoids all anti-patterns:
"""
```

**Benefits:**
- Incorporates evaluation criteria into generation
- Reduces invalid plans
- Better first-attempt quality

---

## Implementation Strategies

### Strategy 1: Incremental Optimization

1. **Baseline:** Test current implementation
2. **Phase 1:** Improve agent descriptions (Section 2)
3. **Phase 2:** Enhance prompt template (Section 4)
4. **Phase 3:** Add robust parsing (Section 3)
5. **Phase 4:** Add helpers and context detection (Section 1)

### Strategy 2: A/B Testing Approach

Test variations:
- **Variant A:** Structured formatting + CoT prompt
- **Variant B:** Few-shot examples + entity extraction
- **Variant C:** Decision tree guidance + self-critique
- **Variant D:** Combination of best performing elements

### Strategy 3: Token Optimization

Balance between:
- **Detailed instructions** (better guidance) vs.
- **Token efficiency** (cost and context limits)

Techniques:
- Truncate task examples to first 3-5
- Use abbreviations in formatting
- Consolidate redundant instructions
- Remove unnecessary emojis (if not helpful)

---

## Examples & Best Practices

### Example 1: Complete Optimized Implementation

```python
# Section 1: Variable Declarations (Lines 16-19)
# Agent capability mapping for better selection
AGENT_KEYWORDS = {
    "IoT Data Download": ["sensor", "asset", "data", "download", "historical", "site"],
    "Failure Mode and Sensor Relevancy Expert": ["failure", "mode", "mapping", "sensor mapping"],
    "Time-Series Foundation Model": ["forecast", "anomaly", "predict", "time series", "trend"],
    "Work Order Generator": ["work order", "maintenance", "repair", "schedule"]
}

def extract_key_info(description):
    """Extract asset names and other key info"""
    import re
    assets = re.findall(r'\b(Chiller|Pump|Boiler|CU|HVAC)[-\s]?\w*\d+\b', description, re.IGNORECASE)
    return list(set(assets))

# Section 2: Agent Description Formatting (Lines 64-89)
for ii, aagent in enumerate(task.agents):
    agent_descriptions += f"\n{'─'*50}\n"
    agent_descriptions += f"Agent #{ii + 1}: **{aagent.name}**\n"
    agent_descriptions += f"Role: {aagent.description}\n"

    # Add capability keywords
    keywords = AGENT_KEYWORDS.get(aagent.name, [])
    if keywords:
        agent_descriptions += f"Keywords: {', '.join(keywords[:5])}\n"

    # Show concise examples
    if "task_examples" in aagent.__dict__ and aagent.task_examples:
        agent_descriptions += f"Example Tasks:\n"
        for idx, ex in enumerate(aagent.task_examples[:3], start=1):
            short_ex = ex[:80] + "..." if len(ex) > 80 else ex
            agent_descriptions += f"  {idx}. {short_ex}\n"
    agent_descriptions += "\n"

# Section 3: Response Post-Processing (Lines 104-174)
# (Keep existing parsing but add validation)
self.memory = []

# Enhanced regex patterns
task_pattern = r"#Task\d+:\s*(.+?)(?=\n#Agent|\Z)"
agent_pattern = r"#Agent\d+:\s*(.+?)(?=\n#Dependency|\Z)"
dependency_pattern = r"#Dependency\d+:\s*(.+?)(?=\n#ExpectedOutput|\Z)"
output_pattern = r"#ExpectedOutput\d+:\s*(.+?)(?=\n##|\n#Task|\Z)"

tasks = [t.strip() for t in re.findall(task_pattern, final_plan, re.DOTALL)]
agents = [a.strip() for a in re.findall(agent_pattern, final_plan, re.DOTALL)]
dependencies = [d.strip() for d in re.findall(dependency_pattern, final_plan, re.DOTALL)]
outputs = [o.strip() for o in re.findall(output_pattern, final_plan, re.DOTALL)]

# ... rest of existing parsing logic ...

# Section 4: Prompt Template (Lines 180-214)
def get_prompt(self, task_description, agent_descriptions):
    # Extract key entities
    key_assets = extract_key_info(task_description)
    asset_reminder = ""
    if key_assets:
        asset_reminder = f"\n⚠️ Your plan MUST reference these assets: {', '.join(key_assets)}\n"

    prompt = f"""You are an expert AI planner for industrial asset operations.

🎯 MISSION: Create a step-by-step plan using ONLY the available agents below.

📋 PLANNING RULES:
1. Each step has: Task, Agent, Dependency, ExpectedOutput
2. Tasks must be SPECIFIC with ALL details (asset names, sensors, time ranges)
3. Use exact agent names from the list
4. Keep plan under 5 steps
5. State dependencies explicitly (#S1, #S2, etc. or None)

🔍 AGENT SELECTION GUIDE:
- Need data about assets/sensors? → IoT Data Download
- Need failure mode info? → Failure Mode Expert
- Need forecasting/anomaly detection? → Time-Series Model
- Need to create work orders? → Work Order Generator

📝 OUTPUT FORMAT:
## Step 1
#Task1: [Specific task with all necessary details]
#Agent1: [Exact agent name]
#Dependency1: None
#ExpectedOutput1: [Concrete, measurable output]

## Step 2
#Task2: [Next specific task]
#Agent2: [Exact agent name]
#Dependency2: #S1
#ExpectedOutput2: [Concrete output]

[Continue as needed, but keep under 5 steps total]

## AVAILABLE AGENTS ##
{agent_descriptions}

## PROBLEM TO SOLVE ##
{task_description}
{asset_reminder}

Generate plan (plan only, no extra text):
"""
    return prompt
```

### Example 2: Minimal but Effective Changes

If you want minimal changes with maximum impact:

```python
# Just modify Section 4: Prompt
prompt = f"""
You are an AI task planner. Break down the problem into clear steps.

CRITICAL REQUIREMENTS:
- Use ONLY these agents (exact names): {', '.join([a.name for a in task.agents])}
- Include ALL specific details from the problem (asset names, sensors, time ranges, file names)
- Maximum 5 steps
- Each step must have: #TaskN, #AgentN, #DependencyN, #ExpectedOutputN
- Dependencies: Use #S1, #S2, etc. to reference previous step outputs, or "None"

AVAILABLE AGENTS:
{agent_descriptions}

PROBLEM:
{task_description}

Create your plan in this exact format:
## Step 1
#Task1: <detailed task>
#Agent1: <exact agent name>
#Dependency1: None
#ExpectedOutput1: <specific output>

OUTPUT YOUR PLAN NOW:
"""
```

---

## Common Pitfalls to Avoid

### ❌ Pitfall 1: Over-Engineering
**Problem:** Adding too much complexity that confuses the LLM
**Solution:** Test incrementally, keep what works

### ❌ Pitfall 2: Token Bloat
**Problem:** Prompt becomes too long, hitting context limits
**Solution:** Truncate examples, use concise language

### ❌ Pitfall 3: Format Breaking
**Problem:** LLM generates plan in different format
**Solution:** Make format requirements very explicit, provide examples

### ❌ Pitfall 4: Inconsistent Agent Names
**Problem:** LLM uses similar but not exact agent names
**Solution:** List exact names explicitly, add fuzzy matching in parsing

### ❌ Pitfall 5: Vague Task Descriptions in Plan
**Problem:** Generated tasks lack specific details
**Solution:** Emphasize specificity in prompt, provide concrete examples

### ❌ Pitfall 6: Ignoring Constraints
**Problem:** Generated plans exceed 5 steps or skip dependencies
**Solution:** Repeat constraints multiple times in prompt

### ❌ Pitfall 7: Not Testing Enough
**Problem:** Changes work on some scenarios but break others
**Solution:** Test on diverse scenarios before submission

---

## Testing & Validation

### Local Testing Process

1. **Setup Docker Environment**
```bash
cd /home/user/AssetOpsBench
docker-compose -f benchmark/cods_track1/docker-compose.yml up
```

2. **Test on Sample Scenarios**
```python
# Test with different scenario types:
- Simple data retrieval: "List all sensors for Chiller 6"
- Multi-step analysis: "Get Chiller 9 data and detect anomalies"
- End-to-end workflow: "Analyze failures and create work order"
```

3. **Validate Plan Quality**
Check generated plans for:
- ✅ Correct format (#Task, #Agent, #Dependency, #ExpectedOutput)
- ✅ Exact agent names
- ✅ Specific details from problem included
- ✅ Logical step sequence
- ✅ Valid dependencies
- ✅ Under 5 steps

4. **Monitor Logs**
```python
logger.info(f"Plan Generation Prompt: \n{prompt}")
logger.info(f"Plan: \n{llm_response}")
logger.info(f"Planned Tasks: \n{planned_tasks}")
```

### Debugging Tips

```python
# Add debug logging in Section 3
logger.info(f"Extracted tasks: {tasks}")
logger.info(f"Extracted agents: {agents}")
logger.info(f"Extracted dependencies: {dependencies}")
logger.info(f"Extracted outputs: {outputs}")
```

---

## Recommended Implementation Order

### Phase 1: Quick Wins (1-2 hours)
1. Improve prompt clarity (Section 4)
2. Add explicit agent name list
3. Emphasize specificity requirements
4. Test on 3-5 scenarios

### Phase 2: Structure Improvements (2-3 hours)
1. Enhance agent description formatting (Section 2)
2. Add capability keywords
3. Truncate verbose examples
4. Test on 10+ scenarios

### Phase 3: Advanced Features (3-4 hours)
1. Add entity extraction (Section 1)
2. Implement robust parsing (Section 3)
3. Add problem type detection
4. Comprehensive testing

### Phase 4: Optimization (2-3 hours)
1. Token optimization
2. A/B testing variations
3. Fine-tuning based on results
4. Final validation

---

## Key Takeaways

### ✅ DO:
- Be explicit and specific in instructions
- Provide concrete examples
- Emphasize constraints multiple times
- Test on diverse scenarios
- Use structured formatting
- Extract and reference entities from the problem
- Keep token count reasonable

### ❌ DON'T:
- Change workflow execution logic
- Modify Task object structure
- Add imports (not allowed)
- Over-complicate the prompt
- Forget to test thoroughly
- Ignore the evaluation criteria
- Use placeholder values in examples

---

## Conclusion

The key to optimizing `track1_planning.py` lies in:

1. **Clear Communication:** Making instructions unambiguous for the LLM
2. **Structured Guidance:** Providing decision frameworks and examples
3. **Entity Grounding:** Ensuring plans reference specific problem details
4. **Format Enforcement:** Making output requirements very explicit
5. **Robust Parsing:** Handling variations in LLM outputs gracefully

Start with simple improvements to the prompt template and agent descriptions, test thoroughly, then incrementally add more sophisticated features based on what works best for your scenarios.

Good luck with the competition! 🚀
