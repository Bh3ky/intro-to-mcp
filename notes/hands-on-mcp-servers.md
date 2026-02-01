## Defining tools with MCP

- building an MCP server becomes much simpler when we use the official Python SDK.
- here we'll be creating document management server with two core tools:
  - one to read documents and another to update them.
  - Note: all documents exist in memory as a simple dictionary where keys are document IDs and values are the content.

### Setting Up the MCP Server

- the Python MCP SDK makes server creation straightforward.
- here we can initialise a serve with just one line:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("DocumentMCP", log_level="ERROR")
```

- our documents can be stored in a simple dictionary structure:

```python
docs = {
    "deposition.md": "This deposition covers the testimony of Angela Smith, P.E.",
    "report.pdf": "The report details the state of a 20m condenser tower.",
    "financials.docx": "These financials outline the project's budget and expenditures",
    "outlook.pdf": "This document presents the projected future performance of the system",
    "plan.md": "The plan outlines the steps for the project's implementation.",
    "spec.txt": "These specifications define the technical requirements for the equipment"
}
```

### Tool Definition with Decorators

- the SDK uses decorators to define tools.
- instead of writing JSON schemas manually we can use Python type hints and field descriptions
- the SDK automatically generate the proper schema that Claude can understand. 


### Creating a Document Reader Tool

- the first tool reads document contents by ID

```python
@mcp.tool(
    name="read_doc_contents",
    description="Read the contents of a document and return it as a string."
)
def read_document(
    doc_id: str = Field(description="Id of the document to read")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    
    return docs[doc_id]

```

- the decorator specifies the tool name and description, while the function parameters define the required arguments
- the `Field` class from Pydantic provides arguments description that help Clause understand what each param expects


### Building a Document Editor Tool

- the second tool performs simple find-and-replace operations on documents:

```python
@mcp.tool(
    name="edit_document",
    description="Edit a document by replacing a string in the documents content with a new string."
)
def edit_document(
    doc_id: str = Field(description="Id of the document that will be edited"),
    old_str: str = Field(description="The text to replace. Must match exactly, including whitespace."),
    new_str: str = Field(description="The new text to insert in place of the old text.")
):
    if doc_id not in docs:
        raise ValueError(f"Doc with id {doc_id} not found")
    
    docs[doc_id] = docs[doc_id].replace(old_str, new_str)
```

- this tool takes three params:
  - document ID
  - text to find
  - replacement text
- Note: this implementation includes error handling for missing documents and performs a straightforward string replacement


QUE: What are some of the key benefits of the SDK approach??

- No manual JSON schema writing required
- Type hints provide automatic validation
- Clear parameter descriptions help Claude understand tool usage
- Error handling integrates naturally with Python exceptions
- Tool registration happens automatically through decorators


### Ther server inspector

- when building MCP servers, we need a way to test the functionality without connecting to a full application. 
- the Python MCP SDK includes a built-in browser-based inspector that lets us debug and test our server in real-time.

`mcp dev mcp_server.py` starts a development server and gives us a local URL

- key elements are:
    - connect button to start our MCP server
    - nagivation tabs for resources, tools, prompts and other features
    - tool listing and testing