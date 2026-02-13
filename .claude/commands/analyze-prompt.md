# Analyze Prompt

You are an expert prompt analyst. Perform a comprehensive analysis of the following prompt to identify weaknesses, gaps, and improvement opportunities.

## Prompt to Analyze
$ARGUMENTS

## Analysis Framework

### 1. Completeness Score (0-10)
Evaluate against Anthropic's 10-component framework:
- [ ] Task Context - Is Claude's role defined?
- [ ] Tone Context - Is communication style specified?
- [ ] Background Data - Is relevant context provided?
- [ ] Detailed Task Description - Are requirements explicit?
- [ ] Examples - Are desired outputs demonstrated?
- [ ] Conversation History - Is prior context included if needed?
- [ ] Immediate Task - Is the current deliverable clear?
- [ ] Step-by-Step Thinking - Is reasoning encouraged?
- [ ] Output Formatting - Is structure defined?
- [ ] Prefilled Response - Is style guidance provided?

### 2. Clarity Assessment
- Ambiguous terms or phrases
- Missing definitions
- Unclear success criteria
- Scope boundaries

### 3. Potential Failure Modes
- Edge cases not handled
- Assumptions that could break
- Missing error handling guidance
- Potential misinterpretations

### 4. Efficiency Analysis
- Token waste from redundancy
- Opportunities for compression
- Unnecessary complexity

### 5. Best Practice Gaps
- Missing XML structure for complex sections
- No chain-of-thought for complex reasoning
- Absent few-shot examples
- Undefined output format

## Output Format

```markdown
## Prompt Analysis Report

### Overall Score: X/10

### Strengths
- [What the prompt does well]

### Critical Issues
- [Must-fix problems that will cause failures]

### Improvement Opportunities
- [Nice-to-have enhancements]

### Specific Recommendations
1. [Actionable recommendation]
2. [Actionable recommendation]
3. [Actionable recommendation]

### Risk Assessment
- Failure Probability: Low/Medium/High
- Most Likely Failure Mode: [Description]
```

Analyze the prompt thoroughly and provide actionable insights.
