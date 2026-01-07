# Long-Running Tool Call with Approvals

This sample demonstrates how to implement AI agent tool calls that require user approval, with support for long-running operations and cancellation handling.

## Overview

The sample showcases:
- **Tool Approval Workflow**: AI functions that require explicit user approval before execution
- **Long-Running Operations**: Simulated slow operations with graceful cancellation support
- **Streaming & Non-Streaming Modes**: Both response modes with approval handling
- **Cancellation Support**: Ability to cancel operations via Ctrl+C or user input

## Features

### AI Functions with Approval

Two weather-related functions demonstrate the approval pattern:
- `get_weather(location)`: Simple weather lookup requiring approval
- `get_weather_detail(location)`: Detailed weather forecast requiring approval

Both functions:
- Use `approval_mode="always_require"` to request user permission
- Simulate 5-second operations to demonstrate long-running tasks
- Support cancellation during execution

### Approval Handling

The sample includes two approval handlers:
- **`handle_approvals()`**: For standard (non-streaming) responses
- **`handle_approvals_streaming()`**: For streaming responses

Both maintain conversation context including:
- Original user query
- Assistant's approval request
- User's approval response

### Cancellation Options

Users can cancel operations in multiple ways:
- Press **Ctrl+C** during tool execution
- Type **'c'** when prompted for approval
- Type **'n'** to deny approval and continue

## Prerequisites

- Python 3.8 or later
- Azure OpenAI account and credentials
- Microsoft Agent Framework

## Setup

1. **Install dependencies**:
   ```bash
   pip install agent-framework --pre
   ```

2. **Configure Azure OpenAI**:
   Set up your Azure OpenAI credentials (environment variables or configuration file as required by `AzureOpenAIResponsesClient`).
   see **`.env.sample`**

3. **Run the sample**:
   ```bash
   python main.py
   ```

## Usage

When you run the sample, you'll see:

1. The agent receives a query: *"Can you give me an update of the weather in Bangkok?"*
2. The agent decides to call the `get_weather` or `get_weather_detail` function
3. You're prompted to approve the function call:
   ```
   User Input Request for function from WeatherAgent:
     Function: get_weather
     Arguments: {"location": "Bangkok"}
   
   Approve function call? (y/n/c to cancel):
   ```
4. Options:
   - **y**: Approve and execute the function
   - **n**: Deny the function call
   - **c**: Cancel the entire operation

5. If approved, the tool executes (with a 5-second delay) and can be cancelled with Ctrl+C

## Code Structure

### Main Components

- **AI Functions**: `get_weather()` and `get_weather_detail()` with `@ai_function` decorator
- **Approval Handlers**: `handle_approvals()` and `handle_approvals_streaming()`
- **Agent Setup**: `run_weather_agent_with_approval()` configures and runs the agent
- **Entry Point**: `main()` orchestrates the demonstration

### Key Patterns

**Approval Mode Configuration**:
```python
@ai_function(approval_mode="always_require")
async def get_weather(location: str) -> str:
    """Get the current weather for a given location."""
    # Function implementation
```

**Cancellable Operations**:
```python
try:
    await asyncio.sleep(simulate_delay_seconds)
except asyncio.CancelledError:
    print(f"[Tool execution cancelled]")
    raise
```

**Context Management**:
```python
# Include original query, approval request, and response
new_inputs = [
    query,
    ChatMessage(role="assistant", contents=[user_input_needed]),
    ChatMessage(role="user", contents=[approval_response])
]
```

## Configuration

Adjust the simulation delay by modifying:
```python
simulate_delay_seconds = 5  # Change to desired duration
```

Switch between streaming and non-streaming modes in `main()`:
```python
# Non-streaming mode
await run_weather_agent_with_approval(is_streaming=False)

# Streaming mode (default)
await run_weather_agent_with_approval(is_streaming=True)
```
