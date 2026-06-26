# Log and Progress Notifications

- logging and progress notifications help users understand what's happening during long-running operations.
- users get real-time feedback showing exactly what's happening behind the scenes. they can see progress bars, status message, and detailed logs as the operation runs.

**How it Works??**

- in the Python MCP SDK, logging and progress notifications work through the Context arguments that's automatically provided to your tool functions.
- this context object gives methods to communicate back to the client during execution.

```python
@mcp.tool(
    name="research",
    description="Research a given topic"
)
async def research(
    topic: str = Field(description="Topic to research"),
    *,
    context: Context
):
    await context.info("About to do research...")
    await context.report_progress(20, 100)
    sources = await do_research(topic)
    
    await context.info("Writing report...")
    await context.report_progress(70, 100)
    results = await generate_report(sources)
    
    return results
```

- key methods:

1. `context.info()` - send log messages to the client
2. `context.report_progress()` - update progress with current and total values.


**Client-Side Implementation**

- on the client side, we set up callback functions to handle these notifications. the server emits these messages, but it's up to the client application to decide how to present them to users.

```python
async def logging_callback(params: LoggingMessageNotificationParams):
    print(params.data)

async def print_progress_callback(
    progress: float, total: float | None, message: str | None
):
    if total is not None:
        percentage = (progress / total) * 100
        print(f"Progress: {progress}/{total} ({percentage:.1f}%)")
    else:
        print(f"Progress: {progress}")

async def run():
    async with stdio_client(server_params) as (read, write):
        async with ClientSession(
            read,
            write,
            logging_callback=logging_callback
        ) as session:
            await session.initialize()
            
            await session.call_tool(
                name="add",
                arguments={"a": 1, "b": 3},
                progress_callback=print_progress_callback,
            )
```

> we provide the logging callback when creating the client session, and the progress callback when making individual tool calls. this gives us flexibility to handle different types of notifications appropriately.


**How to present the notifications**

- depends on the application type:

1. CLI applications - simply print messages and progress to the terminal
2. Web applications - use WebSockets, server-sent events, or polling to push updates to the browser.
3. Desktop applications - update progress bar and status displays in the UI