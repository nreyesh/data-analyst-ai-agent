---
name: doc-coauthoring
description: Guide users through a structured workflow for co-authoring documentation, technical specifications, architecture audits, and decision docs. This workflow helps users efficiently transfer context, refine content through iteration, and verify the doc works for readers. Trigger when user mentions writing docs, creating proposals, drafting specs, auditing codebases, or similar documentation tasks.
---

# Doc Co-Authoring Skill

## Overview
This skill guides users through a structured, iterative co-authoring process for complex technical documentation, architecture specifications, system audits, and PRDs. Rather than generating a document in one unguided pass, this workflow enforces alignment, structure validation, and reader testing.

## Three-Stage Workflow

### Stage 1: Context Gathering & Scoping
- **Meta-Questions**: Clarify the document type, target audience, intended impact, technical constraints, and depth.
- **Context Ingestion**: Absorb codebase files, architectural contracts, data models, workflows, and test suites.
- **Clarifying Questions**: Surface architectural trade-offs, critical design patterns, or ambiguities before drafting.

### Stage 2: Refinement & Structure
- **Outline Proposal**: Present a comprehensive Table of Contents / section roadmap for user approval.
- **Iterative Drafting**: Draft section-by-section or modularly, allowing the user to provide feedback, steer focus, and refine details.
- **Deep Technical Accuracy**: Ground every assertion in actual code paths, schema models, error handlers, and state transitions.

### Stage 3: Reader Testing & Polish
- **Fresh-Eyes Review**: Evaluate the documentation from the perspective of an engineer or stakeholder reading it without internal context.
- **Gap & Blind-Spot Detection**: Identify unstated assumptions, ambiguous terminology, or missing edge-case explanations.
- **Final Polish**: Format with clear diagrams (Mermaid), tables, code snippets, and cross-references.
