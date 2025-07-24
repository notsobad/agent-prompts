# 使用function call形式实现MCP调用
这种方式，每个user消息中，带上一个tools，将mcp tools包装成function call提供给LLM。


## User msg
```json
{
    "messages": [{
        "content": "You are a helpful AI assistant.
You have access to the following tools:

- read_file: Read the complete contents of a file from the file system. Handles various text encodings and provides detailed error messages if the file cannot be read. Use this tool when you need to examine the contents of a single file. Only works within allowed directories.
- read_multiple_files: Read the contents of multiple files simultaneously. This is more efficient than reading files one by one when you need to analyze or compare multiple files. Each file's content is returned with its path as a reference. Failed reads for individual files won't stop the entire operation. Only works within allowed directories.
- write_file: Create a new file or completely overwrite an existing file with new content. Use with caution as it will overwrite existing files without warning. Handles text content with proper encoding. Only works within allowed directories.
- edit_file: Make line-based edits to a text file. Each edit replaces exact line sequences with new content. Returns a git-style diff showing the changes made. Only works within allowed directories.
- create_directory: Create a new directory or ensure a directory exists. Can create multiple nested directories in one operation. If the directory already exists, this operation will succeed silently. Perfect for setting up directory structures for projects or ensuring required paths exist. Only works within allowed directories.
- list_directory: Get a detailed listing of all files and directories in a specified path. Results clearly distinguish between files and directories with [FILE] and [DIR] prefixes. This tool is essential for understanding directory structure and finding specific files within a directory. Only works within allowed directories.
- directory_tree: Get a recursive tree view of files and directories as a JSON structure. Each entry includes 'name', 'type' (file/directory), and 'children' for directories. Files have no children array, while directories always have a children array (which may be empty). The output is formatted with 2-space indentation for readability. Only works within allowed directories.
- move_file: Move or rename files and directories. Can move files between directories and rename them in a single operation. If the destination exists, the operation will fail. Works across different directories and can be used for simple renaming within the same directory. Both source and destination must be within allowed directories.
- search_files: Recursively search for files and directories matching a pattern. Searches through all subdirectories from the starting path. The search is case-insensitive and matches partial names. Returns full paths to all matching items. Great for finding files when you don't know their exact location. Only searches within allowed directories.
- get_file_info: Retrieve detailed metadata about a file or directory. Returns comprehensive information including size, creation time, last modified time, permissions, and type. This tool is perfect for understanding file characteristics without reading the actual content. Only works within allowed directories.
- list_allowed_directories: Returns the list of directories that this server is allowed to access. Use this to understand which directories are available before trying to access files.

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of the available tools
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question",
        "role": "system"
    }, {
        "content": "Hello can you give me a list of files and directories in the current directory",
        "role": "user"
    }, {
        "content": "Hello can you give me a list of files and directories in the current directory",
        "role": "user"
    }],
    "model": "auto",
    "stream": true,
    "tools": [{
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read the complete contents of a file from the file system. Handles various text encodings and provides detailed error messages if the file cannot be read. Use this tool when you need to examine the contents of a single file. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "read_multiple_files",
            "description": "Read the contents of multiple files simultaneously. This is more efficient than reading files one by one when you need to analyze or compare multiple files. Each file's content is returned with its path as a reference. Failed reads for individual files won't stop the entire operation. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "paths": {
                        "items": {
                            "type": "string"
                        },
                        "type": "array"
                    }
                },
                "required": ["paths"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "write_file",
            "description": "Create a new file or completely overwrite an existing file with new content. Use with caution as it will overwrite existing files without warning. Handles text content with proper encoding. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    },
                    "content": {
                        "type": "string"
                    }
                },
                "required": ["path", "content"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "edit_file",
            "description": "Make line-based edits to a text file. Each edit replaces exact line sequences with new content. Returns a git-style diff showing the changes made. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    },
                    "edits": {
                        "items": {
                            "properties": {
                                "oldText": {
                                    "description": "Text to search for - must match exactly",
                                    "type": "string"
                                },
                                "newText": {
                                    "description": "Text to replace with",
                                    "type": "string"
                                }
                            },
                            "required": ["oldText", "newText"],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "dryRun": {
                        "anyOf": [{
                            "type": "boolean"
                        }, {
                            "type": "null"
                        }],
                        "default": false,
                        "description": "Preview changes using git-style diff format"
                    }
                },
                "required": ["path", "edits"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "create_directory",
            "description": "Create a new directory or ensure a directory exists. Can create multiple nested directories in one operation. If the directory already exists, this operation will succeed silently. Perfect for setting up directory structures for projects or ensuring required paths exist. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "list_directory",
            "description": "Get a detailed listing of all files and directories in a specified path. Results clearly distinguish between files and directories with [FILE] and [DIR] prefixes. This tool is essential for understanding directory structure and finding specific files within a directory. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "directory_tree",
            "description": "Get a recursive tree view of files and directories as a JSON structure. Each entry includes 'name', 'type' (file/directory), and 'children' for directories. Files have no children array, while directories always have a children array (which may be empty). The output is formatted with 2-space indentation for readability. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "move_file",
            "description": "Move or rename files and directories. Can move files between directories and rename them in a single operation. If the destination exists, the operation will fail. Works across different directories and can be used for simple renaming within the same directory. Both source and destination must be within allowed directories.",
            "parameters": {
                "properties": {
                    "source": {
                        "type": "string"
                    },
                    "destination": {
                        "type": "string"
                    }
                },
                "required": ["source", "destination"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "search_files",
            "description": "Recursively search for files and directories matching a pattern. Searches through all subdirectories from the starting path. The search is case-insensitive and matches partial names. Returns full paths to all matching items. Great for finding files when you don't know their exact location. Only searches within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    },
                    "pattern": {
                        "type": "string"
                    },
                    "excludePatterns": {
                        "anyOf": [{
                            "items": {
                                "type": "string"
                            },
                            "type": "array"
                        }, {
                            "type": "null"
                        }],
                        "default": []
                    }
                },
                "required": ["path", "pattern"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "get_file_info",
            "description": "Retrieve detailed metadata about a file or directory. Returns comprehensive information including size, creation time, last modified time, permissions, and type. This tool is perfect for understanding file characteristics without reading the actual content. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "list_allowed_directories",
            "description": "Returns the list of directories that this server is allowed to access. Use this to understand which directories are available before trying to access files.",
            "parameters": {
                "properties": {},
                "type": "object"
            }
        }
    }]
}
```

