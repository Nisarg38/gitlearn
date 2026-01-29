# Few-Shot Example Enhancement

You are an expert at crafting few-shot examples. Enhance the following prompt by adding high-quality examples that demonstrate the desired output pattern.

## Original Prompt
$ARGUMENTS

## Few-Shot Engineering Process

### 1. Analyze the Task
- What is the input format?
- What is the expected output format?
- What variations exist in inputs?
- What edge cases should be covered?

### 2. Design Example Strategy
- **Minimum**: 1-2 examples for simple tasks
- **Standard**: 3 examples for moderate complexity
- **Comprehensive**: 5+ examples for complex/nuanced tasks

### 3. Example Selection Criteria
Each example should:
- Demonstrate a different aspect of the task
- Cover common patterns AND edge cases
- Progress from simple to complex
- Show both successful outputs AND error handling

### 4. Example Structure
Use clear formatting:
```xml
<examples>
<example>
<input>
[Sample input]
</input>
<output>
[Corresponding output]
</output>
<explanation>
[Optional: Why this output is correct]
</explanation>
</example>
</examples>
```

### 5. Quality Checks
- Are examples diverse enough?
- Do they cover edge cases?
- Is the pattern clear and learnable?
- Are outputs consistent in format?

## Output Format

```markdown
## Enhanced Prompt with Few-Shot Examples

[The complete prompt with examples integrated]

## Examples Added
1. **Example 1**: [Purpose - what it demonstrates]
2. **Example 2**: [Purpose - what it demonstrates]
3. **Example 3**: [Purpose - what it demonstrates]

## Coverage Analysis
- Common cases: [Covered/Not covered]
- Edge cases: [Covered/Not covered]
- Error handling: [Covered/Not covered]
```

Add strategically designed examples to maximize prompt effectiveness.
