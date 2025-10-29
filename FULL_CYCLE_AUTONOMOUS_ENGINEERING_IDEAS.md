## General Concepts to Include

- In Claude Code, use before_compact hook to write out status and next steps
- Use a local SQLLite to track issues list
- Issues list should track priorities, predecessors, status
- Should include a command to “re-engineer” the issue list
- What’s the connection between issue list and to do list? Is there a to do list for each issue? Is the GitHub Issue and PR model the right one to use?
- Leverage skills from something like Superpowers, agents from a big list of agents
- Send each issue to a subagent. How are things coordinated centrally? Single coordinator agent? How do they all “survive” compactification?
- Is there a way to manage Claude Code context so it’s not a dump of every blasted conversation?
- Where is an index of the codebase stored?
- What about special branches for things like deployment or migration?
