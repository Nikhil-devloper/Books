# LangChain – Tools, Runtime Configuration, System Prompts, Structured Output & Memory

## 1. Tools in LangChain

### What are Tools?

Tools are components that allow an AI agent (LLM) to interact with external systems and perform actions.

An LLM by itself can generate responses based on its training and available context. However, it cannot automatically access our private backend, databases, APIs or internal business logic.

Tools bridge this gap.

**Definition:** Tools are callable functions that agents use to interact with external systems and execute specific actions.

### Why do we need Tools?

Consider a weather application.

A user visits our website and asks:

- What is the weather in New York?
- What is the current time in Paris?
- Has an earthquake occurred in Delhi?

A general LLM cannot automatically access our application's private backend or guarantee that its response comes from our weather API.

We need to connect our LLM to our backend using Tools.

### How Tools work

Example:

1. User asks: "What is the weather in New York?"
2. LLM analyzes the question.
3. LLM identifies that it needs weather information.
4. LLM selects the `get_weather` tool.
5. LLM extracts the city name: New York.
6. LLM calls the tool with the required input.
7. Tool executes the backend API/function.
8. Tool returns the weather information.
9. LLM processes the result and generates a natural-language response.

**Important:** The LLM decides which tool to call. The developer implements the actual functionality of the tool.

### Tool implementation

LangChain provides the `tool()` function.

A tool generally has two important arguments:

1. Function – actual implementation.
2. Metadata – information describing the tool.

Example:

```typescript
import { tool } from "langchain";
import z from "zod";

const getWeather = tool(
  (input) => {
    // In a real application, call the weather API.
    return `It's sunny in ${input.city}`;
  },
  {
    name: "get_weather",
    description: "Get the weather for a given city",
    schema: z.object({
      city: z.string()
    })
  }
);
```

### Understanding Tool Metadata

| Property | Purpose |
|---|---|
| name | Unique tool name |
| description | Explains what the tool does |
| schema | Defines expected input parameters |

**Description is extremely important.**

The LLM reads the tool description to decide whether the tool is suitable for a particular user query.

For example:

- `get_weather`: Get weather for a given city.
- `get_time`: Get the current time for a given city.

If the user asks for weather, the LLM should select `get_weather`, not `get_time`.

### Understanding Zod Schema

Zod is a TypeScript-first schema validation library.

It defines the structure and data types of the input expected by a tool.

Example:

```typescript
schema: z.object({
  city: z.string()
})
```

This tells the LLM that:

- Input must be an object.
- It must contain a `city` property.
- The city must be a string.

Example input:

```json
{
  "city": "New York"
}
```

Zod also validates the input received by the tool.

### Registering Tools with an Agent

Once tools are implemented, register them with the agent.

```typescript
const agent = createAgent({
  model: "claude-sonnet-4-5-20250929",
  tools: [getWeather, getTime]
});
```

We can register multiple tools in an array.

The LLM can decide which tool or combination of tools is required.

---

## 2. Multiple Tools in One Agent

An agent can have multiple tools.

Example:

- `get_weather`
- `get_time`

### Scenario 1: User asks for weather

Prompt:

"What is the weather in New York?"

Expected execution:

```text
User
  |
  v
LLM
  |
  v
get_weather()
  |
  v
Weather API
  |
  v
Tool Response
  |
  v
LLM
  |
  v
