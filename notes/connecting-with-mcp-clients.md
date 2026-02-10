## Connecting with MCP clients

### Implementing a client

- the client side is what allows our application code to communicate with the MCP server and access its functionality. 

- the MCP client consists of two main components: 
    1. MCP client - a custom class we create to make using the session easier
    2. Client session - the actual connection to the server [part of the MCP Python SDK]

- the client session requires resource management i.e., we need to properly clean up connections when we're done.
- how?? wrap it in our own class that handles the cleanup automatically. 

- our CLI code uses the client to:
    - get a list of available tools to send to Claude
    - execute tools when Claude requests them


### Implementing core client functions

- two essential functions:
    - `list_tools()`
    - `call_tool()`

1. List Tools Function

- this function gets all available tools from the MCP server

```python
async def list_tools(self) -> list[types.Tool]:
    result = await self.session().list_tools()
    return result.tools
```

- here we access our session which is the connection to the server, call the built-in method and return the tools from the result. 


2. Call Tool Function

- this function executes a specific tool on the server:

```python
async def call_tool(
    self, tool_name: str, tool_input: dict
        ) -> types.CallToolResult | None:
    return await self.session().call_tool(tool_name, tool_input)
```

- we pass the tool name and input params (provided by Claude) to the server and return the result.


### Defining resources

- resources in the MCP servers allow us to expose data to clients, similar to `GET` request handlers in a typical HTTP server.
- they're perfect for scenarios where we need to fetch information rather than perform actions. 

- for example: if we want to build a document mention feature where users can type `@document_name` to reference files, we need two operations:
    1. one for getting a list of all available documents (for autocomplete)
    2. the other one for fetching the contents of a specific document (when mentioned)

- now when a user mentions a document, the system automatically injects the document's contents into the prompt send to Claude, eliminating the need for Claude to use tools to fetch the information.


### How resources work

- resources follow a request-response pattern. 
- when the client needs data, it sends a `ReadResourceRequest` with a URI to identify which resource it wants.
- the MCP server processes this request and returns the data in a `ReadResourceResult`. 


### Types of resources

- there are two types of resources:

1. Direct Resources

- direct resources have static URIs that never change. they're perfect for operations that don't need parameters. 

```python
@mcp.resource(
    "docs://documents",
    mime_type="application/json"
)
def list_docs() -> list[str]:
    return list(docs.keys())
```

2. Templated Resources

- templated resources include params in their URIs. the python SDK automatically parses these params and passes them as keyword arguments to our function. 

```python
@mcp.resource(
    "docs://documents/{doc_id}",
    mime_type="text/plain"
)
def fetch_doc(doc_id: str) -> str:
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    return docs[doc_id]
```


### Implementation Details

- resources can return any type of data; strings, JSON, binary data etc
- we can use the `mime_type parameter to give clients a hint about what kind of data we are returning:
    - "application/json" for structured data
    - "text/plain" for plain text
    - "application/pdf" for binary files

- the MCP Python SDK automatically serialize our return values. 
- we don't need to manually convert objects to JSON strings just return the data structure and let the SDK handle serialization. 



---

### Accessing resources

- resources in MCP allow our server to expose information that can be directly included in prompt, rather than requiring tool calls to access data.
- this creates a more efficient way to provide context to AI models

- a quick recap on the workflow of how resources work:
    - when a user types something like "What's in the @..." our code immediately recognises this as a resource request, sends a ReadResourceRequest to the MCP server, and gets back a ReadResourceResult with the actual content. 


### Implementing resource reading

- to enable resource access in our MCP client, we need to implement a `read_resource` function. 
- first we import the necessary libraries

```python
import json
from pydantic import AnyUrl
```

- the core function makes a request to the MCP server and processes the response based on its MIME type:

```python
async def read_resource(self, uri: str) -> Any:
    result = await self.session().read_resource(AnyUrl(uri))
    resource = result.contents[0]
    
    if isinstance(resource, types.TextResourceContents):
        if resource.mimeType == "application/json":
            return json.loads(resource.text)
    
    return resource.text
```


### Understanding the response structure

- when we request a resource, the server returns a result with a `contents` list. 
- we access the first element since we typically only need one resource at a time. - the response includes:
    - the actual content (text or data)
    - a MIME type that tells us how to parse the content
    - other metadata about the resource


### Content Type Handling

- the function checks the MIME type to determine how to process the content:

    - if it's `application/json`, parse the text as JSON and return the parsed object.
    - otherwise, return the raw text content

- this approach handles both structured data (like JSON) and plain documents seamlessly.


---

### Defining prompts

- prompts in MCP let's define pre-built, high-quality instructions that clients can use instead of writing their own prompts from scratch. 
- we can think of these as carefully crafted templates that give better results than what users might come up with on their own.


### Why Use Prompts?

- key insight: user can already ask Claude to do most tasks directly. for example, a user could type "reformat the report.pdf in markdown" and get decent results.
- but they will get much better results if we can provide a thoroughly tested, specialised prompt that handles edge cases and follows best practices.

- as the MCP server authors, we can spend time crafting, testing, and evaluating prompts that work consistently across different scenarios. 
- users can benefit from this expertise without having to become prompt engineering experts themselves. 


### Building a Format Command

- let's take for example, a format command that converts documents to markdown. users can type `/format doc_id` and get back a professionally formatted markdown version of their document.

the workflow looks like this:
    - users type / to see available commands
    - they then select `format` and specify a document ID
    - claude uses the pre-built prompt to read and reformat the document
    - the result is clean markdown with proper headers, lists, and formatting

### Defining Prompts

- prompts use a similar decorator pattern to tools and resources:

```python
@mcp.prompt(
    name="format",
    description="Rewrites the contents of the document in Markdown format."
)
def format_document(
    doc_id: str = Field(description="Id of the document to format")
) -> list[base.Message]:
    prompt = f"""
Your goal is to reformat a document to be written with markdown syntax.

The id of the document you need to reformat is:
<document_id>
{doc_id}
</document_id>

Add in headers, bullet points, tables, etc as necessary. Feel free to add in structure.
Use the 'edit_document' tool to edit the document. After the document has been reformatted...
"""
    
    return [
        base.UserMessage(prompt)
    ]
```

- the function above returns a list of messages that get sent directly to Claude. we can include multiple user and assistant messages to create more complex conversation flows. 


Key Benefits

- consistency: users get reliable results every time
- expertise: we can encode domain knowledge into prompts
- reusability: multiple client applications can use the same prompts
- maintainance: update prompts in one place to improve all

Note: prompts work best when they are specialised for our MCP server's domain. any document management server might have prompts for formatting, summarising, or analysing documents.
- a data analysis server might have prompts for generating reports or visualisations
- the goal is to provide prompts that are so well-crafted and tested that users prefer them over writing their own instructions from scratch. 

---

### Prompts in the client

- the `list_prompts` method is straightforward. it calls the session's list prompts function and returns the prompts:

```python
async def list_prompts(self) -> list[types.Prompt]:
    result = await self.session().list_prompts()
    return result.prompts
```

- the `get_prompts` method handles variable interpolation. when we request a prompt, we provide arguments that get passed to the prompt function as keyword arguments:

```python
async def get_prompt(self, prompt_name, args: dict[str, str]):
    result = await self.session().get_prompt(prompt_name, args)
    return result.messages
```

- e.g., if the server has a `format_document` prompt that expects a `doc_id` param, the arguments dictionary would contain `{"doc_id": "plan.md"}`. this value gets interpolated into the prompt template


### How Prompts Work?

- prompts define a set of user and assistant messages that clients can use. they should be high-quality, well-tested, and relevant to our MCP server's purpose