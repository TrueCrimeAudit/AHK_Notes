# Research Task Template

## Task Information
**Task Name:** AHK_Timers_Deep_Dive  
**Created:** 2025-06-18  
**Status:** Planning

## Core Objective
<task>
Research and produce a detailed report explaining AutoHotkey v2 timers. This should cover the complete lifecycle of timer usage, from basic implementation to advanced patterns and best practices. The goal is to create comprehensive documentation that can serve as the definitive reference for timer usage in my AHK knowledge base.
</task>

## Research Plan / Key Questions

1. Explain the basic syntax and parameters of SetTimer, including all available options and their effects
2. Detail the difference between a one-time timer (negative period) and a recurring timer, with practical use cases for each
3. Explain how to use .Bind() to link a timer to a class method, with a complete working code example
4. Discuss best practices for timer cleanup to avoid resource leaks and memory issues
5. Cover advanced timer patterns like conditional timers, timer chaining, and timer state management
6. Provide guidance on debugging timer-related issues and common pitfalls to avoid

## Source Quality Rules

**Default Guidelines:**
- Prioritize primary sources (official documentation, API references, original authors)
- Be skeptical of marketing language and promotional content
- Prefer recent sources unless historical context is specifically needed
- Cross-reference claims with multiple authoritative sources
- Flag any conflicting information found across sources

**Custom Guidelines for this task:**
- Focus specifically on AutoHotkey v2 (not v1) documentation and examples
- Prioritize official AHK documentation and well-maintained community resources
- Look for real-world examples and common usage patterns
- Verify that code examples are syntactically correct for v2

## Final Report Format

**Structure Requirements:**
- Use H2 headers for each key question from the research plan
- Include concrete code examples where applicable
- Provide concise summaries at the end of each section
- Flag any areas where information was incomplete or conflicting

**Specific Format Notes:**
- Each code example should be complete and runnable
- Include comments in code explaining key concepts
- Use consistent variable naming and code style
- Provide both simple and advanced examples where appropriate
- Include performance considerations where relevant

## Integration Notes

**Target Integration:**
- [x] Will create new documentation in: AHK_Notes/Methods/Timers.md
- [x] Will influence coding patterns in: Event handling and automation scripts
- [ ] Will update existing documentation in: [none currently]
- [ ] Other: May reference from GUI and automation pattern documentation

---

## Execution Checklist
- [x] Template filled out completely
- [x] Research scope is clear and bounded
- [x] Questions are specific and actionable
- [x] Source quality rules are appropriate
- [x] Report format matches intended use
- [x] Integration plan is defined

---

## Full Prompt for Claude

```
Your task is to research and produce a detailed report on AutoHotkey v2 timers. This should cover the complete lifecycle of timer usage, from basic implementation to advanced patterns and best practices.

Please follow this research plan:

1. Explain the basic syntax and parameters of SetTimer, including all available options and their effects
2. Detail the difference between a one-time timer (negative period) and a recurring timer, with practical use cases for each
3. Explain how to use .Bind() to link a timer to a class method, with a complete working code example
4. Discuss best practices for timer cleanup to avoid resource leaks and memory issues
5. Cover advanced timer patterns like conditional timers, timer chaining, and timer state management
6. Provide guidance on debugging timer-related issues and common pitfalls to avoid

Source Quality Guidelines:
- Focus specifically on AutoHotkey v2 (not v1) documentation and examples
- Prioritize official AHK documentation and well-maintained community resources
- Look for real-world examples and common usage patterns
- Verify that code examples are syntactically correct for v2
- Cross-reference claims with multiple authoritative sources
- Flag any conflicting information found across sources

Final Report Requirements:
- Use H2 headers for each key question from the research plan
- Each code example should be complete and runnable
- Include comments in code explaining key concepts
- Use consistent variable naming and code style
- Provide both simple and advanced examples where appropriate
- Include performance considerations where relevant
- Provide concise summaries at the end of each section
- Flag any areas where information was incomplete or conflicting

Please provide a comprehensive, well-structured response that addresses each point in the research plan with the specified format and source quality standards.
```