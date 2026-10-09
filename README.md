# Sensor Run Triage Agent

This project implements a read-only assistant for analyzing IMU sensor CSV files. It demonstrates a basic agent harness pattern where a language model (LLM) interacts with a limited set of Python tools to perform specific tasks.

## Features

- **List Runs**: Lists available CSV sensor run files in the `data/` directory.
- **Summarize Axis**: Calculates basic statistics (mean, standard deviation, min, max, RMS) for a specified sensor axis within a CSV file.
- **Detect Spikes**: Identifies potential statistical spikes in a sensor axis using a robust z-score method based on the median and median absolute deviation (MAD).

## Project Structure

```
sensor-triage-agent/
    data/                   # Contains sample sensor CSV files
    generate_data.py        # Script to generate sample sensor data
    tools.py                # Python functions that the agent can call
    test_tools.py           # Unit tests for the tools.py functions
    agent.py                # Implements the LLM-connected harness and tool calling logic
    main.py                 # Command-line entry point for the agent
    requirements.txt        # Project dependencies
```

## How the Agent Harness Works

The agent harness acts as a secure intermediary between the LLM and the Python tools. It:

1.  **Tells the model what it may do**: Provides the LLM with descriptions (schemas) of the available tools.
2.  **Gives it a limited set of tools**: The LLM can only request functions defined in `tools.py` and explicitly allowed by the harness.
3.  **Checks the model’s tool requests**: Validates the tool name and arguments requested by the LLM against an allowlist and schema.
4.  **Calls the Python functions itself**: Executes the allowed Python functions with validated arguments.
5.  **Sends results back to the model**: Returns the output of the tool execution to the LLM.
6.  **Limits loops and records what happened**: Imposes limits on the number of tool calls and rounds, and logs tool usage.

Crucially, the LLM does not directly execute any Python code. It *requests* a tool, and the harness *decides* whether to run it after rigorous validation.

## Setup and Installation

1.  **Clone the repository (if applicable) or create the project directory:**
    ```bash
    mkdir sensor-triage-agent
    cd sensor-triage-agent
    ```

2.  **Create a virtual environment and activate it:**
    ```bash
    python -m venv .venv
    # macOS / Linux
    source .venv/bin/activate
    # Windows PowerShell
    .venv\Scripts\Activate.ps1
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Generate sample sensor data:**
    ```bash
    python generate_data.py
    ```
    This will create a `data/` directory and a `sample_run.csv` file within it.

5.  **Run tests for the tools (optional but recommended):**
    ```bash
    pytest -q
    ```
    All tests in `test_tools.py` should pass.

## Running the Agent

To run the agent, you need a Groq API key. Set it as an environment variable:

```bash
# macOS / Linux
export GROQ_API_KEY="your-key"

# Windows PowerShell
$env:GROQ_API_KEY="your-key"
```

If you want to override the default model, set `GROQ_MODEL`:

```bash
export GROQ_MODEL="llama-3.3-70b-versatile"
# or in PowerShell
$env:GROQ_MODEL="llama-3.3-70b-versatile"
```

Now you can ask the agent questions:

```bash
python main.py "List the available runs."
```

```bash
python main.py "Check sample_run.csv for unusual az spikes and summarize the axis."
```

## Limitations and Safety

-   **Read-Only**: The agent is designed to be read-only and cannot modify files or control external systems. This is enforced by the harness's tool allowlist and input validation.
-   **Tool Allowlist**: Only explicitly defined and allowed functions in `tools.py` can be called by the LLM.
-   **Input Validation**: All arguments passed to tools are validated by the harness to prevent malicious inputs or unexpected behavior.
-   **Loop Limits**: The agent has limits on the number of tool calls and rounds to prevent infinite loops or excessive API usage.
-   **No Arbitrary Code Execution**: The LLM cannot execute arbitrary shell commands or Python code; it can only request predefined tools.
-   **Statistical vs. Diagnostic**: The spike detection is a statistical flag and not a definitive diagnosis of a hardware fault. Further human inspection is always required.

## Evaluation

When evaluating the agent, focus on:

-   **Tool Selection**: Did the agent choose the correct tool for the given question?
-   **Argument Validity**: Were the arguments provided to the tool valid and correctly formatted?
-   **Factual Accuracy**: Does the agent's final answer accurately reflect the results returned by the tools?
-   **Safety Compliance**: Did the agent adhere to all safety constraints (e.g., no unauthorized file access, no arbitrary code execution)?
-   **Efficiency**: Did the agent stop within the call limits and respond in a reasonable time?

Avoid judging solely on the LLM's wording; instead, verify that its claims are supported by the actual tool outputs and that the underlying Python functions are robust and correct (as verified by `test_tools.py`).
