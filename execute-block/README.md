# Execute Block (at Insertion Point in an Editor)

To run the block-at-point, you need a standard way to get the code block from the document.

There's no universal standard. 
But a popular way would be to conform to CommonMark.
Pass the **line number and UTF-8 column offset (1-based)** so that any CommonMark parser can walk the AST to get there.

    (File, LineNum, Column) -> (CodeBlock, ResultBlock?, ErrorBlock?)

The job of `md-babel` as a program is to offer this and perform the code block selection, and then the transformation.

While this can be run manually from the command line, executing a block-at-point implies an interactive editor that keeps track of cursor positions.

To prepare for the _N:M_ problem and avoid that each editor needs to implement this, a **communication protocol** is in order, with `md-babel` acting as server.

Its result can be expressed as a **command** or effect object.
Somewhat similar to LSP, JSON payloads can be devised to represent the result `(CodeBlock, ResultBlock?, ErrorBlock?)` in rendered form, preserving context like indentation levels and blockquote embedding.

## Principles

- **Input range and replacement range usually differ.** 
  The editor can request to execute a code block identified by a location with length 0. 
  The replacement range is expanded to at least the code block, maybe even further to replace old results.
- **Provide a pre-formatted string to clients.**
  An AST-aware tool can do a great job at formatting the replacement to fit the context, preserving indentation and blockquote levels, for example.
- **Offer individual pieces for client flexibility.**
  Clients may not *want* to display the result in the document, but in a separate result buffer or console instead.
- **Errors are not results.** 
  Clients can decide to discard the replacement string in case of errors, and display informational dialogs instead.

## Sample 

Given this part of a document:

    Text ...
    
    ```sh
    dateˇ
    ```
    
    ... more text ...

Where `ˇ` denotes the insertion point, and its location is line 101, and its offset is 5.

Then the result of executing the `date` program for the ISO time `2025-04-06T13:44:00+0200` should transform the document into:


    Text ...
    
    ```sh
    dateˇ
    ```

    <!--Result-->
    ```
    Sun Apr  6 13:44:00 CEST 2025
    ```

    ... more text ...

From the user's perspective, the result is inlined as another code block with a metadata comment above.

The command to perform this change coming from `md-babel` expands the input range to cover the whole code block:

```json
{
  "range": {
    "from": { "line": 104, "column": 5 },
    "to": { "line": 104, "column": 5 }
  },
  "replacement_range": {
    "from": { "line": 100, "column": 1 },
    "to": { "line": 103, "column": 1 }
  },
  "replacement_string": "```sh\ndate\n```\n\n<!--Result-->\n```\nSun Apr  6 13:39:24 CEST 2025\n```",
  "result": "Sun Apr  6 13:39:24 CEST 2025\n",
  "error": null
}
```

Running the code block again will update the time in the result block below, extending the replacement range by 5 more lines to cover the existing result block:

```json
{
  "range": {
    "from": { "line": 104, "column": 5 },
    "to": { "line": 104, "column": 5 }
  },
  "replacement_range": {
    "from": { "line": 100, "column": 1 },
    "to": { "line": 108, "column": 1 }
  },
  "replacement_string": "```sh\ndate\n```\n\n<!--Result-->\n```\nSun Apr  6 13:51:47 CEST 2025\n```",
  "result": "Sun Apr  6 13:51:47 CEST 2025\n",
  "error": null
}
```
