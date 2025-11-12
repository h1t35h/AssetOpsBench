# Track 1 Planning - Quick Optimization Reference

## 🎯 Goal
Improve task planning quality through better prompt engineering in `track1_planning.py`

## 📊 Evaluation Metrics
Plans are scored on:
1. ✅ **Completeness** - All necessary steps included
2. ✅ **Relevance** - Each step contributes to the goal
3. ✅ **Correctness** - Logical sequence, valid dependencies
4. ✅ **Expertise Alignment** - Right agent for each task
5. ✅ **Efficiency** - No redundant steps
6. ✅ **Clarity** - Clear, specific, actionable tasks

## 🔧 Editable Sections

| Section | Lines | Purpose | Key Opportunities |
|---------|-------|---------|-------------------|
| **Variables** | 16-19 | Helper code | Add utility functions, entity extraction, agent mappings |
| **Agent Descriptions** | 64-89 | Format agent info | Improve hierarchy, add keywords, categorize capabilities |
| **Post-Processing** | 104-174 | Parse LLM response | Robust regex, fuzzy agent matching, validation |
| **Prompt Template** | 180-214 | Guide LLM | CoT prompting, examples, constraints, decision trees |

## 🚀 Top 5 Quick Wins

### 1. Enhanced Prompt Clarity
```python
prompt = f"""
CRITICAL: Use EXACT agent names: {', '.join([a.name for a in task.agents])}
CRITICAL: Include ALL specific details (asset names, sensors, time ranges)
CRITICAL: Maximum 5 steps
CRITICAL: Format: #Task1, #Agent1, #Dependency1, #ExpectedOutput1

{agent_descriptions}
{task_description}

Generate plan now:
"""
```

### 2. Better Agent Descriptions
```python
for ii, aagent in enumerate(task.agents):
    agent_descriptions += f"\n{'='*50}\n"
    agent_descriptions += f"🤖 Agent {ii+1}: {aagent.name}\n"
    agent_descriptions += f"Role: {aagent.description}\n"
    if aagent.task_examples:
        agent_descriptions += "Examples:\n"
        for idx, ex in enumerate(aagent.task_examples[:3], 1):
            agent_descriptions += f"  {idx}. {ex[:80]}...\n"
```

### 3. Entity Extraction
```python
# In Section 1
import re
def extract_entities(desc):
    return re.findall(r'\b(Chiller|Pump|Boiler|CU)\s*\d+\b', desc, re.IGNORECASE)

# In Section 4
entities = extract_entities(task_description)
if entities:
    prompt += f"\n⚠️ MUST reference: {', '.join(entities)}\n"
```

### 4. Decision Tree for Agent Selection
```python
prompt = f"""
AGENT SELECTION GUIDE:
- Need asset/sensor data? → IoT Data Download
- Need failure mode info? → Failure Mode Expert
- Need forecasting/anomalies? → Time-Series Model
- Need work orders? → Work Order Generator
...
"""
```

### 5. Few-Shot Examples
```python
# In Section 1
EXAMPLE = """
EXAMPLE PLAN:
Problem: "Get Chiller 6 data and detect anomalies"

## Step 1
#Task1: Download historical sensor data for Chiller 6...
#Agent1: IoT Data Download
#Dependency1: None
#ExpectedOutput1: JSON file with sensor readings
...
"""

# Add to prompt
prompt = f"Study this example:\n{EXAMPLE}\n\nNow solve:\n{task_description}"
```

## 🎨 Optimization Strategies

### Strategy A: Minimal Changes (1-2 hours)
1. ✏️ Clarify prompt instructions (Section 4)
2. ✏️ List exact agent names explicitly
3. ✏️ Emphasize specificity requirements
4. ✅ Test on 5 scenarios

**Expected Impact:** 10-15% improvement

### Strategy B: Structured Approach (3-4 hours)
1. ✏️ Add entity extraction (Section 1)
2. ✏️ Improve agent descriptions (Section 2)
3. ✏️ Enhance prompt with CoT (Section 4)
4. ✅ Test on 15 scenarios

