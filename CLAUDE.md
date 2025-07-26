# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a comprehensive research repository focused on AI's impact on various development practices and methodologies. The repository contains detailed research reports, practical methodologies, and AI-enhanced toolkits across multiple domains, representing an evolution from theoretical research to practical implementation frameworks.

## Repository Structure

### Main Research Domains

- **EXPLORE_DEVOPS/**: Traditional DevOps vs AI-enhanced DevOps practices
- **EXPLORE_PMBA/**: AI transformation in project management and business analysis  
- **EXPLORE_RND/**: AI's impact on research and development processes
- **EXPLORE_TDD/**: Classic TDD vs AI-enhanced TDD methodologies
- **EXPLORE_towards_TestAsset_v*/**: Evolution of AI testing frameworks and practical toolkits

### Structure Evolution

The repository shows clear evolution from exploratory research to practical implementation:

1. **Early Exploration** (`EXPLORE_*` directories): Raw research and comparative analysis
2. **Practical Framework Development** (`EXPLORE_towards_TestAsset_v*`): Implementation-ready methodologies
3. **Mature Toolkit** (`EXPLORE_towards_TestAsset_v2/`): Complete spec-driven development system with AI testing toolkit

### Directory Patterns

Each exploration directory typically contains:
- `research_dr*/`: Raw research data organized by AI model source (deepseek, gemini, openai, kimi, mita, perplexity, grok)
- `result/`: Final reports and structured outputs, often with `parts/` subdirectories for modular content
- `result_ppt/`: Presentation-ready slide formats
- `_extra/`: Additional supporting materials and table of contents

### Key Practical Resources

The most mature and actionable content is in `EXPLORE_towards_TestAsset_v2/`:
- **spec_driven_development/**: Complete methodology for specification-driven development with 4-stage workflow
- **ai_testing_toolkit/**: Professional AI testing tools with conversation examples and advanced templates

## File Organization Conventions

### Research Files
- Raw research files prefixed with `raw_dr_` followed by domain and AI model
- Final reports in both markdown (`.md`) and PDF (`.pdf`) formats
- Modular content in `parts/` subdirectories for easy navigation

### Content Types
- **Methodology**: Step-by-step processes and frameworks
- **Templates**: Ready-to-use document templates and checklists  
- **Examples**: Complete project examples with requirements, design, tasks, and execution logs
- **Guides**: Role-specific guidance for different team members

## Working with This Repository

### No Traditional Development Environment
- No package.json, Makefile, or build scripts
- No linting or testing commands configured
- Content is primarily documentation that can be read and edited directly

### Content Guidelines
- Maintain consistent directory structure and naming conventions
- Preserve the pattern of organizing raw research by AI model source
- Keep reports in both markdown and PDF formats where applicable
- Follow the modular approach with `parts/` subdirectories for complex documents
- Maintain separation between raw research data and processed results

### Key Navigation Points
- Start with `README.md` files in each directory for orientation
- Use `00_quick_start/` directories for immediate practical application
- Reference `_extra/` directories for supporting materials and table of contents
- Focus on `EXPLORE_towards_TestAsset_v2/` for the most mature practical frameworks

### Quality Standards
- Documents include clear role-based guidance (product managers, developers, testers, etc.)
- Templates and examples are immediately actionable
- Content follows progressive disclosure from quick-start to advanced practices
- AI collaboration patterns and prompts are extensively documented

This repository represents a comprehensive journey from AI impact research to practical implementation frameworks, with the latest versions providing immediately usable methodologies and tools for modern development practices.