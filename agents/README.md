# Agents
This is for the executable code that uses the prompts.

## API Scripts
Contains the code files (e.g., Python) that implement the agent logic, handle API calls, manage context, and call tools.

- agent_config.json (A configuration file to store settings for all agents (e.g., model name, default temperature $T$, API key environment variable names))

### Manage senior SWE companion

- senior_software_engineer_agent.py (The script that loads senior_software_engineer_companion.md, defines its tools, and manages the conversation loop)

## Tools
For reusable functions that your agents can call. This keeps your main agent logic clean.

### Manage Gemini API

- google_search_tool.py (A function wrapped for the Gemini API that executes a search)