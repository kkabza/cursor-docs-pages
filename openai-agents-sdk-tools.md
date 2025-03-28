# Tools

## Hosted tools

The SDK has built-in support for invoking OpenAI's hosted tools, such as web search and file search.

```python
from agents import Agent, Runner, HostedTool

web_search_tool = HostedTool.web_search_tool()

agent = Agent(
    name="WebSearcher",
    instructions="Search the web to answer the user's question.",
    tools=[web_search_tool],
)

result = Runner.run_sync(agent, "What was the weather in Paris yesterday?")
```

Available hosted tools are:

* `HostedTool.file_search_tool()` - Searches a corpus of file-based documents.
* `HostedTool.web_search_tool()` - Searches the web.

## Function tools

Function tools are Python functions that can be invoked by the LLM.

You can create a function tool using the `@function_tool` decorator:

```python
from agents import Agent, function_tool, Runner

@function_tool
def triple(x: int) -> int:
    """Triple a number"""
    return x * 3

agent = Agent(
    name="Math Agent",
    instructions="Use the tools to triple the number",
    tools=[triple],
)

result = Runner.run_sync(agent, "Triple the number 4.")
print(result.final_output)
# 12
```

### Custom function tools

Sometimes you need a more customizable function tool. For this, you can use the `FunctionTool` class:

```python
import random
from agents import Agent, FunctionTool, Runner

def roll_invoke(ctx, sides: int):
    """Roll a die with the given number of sides."""
    return random.randint(1, sides)

roll_tool = FunctionTool(
    name="roll_die",
    description="Roll a die with the given number of sides",
    input_schema={"sides": {"type": "integer", "description": "Number of sides on the die"}},
    on_invoke_tool=roll_invoke,
)

agent = Agent(
    name="Dice Roller",
    instructions="Roll some dice",
    tools=[roll_tool],
)

result = Runner.run_sync(agent, "Roll a 6-sided die.")
print(result.final_output)
# I rolled a 6-sided die and got: 4
```

### Automatic argument and docstring parsing

When you use the `@function_tool` decorator, the SDK automatically converts the function's type hints to a JSON schema. Here are the rules:

- Standard Python types are converted directly: `int` → `integer`, `float` → `number`, `bool` → `boolean`, `str` → `string`, `None` → `null`.
- Docstrings are used to populate the function and parameter descriptions.
- Pydantic models are converted recursively to JSON schemas.

Here's an example:

```python
from pydantic import BaseModel
from typing import List, Optional
from agents import function_tool

class Address(BaseModel):
    street: str
    city: str
    zip_code: str
    state: Optional[str] = None

@function_tool
def create_person(name: str, age: int, addresses: List[Address]) -> str:
    """Create a new person with name and addresses.
    
    Args:
        name: The person's full name
        age: The person's age in years
        addresses: A list of the person's addresses
    
    Returns:
        The ID of the newly created person
    """
    # Implementation here
    return "person_1234"
```

The generated schema for this would include the function description, parameter descriptions, and the full schema for the `Address` model.

If you want to disable the automatic docstring parsing, you can pass `use_docstring_info=False` when creating the tool.

## Agents as tools

In some workflows, you may want a central agent to orchestrate a network of specialized agents, instead of handing off control. You can do this by modeling agents as tools.

```python
from agents import Agent, Runner
import asyncio

spanish_agent = Agent(
    name="Spanish agent",
    instructions="You translate the user's message to Spanish",
)

french_agent = Agent(
    name="French agent",
    instructions="You translate the user's message to French",
)

orchestrator_agent = Agent(
    name="orchestrator_agent",
    instructions=(
        "You are a translation agent. You use the tools given to you to translate."
        "If asked for multiple translations, you call the relevant tools."
    ),
    tools=[
        spanish_agent.as_tool(
            tool_name="translate_to_spanish",
            tool_description="Translate the user's message to Spanish",
        ),
        french_agent.as_tool(
            tool_name="translate_to_french",
            tool_description="Translate the user's message to French",
        ),
    ],
)

async def main():
    result = await Runner.run(orchestrator_agent, input="Say 'Hello, how are you?' in Spanish.")
    print(result.final_output)
```

## Handling errors in function tools

When you create a function tool via `@function_tool`, you can pass a `failure_error_function`. This is a function that provides an error response to the LLM in case the tool call crashes.

* By default (i.e. if you don't pass anything), it runs a `default_tool_error_function` which tells the LLM an error occurred.
* If you pass your own error function, it runs that instead, and sends the response to the LLM.
* If you explicitly pass `None`, then any tool call errors will be re-raised for you to handle. This could be a `ModelBehaviorError` if the model produced invalid JSON, or a `UserError` if your code crashed, etc.

If you are manually creating a `FunctionTool` object, then you must handle errors inside the `on_invoke_tool` function.

Source: [OpenAI Agents SDK Tools](https://openai.github.io/openai-agents-python/tools/) 