Final Response
```

The LLM selects only `get_weather`.

### Scenario 2: User asks for time

Prompt:

"What is the time in New York?"

The LLM selects `get_time`.

It does not need to call `get_weather`.

### Scenario 3: User asks for both

Prompt:

"What is the weather and time in New York?"

The LLM identifies that two tools are required.

Execution:

1. Calls `get_weather`.
2. Calls `get_time`.
3. Receives both results.
4. Combines the results.
5. Returns the final response.

**Key takeaway:** Tool selection and orchestration are handled by the LLM based on the tool descriptions, schemas and available context.

The developer does not need to manually write an if-else condition for every possible user question.

---

## 3. Runtime Configuration

### What is Runtime Configuration?

Runtime configuration allows us to pass additional contextual information to an agent when invoking it.

This information can dynamically change how tools behave.

### Why do we need it?

Consider a weather application where users can register and log in.

During registration, users provide their location.

Later, a user asks:

"What is the weather outside?"

The user does not provide a city.

However, our application already knows the logged-in user's identity.

We need to:

1. Identify the logged-in user.
2. Retrieve their user ID.
3. Fetch their location from the database.
4. Pass the location to the weather tool.
5. Retrieve the weather.
6. Return the result.

This is where runtime configuration becomes useful.

### Example: User Location Tool

```typescript
const getUserLocation = tool(
  (_, config) => {
    const userId = config.context.user_id;

    // In a real application, fetch location
    // from the database using userId.

    return userId === "1"
      ? "Florida"
      : "San Francisco";
  },
  {
    name: "get_user_location",
    description: "Retrieve user location based on user ID",
    schema: z.object({})
  }
);
```

Notice two things:

**First argument:**

`_` represents tool input.

We use `_` because this tool does not require any input from the LLM.

**Second argument:**

`config` contains runtime configuration.

We access:

```typescript
config.context.user_id
```

This allows the tool to identify the logged-in user.

### Passing Runtime Configuration

```typescript
const config = {
  context: {
    user_id: "1"
  },
  db: {}
};
```

Another configuration:

```typescript
const qaConfig = {
  context: {
    user_id: "3"
  },
  db: {}
};
```

Invoke the agent:

```typescript
const response = await agent.invoke(
  {
    messages: [
      {
        role: "user",
        content: "What is the weather outside?"
      }
    ]
  },
  config
);
```

### Important: Configuration must be passed during invocation

Declaring a configuration object separately is not enough.

You must pass it while invoking the agent.

The configuration becomes available to tools through their runtime configuration argument.

### Runtime Configuration Use Cases

| Use case | Example |
|---|---|
| User context | Logged-in user ID |
| Database selection | Production / QA database |
| Authentication | User/session context |
| Tenant identification | Multi-tenant applications |
| Environment selection | DEV / QA / PROD |
| Application context | User preferences |

### Production vs QA Example

```typescript
const prodConfig = {
  context: {
    user_id: "1"
  },
  db: prodDatabase
};

const qaConfig = {
  context: {
    user_id: "3"
  },
  db: qaDatabase
};
```

The same agent and tools can work with different runtime configurations.

**Important:** Passing a database object does not automatically switch database connections. Your tool implementation must explicitly use the supplied database configuration.

---

## 4. Multiple Tools with Runtime Configuration

Now combine the previous concepts.

We have two tools:

1. `get_user_location`
2. `get_weather`

The user asks:

"What is the weather outside?"

The user has not provided a city.

### How does the agent solve this?

Step 1: LLM analyzes the question.

Step 2: It realizes that weather information requires a location.

Step 3: It checks the available tools.

Step 4: It identifies `get_user_location`.

Step 5: It invokes the location tool without additional input.

Step 6: The tool reads `user_id` from runtime configuration.

Step 7: It retrieves the user's location.

Step 8: The LLM receives the location.

Step 9: It invokes `get_weather` with the retrieved city.

Step 10: The weather tool returns the weather.

Step 11: The LLM generates the final response.

Example:

```text
User: What is the weather outside?
              |
              v
        Location Tool
              |
              v
       User ID = 1
              |
              v
       Location = Florida
              |
              v
        Weather Tool
              |
              v
        Sunny in Florida
              |
              v
         Final Answer
