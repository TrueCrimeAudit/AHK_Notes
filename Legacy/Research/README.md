# AHK Research Delegation System

## Overview
This system transforms Claude Desktop into a systematic research assistant for building your AutoHotkey knowledge base. Instead of scattered questions, you create structured research tasks that produce high-quality, reusable documentation.

## Quick Start

### 1. Create New Research Task
```
cp Research/Templates/research_prompt_template.md Research/[New_Task_Name]/research_template.md
```

### 2. Fill Out Template
Edit the research_template.md with:
- Clear objective
- Specific numbered questions  
- Source quality requirements
- Desired output format

### 3. Execute Research
- Copy the assembled prompt from template
- Send to Claude in new conversation
- Save prompt to `_prompt_sent.md`
- Save response to `response.md`
- Add your insights to `my_notes.md`

## Example Folder Structure
```
Research/
├── Templates/
│   └── research_prompt_template.md    # Master template
├── AHK_Timers_Deep_Dive/              # Example completed research
│   ├── research_template.md           # Filled template
│   ├── _prompt_sent.md               # Actual prompt sent
│   ├── response.md                   # Claude's research response
│   └── my_notes.md                   # Your analysis & insights
└── WORKFLOW_GUIDE.md                 # Detailed workflow instructions
```

## Key Benefits

✅ **Enforces Quality**: Template structure ensures thorough, well-planned research  
✅ **Creates Reusable Assets**: Each research task produces valuable documentation  
✅ **Separates Research from Production**: Research folder is staging area  
✅ **Reduces Cognitive Load**: Template handles prompt structure  
✅ **Builds Knowledge Systematically**: Structured approach to learning

## Research Types

- **Concept Research**: Understanding new AHK concepts deeply
- **Practical Guides**: Step-by-step how-to documentation  
- **Pattern Analysis**: Comparing different coding approaches
- **Feature Deep-Dives**: Comprehensive coverage of specific features

## Integration Workflow

1. **Research Phase**: Use this system to gather comprehensive information
2. **Review Phase**: Analyze results, identify gaps, create follow-up research
3. **Integration Phase**: Move polished content to main AHK_Notes structure
4. **Reference Phase**: Use research archive for future projects

## Tips for Success

- **Be Specific**: Ask concrete, numbered questions rather than broad topics
- **Plan Integration**: Know where knowledge will go in your main system
- **Review Systematically**: Schedule regular review of completed research
- **Chain Research**: Complex topics often need multiple research tasks

This system has transformed one-off questions into a systematic knowledge-building process. Each research task becomes a valuable asset in your AutoHotkey learning journey.