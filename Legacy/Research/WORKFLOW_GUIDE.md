# Research Delegation Workflow

## Overview
This system transforms Claude Desktop into a systematic research assistant for building your AutoHotkey knowledge base. Instead of ad-hoc questions, you delegate well-structured research tasks that produce reusable, high-quality documentation.

## Step-by-Step Workflow

### 1. Initiate New Research Task
When you have a research need:
- Copy `/Research/Templates/research_prompt_template.md`
- Create a new folder: `/Research/[Task_Name]/`
- Place the copied template in the new folder as `research_template.md`

### 2. Fill Out the Template
Complete each section thoughtfully:
- **Task Name**: Short, descriptive (becomes folder name)
- **Core Objective**: What exactly do you need to understand?
- **Research Plan**: Specific, numbered questions
- **Source Quality Rules**: Customize for this task's needs
- **Final Report Format**: How should results be structured?
- **Integration Notes**: Where will this knowledge go?

### 3. Execute Research
- Assemble the final prompt from your template
- Send to Claude in a dedicated conversation
- Save the full prompt to `_prompt_sent.md`
- Save Claude's response to `response.md`

### 4. Review and Refine
- Read through `response.md`
- Add your notes and insights to `my_notes.md`
- Identify gaps or follow-up questions
- Create additional research tasks if needed

### 5. Integrate Knowledge
- Move polished content from Research to main knowledge base
- Update relevant files in `/Concepts/`, `/Methods/`, `/Patterns/`
- Archive completed research or keep for reference

## Folder Structure Example

```
AHK_Notes/
├── Research/
│   ├── Templates/
│   │   └── research_prompt_template.md
│   ├── AHK_Timers_Deep_Dive/
│   │   ├── research_template.md
│   │   ├── _prompt_sent.md
│   │   ├── response.md
│   │   └── my_notes.md
│   ├── GUI_Event_Handling/
│   │   ├── research_template.md
│   │   ├── _prompt_sent.md
│   │   ├── response.md
│   │   └── my_notes.md
│   └── Completed/
│       └── [archived research tasks]
├── Concepts/
├── Methods/
├── Patterns/
└── Templates/
```

## Benefits of This Approach

### Quality & Consistency
- Template forces thorough planning
- Consistently structured results
- Repeatable process for complex topics

### Knowledge Organization
- Research stays separate from production knowledge
- Easy to track what's been researched
- Clear integration path to main knowledge base

### Reusable Assets
- Each research task produces valuable documentation
- Templates can be adapted for different question types
- Research history becomes searchable resource

## Tips for Effective Research Tasks

### Good Task Examples
- "Deep dive into AHK v2 timer methods and best practices"
- "Comprehensive guide to GUI event handling patterns"
- "Class inheritance strategies with practical examples"

### Poor Task Examples  
- "Tell me about AHK" (too broad)
- "Fix this code" (not research)
- "Quick question about timers" (use regular chat)

### Research Plan Guidelines
- Ask 3-6 specific questions per task
- Progress from basic concepts to advanced applications
- Include request for code examples
- Ask for best practices and common pitfalls

### Source Quality Notes
- AutoHotkey documentation and forums are primary sources
- GitHub repositories with good examples
- Be wary of outdated v1 information when researching v2
- Cross-reference complex topics across multiple sources

## Advanced Usage

### Research Chains
Break complex topics into multiple linked research tasks:
1. Basic concepts and syntax
2. Advanced patterns and techniques  
3. Integration with other systems
4. Performance and optimization

### Template Variants
Create specialized templates for different research types:
- `concept_research_template.md` - For understanding new concepts
- `practical_guide_template.md` - For how-to guides and tutorials
- `comparison_research_template.md` - For comparing different approaches

### Integration Workflows
- Weekly review of completed research
- Monthly integration sessions to update main knowledge base
- Quarterly archive of old research to keep system clean