```

**Important concept:** The LLM can reason about dependencies between tools. It can recognize that it needs the user's location before it can retrieve the weather.

---

## 5. System Prompts

### What is a System Prompt?

A system prompt defines the agent's role, responsibilities, behavior and instructions.

It provides context to the LLM about how it should behave.

Without a system prompt, the LLM still has general reasoning capabilities, but it may not follow the specific business rules or workflow expected by our application.

### Example

```typescript
const systemPrompt = `
You are an expert weather forecaster.

You have access to two tools:

- get_weather_for_location:
  Use this to get the weather for a specific location.

- get_user_location:
  Use this to get the user's location.

If a user asks you for the weather,
make sure you know the location first.

If the user means their current location,
use get_user_location to find their location.
`;
```

Register it with the agent:

```typescript
const agent = createAgent({
  model: "claude-sonnet-4-5-20250929",
  tools: [getUserLocation, getWeather],
  systemPrompt
});
```

### Why do we need System Prompts?

Imagine an agent with 10–12 tools.

The agent might need to follow a specific sequence of actions.

Instead of relying entirely on the LLM to figure out the workflow every time, we can guide it through a system prompt.

For example:

1. Always identify the user's location first.
2. Retrieve weather only after identifying the location.
3. Never invent weather information.
4. Use the weather tool for actual weather data.
5. Return the result in a user-friendly format.

### System Prompt vs Tool Description

| System Prompt | Tool Description |
|---|---|
| Defines agent behavior | Defines individual tool functionality |
| Provides overall context | Explains when a tool should be used |
| Defines rules and workflow | Defines the tool's purpose |
| Applies to the agent | Applies to a particular tool |

**Key takeaway:** System prompts guide the agent; tool descriptions help the LLM select the appropriate tools.

---

## 6. Structured Output / Response Format

### What is Structured Output?

By default, an LLM generally returns a natural-language response.

However, sometimes our application needs the response in a specific JSON structure.

For example, instead of receiving:

"It's sunny in Florida. Looks like a beautiful day!"

We might want:

```json
{
  "humour_response": "Looks like Mother Nature is showing off!",
  "weather_conditions": "Sunny"
}
```

This is called structured output.

### Why do we need it?

Structured responses are useful when:

- Frontend needs specific fields.
- Backend needs to process individual values.
- We need predictable response structures.
- We want to display different information in different UI components.
- We need to store the response in a database.

### Defining Response Schema with Zod

```typescript
const responseFormat = z.object({
  humour_response: z.string(),
  weather_conditions: z.string()
});
```

This tells the agent that the response must contain:

| Property | Type |
|---|---|
| humour_response | String |
| weather_conditions | String |

### Registering Response Format

```typescript
const agent = createAgent({
  model,
  tools: [getUserLocation, getWeather],
  systemPrompt,
  responseFormat
});
```

### Accessing Structured Response

```typescript
const response = await agent.invoke({
  messages: [
    {
      role: "user",
      content: "What is the weather outside?"
    }
  ]
}, config);

console.log(response.structuredResponse);
```

The structured response can be accessed separately from the complete agent execution history.

**Important:** Structured output defines the expected response format. It does not replace tools or system prompts.

---

## 7. Model Configuration – initChatModel()

Until now, we have configured the agent.

However, we can also customize the underlying LLM.

LangChain provides `initChatModel()` to initialize a chat model with additional configuration.

### Example

```typescript
const model = await initChatModel(
  "claude-sonnet-4-5-20250929",
  {
    temperature: 0.7,
    timeout: 30,
    max_tokens: 1000
  }
);
```

### Important Model Parameters

| Parameter | Purpose |
|---|---|
| temperature | Controls response randomness |
| timeout | Controls request timeout |
| max_tokens | Limits generated output tokens |

### 1. Temperature

Temperature controls the randomness of the model's responses.

- Lower temperature: More predictable responses.
- Higher temperature: More varied and creative responses.

Example:

```typescript
temperature: 0.1
```

Useful for deterministic tasks, such as extracting information.

```typescript
temperature: 0.7
```

Useful for more conversational responses.

**Correction:** Temperature does not directly control response length.

### 2. Timeout

```typescript
timeout: 30
```

Sets the request timeout according to the model integration's supported timeout units.

The request may fail if the model takes longer than the configured limit.

### 3. Max Tokens

```typescript
max_tokens: 1000
```

Limits the maximum number of generated output tokens, subject to the model provider's supported configuration.

Tokens are units of text processed by LLMs.

They are not exactly equivalent to characters or words.

A token can represent a whole word, part of a word or punctuation.

### Using the Configured Model

```typescript
const agent = createAgent({
  model,
  tools: [getUserLocation, getWeather],
  systemPrompt,
  responseFormat
});
```

**Key takeaway:**

- Model configuration controls LLM-level behavior.
- System prompt controls agent-level instructions.
- Tools provide external capabilities.
- Response format controls output structure.

---

## 8. Memory Management

### What is Memory in LangChain?

Memory allows an agent to retain conversation context across multiple interactions.

Consider this conversation:

User: What is the weather outside?

AI: It is sunny in Florida.

User: Which location did you just mention?

AI: Florida.

User: Suggest some good places to eat there.

AI: The agent can understand that "there" refers to Florida.

Without conversation history, the LLM may not understand what "there" refers to.

### Why do we need Memory?

Memory helps with:

1. Maintaining conversation context.
2. Understanding follow-up questions.
3. Remembering previous responses.
4. Handling references such as "that city", "there" and "the previous one".
5. Supporting multi-turn conversations.

### MemorySaver

In the course, the instructor uses `MemorySaver` from LangGraph.

```typescript
import { MemorySaver } from "@langchain/langgraph";

