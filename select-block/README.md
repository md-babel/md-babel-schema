# Select Block (at Insertion Point in an Editor)

A tool for interactive inspection:
Returns the code block (and results, if any) from a file and a source location.

There's no universal standard to specify source locations, but a popular way would be to conform to CommonMark.
Pass the **line number and UTF-8 column offset (1-based)** so that any CommonMark parser can walk the AST to get there.

The tool then returns an untransformed triple, a view into the document:

    (File, LineNum, Column) -> (CodeBlock, ResultBlock?, ErrorBlock?)

In a more general form that accepts not just code blocks but also tables:

    (File, LineNum, Column) -> (Input, ResultBlock?, ErrorBlock?)

## Sample

Given this part of a document:


    ... text ...
    
    ```sh
    dateˇ
    ```

    <!--Result-->
    ```
    Sun Apr  6 13:39:24 CEST 2025
    ```

    ... more text ...


Where `ˇ` denotes the insertion point, and its location is line 101, and its offset is 5.

The command to select the block and its associated result provided by `md-babel` expands the input range and extracts the contents for you:

```json
{
  "range": {
    "from": { "line": 104, "column": 5 },
    "to": { "line": 104, "column": 5 }
  },
  "input": {
    "range": {
      "from": { "line": 100, "column": 1 },
      "to": { "line": 103, "column": 1 }
    },
    "type": "code_block",
    "language": "sh",
    "content": "date"
  },
  "result": {
    "range": {
      "from": { "line": 104, "column": 1 },
      "to": { "line": 108, "column": 1 }
    },
    "type": "code_block",
    "language": "",
    "content": "Sun Apr  6 13:51:47 CEST 2025",
  }
}
```

- Each block (input and result) has its own range and Markdown block type.
- Notably absent: the optional `"error"` key, since no dedicated, persistent error block is in the document.
