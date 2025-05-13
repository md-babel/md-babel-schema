# Configuration Files

> [!IMPORTANT]
> This is work in progress. We converged on a working solution, but the configuration schema may change.

## Configuration File Locations 

Your global user configuration is stored in:

    $HOME/.config/md-babel/config.json

The root level object is a JSON object (`{...}`).

You can also pass the `--config` parameter to `md-babel` and load other files.
This allows editor frontends to store editor-specific settings internally, and merge them with the user's global configuration during execution.

You can also use per-project configuration files. 
It's a bit cumbersome to pass a file like this every time, until [automatic configuration file resolution is implemented](https://github.com/md-babel/swift-markdown-babel/issues/33).

## Configuring "Code Block Evaluators" 

Configure your `md-babel` backends so that the runtime knows how to interpret code blocks.

Code block evaluators go into the `evaluators.codeBlock` object path:

```json
{
  "evaluators": {
    "codeBlock": {
      ❶
    }
  }
}
```

❶ denotes where your evaluator configurations go.

The general setup is simple: 
For each code block's language, 
you need an entry of the same name in the configuration file.

This code block requires an entry called "bash":

    ```bash
    echo "Hi"
    ```

This requires one called "python":

    ```python
    print("Hello, world")
    ```

The language name right after the three backticks determines which evaluator will be used.

### How to Define Code Evaluator Backends

Here is an example configuration from `$HOME/.config/md-babel/config.json` that configures evaluators for `bash`, `ruby`, and `python`:

```json
{
  "evaluators": {
    "codeBlock": {
      "bash": {
	    "path": "/usr/bin/env",
	    "defaultArguments": ["bash"],
	    "result": "codeBlock"
	  },
      "ruby": {
	    "path": "/usr/bin/env",
	    "defaultArguments": ["ruby"],
	    "result": "codeBlock"
	  },
      "python": {
	    "path": "/Users/YOUR_USER_NAME/.config/md-babel/python3env/bin/python3",
	    "defaultArguments": [],
	    "result": "codeBlock"
	  }
    }
  }
}
```

Note that the entry for Python looks a bit different.

    /Users/YOUR_USER_NAME/.config/md-babel/python3env/bin/python3

Instead of discovering the system's `python3` via `/usr/bin/env`, it uses a virtual environment.

Create a virtual environment via:

```bash
python3 -m venv ~/.config/md-babel/python3env
```


> [!TIP]
> Do replace `/Users/YOUR_USER_NAME/` with the absolute path to your home directory.
> If you have md-babel installed, run this code block and copy the result:
> 
> ```bash
> echo $HOME/.config/md-babel/
> ```

With that, all your `md-babel`-executed Python code blocks can a single environment. 
You can install Python packages that are only available to `md-babel` this way and isolate your code blocks from your global setup by running `./bin/pip3` from that environment:

```bash
~/.config/md-babel/python3env/bin/pip3 install tomli_w
```

### Evaluator Examples

[Check out the Examples page](https://github.com/md-babel/swift-markdown-babel/blob/main/Examples.md) for configuration examples for Mermaid, PlantUML, and more.
