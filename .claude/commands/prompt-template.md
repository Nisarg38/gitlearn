# Prompt Template Generator

You are an expert prompt architect. Generate a production-ready prompt template based on the following requirements.

## Requirements
$ARGUMENTS

## Template Generation Process

### 1. Task Classification
Identify the task type:
- **Generation**: Creating new content
- **Analysis**: Examining existing content
- **Transformation**: Converting formats
- **Classification**: Categorizing inputs
- **Extraction**: Pulling specific information
- **Reasoning**: Logical problem-solving

### 2. Select Template Structure
Based on task complexity, use:

**Minimal Template** (simple tasks):
```xml
<task>[Clear task statement]</task>
<input>{user_input}</input>
<output_format>[Expected format]</output_format>
```

**Standard Template** (moderate tasks):
```xml
<context>[Background information]</context>
<task>[Detailed task description]</task>
<constraints>
- [Constraint 1]
- [Constraint 2]
</constraints>
<input>{user_input}</input>
<output_format>[Detailed format specification]</output_format>
```

**Comprehensive Template** (complex tasks):
```xml
<role>[Expert persona]</role>
<context>[Detailed background]</context>
<task>
[Multi-part task description]
</task>
<constraints>
- [Hard constraints]
- [Soft preferences]
</constraints>
<examples>
[Few-shot examples]
</examples>
<thinking>
[Chain-of-thought guidance]
</thinking>
<input>{user_input}</input>
<output_format>
[Detailed format with schema]
</output_format>
<quality_criteria>
[Success metrics]
</quality_criteria>
```

### 3. Add Task-Specific Elements
- **For code tasks**: Add language, style guides, testing requirements
- **For analysis**: Add evaluation criteria, depth requirements
- **For generation**: Add tone, length, audience specifications

## Output Format

```markdown
## Generated Prompt Template

[Complete, ready-to-use template with placeholders marked as {placeholder}]

## Template Type
- Complexity Level: Minimal/Standard/Comprehensive
- Task Type: [Classification]

## Placeholders
- `{placeholder_name}`: [Description of what to insert]

## Usage Instructions
1. [How to fill in placeholders]
2. [When to use this template]
3. [Customization options]

## Example Usage
[One filled-in example showing the template in action]
```

Generate a professional, reusable prompt template tailored to the requirements.
