# Roots

- roots are a way to grand MCP servers access to specific files and folders on our local machine.
    - these are like permission systems

**The Problem Roots Solve**

- without roots, we'd run into a common issue. suppose we have an MCP server with a video conversion tool that takes a file path and converts an MP4 to MOV format. now if we have a complex codebase or filesystem it might be different for our agent to find the specific file we need converted.

Roots in Action

- the workflow changes with roots:
    1. user asks to convert a video file
    2. claude calls `list-roots` to see waht directories it can access
    3. claude calls `read_dir` on accessible directories to find the file
    4. once found, Claude calls the conversion tool with the full path

> Note: this happens automatically


**Security and Boundaries**

- roots also provide security by limiting access e.g., if we only grant access to our Desktop folder, the MCP server cannot access files in other locations like Documents or Downloads.

**Implementation Details**

- the MCP SDK doesn't automatically enforce root restrictions, we need to implement this ourselves. how???
    - create a helper function like `is_path_allowed()` that:
        - takes a request file path, gets the list of approaved roots, checks if the requested path falls within one of those roots, and returns true/false for access permissions.
    - we can then call this function in any tool that accesses files or directories before performing the actual file operation. 