## LLM return tool use

LLM 返回一个tool call调用，本地调用MCP后，再把结果提交给LLM
```json
{
    "id": "9df1d3be4e03f6ababb07ae418314de1",
    "object": "chat.completion.chunk",
    "created": 1748400812,
    "model": "deepseek-v3-0324",
    "choices": [{
            "role": "assistant",
            "content": "",
            "tool_calls": [{
                "index": 0,
                "id": "call_deghjhzy",
                "type": "function",
                "function": {
                    "name": "list_allowed_directories",
                    "arguments": ""
                }
            }]
    }],
    "usage": {
        "prompt_tokens": 2107,
        "completion_tokens": 14,
        "total_tokens": 2121
    }
}
```

## 客户端调用MCP后提交结果给LLM

```
{
    "messages": [{
        "content": "You are a helpful AI assistant.
You have access to the following tools:

- read_file: Read the complete contents of a file from the file system. Handles various text encodings and provides detailed error messages if the file cannot be read. Use this tool when you need to examine the contents of a single file. Only works within allowed directories.
- read_multiple_files: Read the contents of multiple files simultaneously. This is more efficient than reading files one by one when you need to analyze or compare multiple files. Each file's content is returned with its path as a reference. Failed reads for individual files won't stop the entire operation. Only works within allowed directories.
- write_file: Create a new file or completely overwrite an existing file with new content. Use with caution as it will overwrite existing files without warning. Handles text content with proper encoding. Only works within allowed directories.
- edit_file: Make line-based edits to a text file. Each edit replaces exact line sequences with new content. Returns a git-style diff showing the changes made. Only works within allowed directories.
- create_directory: Create a new directory or ensure a directory exists. Can create multiple nested directories in one operation. If the directory already exists, this operation will succeed silently. Perfect for setting up directory structures for projects or ensuring required paths exist. Only works within allowed directories.
- list_directory: Get a detailed listing of all files and directories in a specified path. Results clearly distinguish between files and directories with [FILE] and [DIR] prefixes. This tool is essential for understanding directory structure and finding specific files within a directory. Only works within allowed directories.
- directory_tree: Get a recursive tree view of files and directories as a JSON structure. Each entry includes 'name', 'type' (file/directory), and 'children' for directories. Files have no children array, while directories always have a children array (which may be empty). The output is formatted with 2-space indentation for readability. Only works within allowed directories.
- move_file: Move or rename files and directories. Can move files between directories and rename them in a single operation. If the destination exists, the operation will fail. Works across different directories and can be used for simple renaming within the same directory. Both source and destination must be within allowed directories.
- search_files: Recursively search for files and directories matching a pattern. Searches through all subdirectories from the starting path. The search is case-insensitive and matches partial names. Returns full paths to all matching items. Great for finding files when you don't know their exact location. Only searches within allowed directories.
- get_file_info: Retrieve detailed metadata about a file or directory. Returns comprehensive information including size, creation time, last modified time, permissions, and type. This tool is perfect for understanding file characteristics without reading the actual content. Only works within allowed directories.
- list_allowed_directories: Returns the list of directories that this server is allowed to access. Use this to understand which directories are available before trying to access files.

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of the available tools
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question",
        "role": "system"
    }, {
        "content": "Hello can you give me a list of files and directories in the current directory",
        "role": "user"
    }, {
        "content": "Hello can you give me a list of files and directories in the current directory",
        "role": "user"
    }, {
        "content": null,
        "role": "assistant",
        "tool_calls": [{
            "type": "function",
            "id": "call_deghjhzy",
            "function": {
                "name": "list_allowed_directories",
                "arguments": "{}"
            }
        }]
    }, {
        "content": "Allowed directories:
/Users/xxx/tmp",
        "role": "tool",
        "tool_call_id": "call_deghjhzy"
    }],
    "model": "auto",
    "stream": true,
    "tools": [{
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read the complete contents of a file from the file system. Handles various text encodings and provides detailed error messages if the file cannot be read. Use this tool when you need to examine the contents of a single file. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "read_multiple_files",
            "description": "Read the contents of multiple files simultaneously. This is more efficient than reading files one by one when you need to analyze or compare multiple files. Each file's content is returned with its path as a reference. Failed reads for individual files won't stop the entire operation. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "paths": {
                        "items": {
                            "type": "string"
                        },
                        "type": "array"
                    }
                },
                "required": ["paths"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "write_file",
            "description": "Create a new file or completely overwrite an existing file with new content. Use with caution as it will overwrite existing files without warning. Handles text content with proper encoding. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    },
                    "content": {
                        "type": "string"
                    }
                },
                "required": ["path", "content"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "edit_file",
            "description": "Make line-based edits to a text file. Each edit replaces exact line sequences with new content. Returns a git-style diff showing the changes made. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    },
                    "edits": {
                        "items": {
                            "properties": {
                                "oldText": {
                                    "description": "Text to search for - must match exactly",
                                    "type": "string"
                                },
                                "newText": {
                                    "description": "Text to replace with",
                                    "type": "string"
                                }
                            },
                            "required": ["oldText", "newText"],
                            "type": "object"
                        },
                        "type": "array"
                    },
                    "dryRun": {
                        "anyOf": [{
                            "type": "boolean"
                        }, {
                            "type": "null"
                        }],
                        "default": false,
                        "description": "Preview changes using git-style diff format"
                    }
                },
                "required": ["path", "edits"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "create_directory",
            "description": "Create a new directory or ensure a directory exists. Can create multiple nested directories in one operation. If the directory already exists, this operation will succeed silently. Perfect for setting up directory structures for projects or ensuring required paths exist. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "list_directory",
            "description": "Get a detailed listing of all files and directories in a specified path. Results clearly distinguish between files and directories with [FILE] and [DIR] prefixes. This tool is essential for understanding directory structure and finding specific files within a directory. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "directory_tree",
            "description": "Get a recursive tree view of files and directories as a JSON structure. Each entry includes 'name', 'type' (file/directory), and 'children' for directories. Files have no children array, while directories always have a children array (which may be empty). The output is formatted with 2-space indentation for readability. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "move_file",
            "description": "Move or rename files and directories. Can move files between directories and rename them in a single operation. If the destination exists, the operation will fail. Works across different directories and can be used for simple renaming within the same directory. Both source and destination must be within allowed directories.",
            "parameters": {
                "properties": {
                    "source": {
                        "type": "string"
                    },
                    "destination": {
                        "type": "string"
                    }
                },
                "required": ["source", "destination"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "search_files",
            "description": "Recursively search for files and directories matching a pattern. Searches through all subdirectories from the starting path. The search is case-insensitive and matches partial names. Returns full paths to all matching items. Great for finding files when you don't know their exact location. Only searches within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    },
                    "pattern": {
                        "type": "string"
                    },
                    "excludePatterns": {
                        "anyOf": [{
                            "items": {
                                "type": "string"
                            },
                            "type": "array"
                        }, {
                            "type": "null"
                        }],
                        "default": []
                    }
                },
                "required": ["path", "pattern"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "get_file_info",
            "description": "Retrieve detailed metadata about a file or directory. Returns comprehensive information including size, creation time, last modified time, permissions, and type. This tool is perfect for understanding file characteristics without reading the actual content. Only works within allowed directories.",
            "parameters": {
                "properties": {
                    "path": {
                        "type": "string"
                    }
                },
                "required": ["path"],
                "type": "object"
            }
        }
    }, {
        "type": "function",
        "function": {
            "name": "list_allowed_directories",
            "description": "Returns the list of directories that this server is allowed to access. Use this to understand which directories are available before trying to access files.",
            "parameters": {
                "properties": {},
                "type": "object"
            }
        }
    }]
}
```