const checkpointer = new MemorySaver();
```

This creates an in-memory checkpoint store.

It allows the agent to save and retrieve conversation state using thread IDs.

### Registering Checkpointer

```typescript
const agent = createAgent({
  model,
  tools: [getUserLocation, getWeather],
  systemPrompt,
  responseFormat,
  checkpointer
});
```

The checkpointer enables conversation-state persistence across invocations of the same thread.

### Understanding Thread ID

A thread ID identifies a particular conversation.

Example:

```typescript
const config = {
  configurable: {
    thread_id: "1"
  },
  context: {
    user_id: "1"
  }
};
```

Another thread:

```typescript
const qaConfig = {
  configurable: {
    thread_id: "2"
  },
  context: {
    user_id: "3"
  }
};
```

### How Thread ID Works

**Scenario 1: Same Thread ID**

```typescript
thread_id: "1"
```

If we invoke the agent multiple times using the same thread ID, the checkpointer retrieves the previous conversation state.

Example:

```text
Thread ID: 1

Message 1:
What is the weather outside?

Message 2:
Which location did you mention?

Message 3:
Suggest good restaurants there.
```

The agent can use the earlier messages to understand the follow-up questions.

**Scenario 2: Different Thread ID**

```typescript
thread_id: "2"
```

A different thread starts with a separate conversation history.

The agent will not automatically have access to the previous thread's messages.

### Memory Execution Flow

```text
User Message
     |
     v
Agent Invocation
     |
     v
Read Thread ID
     |
     v
Load Previous State
     |
     v
Process New Message
     |
     v
Generate Response
     |
     v
Save Updated State
     |
     v
Return Response
```

### Important: MemorySaver Limitations

`MemorySaver` stores checkpoint data in memory.

It is suitable for learning and development, but it is not durable storage.

For production applications, use an appropriate persistent checkpointer, such as a database-backed checkpointer.

Also remember:

- Thread ID must remain consistent for the same conversation.
- Different users should have appropriately isolated threads.
- Memory does not automatically mean permanent user-profile memory.
- Conversation state and long-term user memory are different concepts.

---

# 9. Complete Conceptual Architecture

By combining everything we learned, we can build a more capable AI agent.

```text
                 USER
                   |
                   v
             User Question
                   |
                   v
              LANGCHAIN
                AGENT
                   |
          +--------+--------+
          |        |        |
          v        v        v
        MODEL   SYSTEM   MEMORY
                PROMPT  CHECKPOINTER
          |
          v
      TOOL SELECTION
          |
     +----+----------+
     |               |
     v               v
  LOCATION         WEATHER
    TOOL             TOOL
     |               |
     v               v
  USER DB         WEATHER API
     |               |
     +-------+-------+
             |
             v
        TOOL RESULTS
             |
             v
        LLM PROCESSING
             |
             v
       STRUCTURED OUTPUT
             |
             v
        FRONTEND RESPONSE
