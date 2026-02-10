## MCP Review

1. Tools: Model-Controlled

- tools are controlled entirely by Claude
- the AI model decides when to call these functions, and the results are used directly by Claude to accomplish tasks.

- tools are perfecrt for giving Claude additional capabilities it can use autonomously. when we ask Claude to "calculate the square root of 3 using JS," it's Claude that decides to use a JS execution tool to run the calculation.

2. Resources: App-Controlled

- resources are controlled by our application code. the app decides when to fetch resource data and how to use it - typically for UI elements or to add context to conversations

- in this project, we used resources in two ways:
    - fetching data to populate autocomplete options in the UI
    - retrieving content to augment propmpts with additional context

- for example: the "Add from Google Drive" feature in Claude's interface - the application code determines which documents to show and handles injecting their content into the chat context.


3. Prompts: User-Controlled

- prompts are triggered by user actions. users decide when to run these predefined workflows through UI interactions like button clicks, menu selections, or slash commands.

- prompts are ideal for implementing workflows that users can trigger on demand. in Claude's interface, those workflow buttons below the chat input are examples of prompts - predefined, optimised workflows that users can start with a single click.