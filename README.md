# Bubble: Bulk Image Processing Website

Bubble is a web application designed for efficient, large-scale image processing. The project is developed using a unique multi-agent workflow, where each agent is responsible for a specific aspect of the product lifecycle. This ensures clarity, quality, and a clear separation of concerns throughout development.

## Project Overview

Bubble enables users to process images in bulk, streamlining tasks that would otherwise be repetitive and time-consuming. The application is built with a strong focus on reliability, usability, and extensibility.

## Agent-Based Development Process

Bubble is developed by six independent AI agents, each with a clearly defined role:

1. **Knowledge Agent** (Product Owner)
   - Maintains product requirements and decisions
   - Answers questions about user flows and edge cases
   - Updates `docs/requirements.md` as the single source of truth
2. **Architect Agent** (Tech Lead)
   - Makes technical and architectural decisions
   - Defines interfaces and patterns
   - Updates `docs/architecture.md` and `docs/conventions.md`
3. **Planner Agent** (Project Manager)
   - Breaks work into tasks and tracks progress
   - Maintains `docs/tasks.md`
4. **Implementer Agent** (Developer)
   - Writes production code for each task
   - Follows architecture and conventions
5. **Test Agent** (QA Engineer)
   - Writes and maintains tests
   - Ensures code meets requirements and quality standards
6. **Reviewer Agent** (Code Reviewer)
   - Reviews code changes for quality and correctness
   - Provides actionable feedback

Each agent has a private scratchpad in `docs/agents/<agent-name>/notes.md` for session-to-session context.

## Key Documents

- **docs/requirements.md**: Product requirements and decisions (Knowledge Agent)
- **docs/architecture.md**: Technical architecture (Architect Agent)
- **docs/conventions.md**: Coding and testing conventions (Architect Agent)
- **docs/tasks.md**: Task list and progress (Planner Agent)
- **docs/agents/**: Private notes for each agent

## Workflow

1. Check `docs/tasks.md` for the next task
2. Test Agent writes failing tests for the task
3. Implementer Agent writes code to pass the tests
4. Reviewer Agent reviews the changes
5. Human updates task status in `docs/tasks.md`
6. Repeat for the next task

Agents never communicate directly; the human developer coordinates all interactions and resolves conflicts.

## Contributing

- All decisions and changes must be documented in the appropriate file.
- Only the human developer commits code and resolves conflicts.
- See the `docs/agents.md` file for detailed agent roles and guidelines.

## License

[Specify your license here]

---

For more details, see the documentation in the `docs/` folder.