```

# 10. Quick Revision – Important Concepts

| Concept | What it does |
|---|---|
| Tool | Executes external actions |
| Zod | Defines and validates schemas |
| Tool Description | Helps LLM select the correct tool |
| Tool Schema | Defines expected tool input |
| Runtime Config | Passes dynamic context to tools |
| System Prompt | Defines agent role and behavior |
| Structured Output | Returns data in a predefined structure |
| initChatModel | Initializes/configures the LLM |
| Temperature | Controls response randomness |
| Max Tokens | Limits generated output |
| MemorySaver | Saves conversation checkpoints in memory |
| Thread ID | Identifies a conversation |

# 11. Interview Questions

### Q1. What are Tools in LangChain?

Tools are functions that allow an LLM agent to interact with external systems such as APIs, databases and internal application services.

### Q2. Why do we need tools if an LLM can already answer questions?

An LLM can generate responses using its existing knowledge, but it cannot automatically access private application data or execute arbitrary backend operations. Tools provide that capability.

### Q3. How does an LLM decide which tool to execute?

It analyzes the user query and uses the available tool names, descriptions and input schemas to select the appropriate tool.

### Q4. What is the purpose of Zod?

Zod is used to define and validate input and output schemas. It ensures data follows the expected structure and types.

### Q5. What is Runtime Configuration?

Runtime configuration allows us to pass dynamic information, such as user ID, database connection or application context, during agent invocation.

### Q6. What is the difference between a System Prompt and a Tool Description?

A system prompt defines the overall behavior and responsibilities of an agent. A tool description explains the purpose of an individual tool.

### Q7. What is Structured Output?

Structured output allows an LLM to return responses in a predefined schema, such as JSON with specific properties and data types.

### Q8. What is the purpose of Temperature?

Temperature controls the randomness of the model's output. Lower values produce more predictable responses, while higher values allow more variation.

### Q9. What is MemorySaver?

MemorySaver is a LangGraph checkpointer that stores conversation checkpoints in memory, allowing an agent to maintain state across multiple invocations.

### Q10. What is Thread ID?

A thread ID uniquely identifies a conversation. It allows the checkpointer to retrieve the appropriate conversation state for subsequent interactions.

### Q11. What is the difference between Runtime Configuration and Memory?

Runtime configuration passes contextual information to the agent at invocation time. Memory stores and retrieves conversation state across invocations.

### Q12. Can one agent use multiple tools?

Yes. An agent can have multiple registered tools and can invoke one or more tools depending on the user's request.

---

# 12. Practical Implementation Exercise

Build a small AI-powered weather assistant.

**Requirements:**

- Use Claude as the LLM.
- Create a `get_weather` tool.
- Create a `get_user_location` tool.
- Use Zod for tool schemas.
- Pass user ID through runtime configuration.
- Add a system prompt.
- Return structured output containing weather conditions and a human-friendly message.
- Implement MemorySaver.
- Support separate conversation threads.

**Suggested project structure:**

```text
langchain-weather-agent/
│
├── src/
│   ├── tools/
│   │   ├── weather.tool.ts
│   │   └── user-location.tool.ts
│   │
│   ├── agents/
│   │   └── weather.agent.ts
│   │
│   ├── config/
│   │   └── model.config.ts
│   │
│   └── index.ts
│
├── .env
└── package.json
```

**Learning objective:** Understand how an LLM, tools, runtime configuration, system prompts, structured output and memory work together to build an AI agent.

---

## Final Takeaway

The most important thing to understand from this section is that an AI agent is not just an LLM generating text.

It combines several capabilities:

1. **LLM:** Reasoning and decision-making.
2. **Tools:** Performing actual actions.
3. **Runtime Configuration:** Providing dynamic application context.
4. **System Prompt:** Defining responsibilities and rules.
5. **Structured Output:** Producing predictable data.
6. **Memory:** Maintaining conversation state.

Together, these components allow us to build AI applications that can interact with our own backend systems and perform meaningful tasks.