**Expected Impact:** 20-30% improvement

### Strategy C: Advanced (6-8 hours)
1. ✏️ Full entity extraction and problem typing (Section 1)
2. ✏️ Structured agent descriptions with keywords (Section 2)
3. ✏️ Robust parsing with fuzzy matching (Section 3)
4. ✏️ Few-shot + CoT + decision tree prompt (Section 4)
5. ✅ Comprehensive testing

**Expected Impact:** 30-50% improvement

## ⚠️ Common Mistakes to Avoid

| ❌ Don't | ✅ Do |
|---------|-------|
| Vague tasks like "analyze data" | Specific tasks: "Analyze temperature sensor data from Chiller 6 for anomalies in the last 30 days" |
| Miss entity details | Extract and validate all assets/sensors mentioned |
| Break format | Make format requirements crystal clear |
| Use approximate agent names | List exact agent names explicitly |
| Create verbose prompts | Balance detail with token efficiency |
| Skip testing | Test on diverse scenarios |
| Ignore dependencies | Explicitly state step dependencies |

## 🧪 Testing Checklist

```bash
# 1. Run Docker
cd /home/user/AssetOpsBench
docker-compose -f benchmark/cods_track1/docker-compose.yml up

# 2. Check generated plans for:
□ Correct format (#Task, #Agent, #Dependency, #ExpectedOutput)
□ Exact agent names used
□ Specific details from problem included
□ Logical step sequence
□ Valid dependencies (#S1, #S2, etc.)
□ Under 5 steps
□ No hallucinated information

# 3. Test scenario types:
□ Simple queries (data retrieval)
□ Analysis tasks (anomaly detection, forecasting)
□ Multi-step workflows (data → analysis → action)
□ Edge cases (multiple assets, long time ranges)
```

## 📦 Submission Checklist

```bash
# 1. Test locally
docker-compose -f benchmark/cods_track1/docker-compose.yml up

# 2. Create submission
cd src/agent_hive/workflows
zip submission_track1.zip track1_planning.py track1_fact_sheet.json

# 3. Verify contents
unzip -l submission_track1.zip

# 4. Submit to CodaBench
# → https://www.codabench.org/competitions/10206
# → My Submissions tab
# → Upload ZIP
```

## 💡 Pro Tips

1. **Start Simple:** Test basic improvements before adding complexity
2. **Measure Impact:** Compare results with baseline
3. **Iterate:** Make incremental changes based on what works
4. **Balance Tokens:** More detail ≠ always better
5. **Test Edge Cases:** Don't just test on easy scenarios
6. **Use Logging:** Add debug statements to understand LLM outputs
7. **Study Examples:** Review few-shot examples in agent definitions
8. **Check Dependencies:** Ensure step sequences make logical sense
9. **Validate Entities:** Make sure plans reference problem specifics
10. **Time Management:** Leave time for testing and iteration

## 📚 Key Files Reference

```
src/agent_hive/
├── workflows/
│   ├── track1_planning.py          # Your file to edit
│   ├── planning.py                 # Baseline implementation
│   └── planning_review.py          # Advanced version with review loop
├── agents/
│   ├── plan_reviewer_agent.py      # Plan evaluation logic
│   └── plan_reviewer_prompt.py     # Evaluation criteria
└── tools/
    ├── skyspark.py                 # IoT agent definition
    ├── fmsr.py                     # FMSR agent definition
    ├── tsfm.py                     # TSFM agent definition
    └── wo.py                       # Work Order agent definition
```

## 🎯 Success Criteria

Your plan is good if:
- ✅ Parses correctly (format followed)
- ✅ Uses correct agents for each task
- ✅ Includes all specific details from problem
- ✅ Has logical step sequence
- ✅ States dependencies correctly
- ✅ Has concrete, measurable outputs
- ✅ Completes under 5 steps
- ✅ No hallucinated information
- ✅ Results in successful task execution

Good luck! 🚀

For detailed analysis, see: `TRACK1_PLANNING_OPTIMIZATION_ANALYSIS.md`
