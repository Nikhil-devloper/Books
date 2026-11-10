# LangChain Middleware -- Complete Study Notes

> Based on Videos 1--4 and the three code examples. These notes cover
> concepts, implementation patterns, practical use cases, and important
> corrections.

## 1. What is Middleware in LangChain?

Middleware is an interception and customization layer in an agent's
execution flow. It lets you inspect, modify, or control model requests
and other agent operations without rewriting the core agent logic.

### Why use middleware?

-   Choose a cheaper or more capable model depending on the task.
-   Recover when a model fails.
-   Manage long conversations and token usage.
-   Reduce the number of tools presented to a model.
-   Detect and redact sensitive information.

## 2. Dynamic Model Selection Middleware

**Purpose:** Choose a model at runtime based on a condition.

Example: - Simple queries → GPT-4o-mini. - More complex or longer
conversations → Claude Sonnet.

### Flow

1.  User sends a message.
2.  Middleware intercepts the model request.
3.  It checks a condition, such as the number of messages.
4.  It passes the request to the selected model.
5.  The selected model generates the response.

### Example

``` typescript
const dynamicModelSelection = createMiddleware({
  name: "DynamicModelSelection",

  wrapModelCall: (request, handler) => {
    const messageCount = request.messages.length;

    return handler({
      ...request,
      model: messageCount > 3 ? model : basicModel
    });
  }
});
```

### Key elements

  Element                  Meaning
  ------------------------ -----------------------------------------------
  `createMiddleware()`     Creates custom middleware
  `wrapModelCall`          Intercepts a model call
  `request`                Contains details of the model request
  `request.messages`       Messages available to the model call
  `handler`                Continues execution with the supplied request
  `...request`             Preserves existing request properties
  `model` / `basicModel`   Advanced and cheaper model instances

Register the middleware with the agent:

``` typescript
const agent = createAgent({
  model,
  tools: [getUserLocation, getWeather],
  systemPrompt,
  checkpointer,
  middleware: [dynamicModelSelection]
});
```

If middleware is not registered with `createAgent`, the agent will not
use it.

### Other possible selection conditions

-   Query complexity.
-   Keywords in the user's prompt.
-   User subscription tier.
-   Token consumption.
-   Task category.
-   Latency or cost requirements.

**Use case:** A customer-support agent can use a cheaper model for FAQs
and a more capable model for complicated billing disputes.

## 3. Model Fallback Middleware

**Purpose:** Try an alternative model when the primary model fails with
an eligible error.

Possible failure situations include provider outages, rate limits, or
other retryable model errors.

### Flow

``` text
Primary model: GPT-4o
        |
        | eligible failure
        v
Fallback model 1: GPT-4o-mini
        |
        | eligible failure
        v
Fallback model 2: GPT-3.5-turbo
```

### Example

``` typescript
modelFallbackMiddleware(
  "gpt-4o-mini",
  "gpt-3.5-turbo"
)
```

Register it:

``` typescript
const agent = createAgent({
  model: "gpt-4o",
  tools: [searchTool, emailTool, getWeather],
  middleware: [
    modelFallbackMiddleware(
      "gpt-4o-mini",
      "gpt-3.5-turbo"
    )
  ]
});
```

### Dynamic selection vs fallback

  -----------------------------------------------------------------------
  Dynamic Model Selection             Model Fallback
  ----------------------------------- -----------------------------------
  Chooses a model based on a          Switches to an alternative after an
  condition                           eligible failure

  Primarily supports routing and cost Primarily supports resilience
  optimization                        

  Example: simple vs complex query    Example: primary provider
                                      unavailable
  -----------------------------------------------------------------------

**Remember:** Fallback behavior depends on the errors considered
eligible and on provider/model compatibility. It does not guarantee
recovery from every failure.

## 4. Summarization Middleware

**Purpose:** Manage long conversation histories by summarizing older
context and retaining recent messages.

### Why is it needed?

As a conversation grows: - More historical messages are included in
context. - Token usage can increase. - Requests may become more
expensive. - The model's context window can eventually be exceeded.

### Example configuration

``` typescript
summarizationMiddleware({
  model: "gpt-4o",
  maxTokensBeforeSummary: 8000,
  messagesToKeep: 20
})
```

### Properties

  -----------------------------------------------------------------------
  Property                            Meaning
  ----------------------------------- -----------------------------------
  `model`                             Model used for summarization

  `maxTokensBeforeSummary`            Token threshold that triggers
                                      summarization

  `messagesToKeep`                    Number of recent messages to retain
  -----------------------------------------------------------------------

### Example

Assume: - Illustrative context budget: 10,000 tokens. - Summarization
threshold: 8,000 tokens. - Recent messages to retain: 20.

When the configured threshold is reached, older conversation history can
be summarized. The agent retains the summary and the configured recent
messages, helping it continue without repeatedly carrying the entire
original history.

### Real-world example

A coding assistant may have discussed: - Project architecture. -
Database schema. - API endpoints. - Authentication. - Deployment.

Summarization can preserve important decisions from earlier discussion
while keeping recent interactions available.

**Important:** Summarization is not simply deleting the first 80
messages. It creates a summary of older context and retains recent
messages according to the middleware's behavior. A token threshold is
not the same as a subscription quota or billing limit. Summarization
also does not automatically force shorter answers.

## 5. LLM Tool Selector Middleware

**Purpose:** Shortlist tools that are relevant to a user query,
especially when an agent has many tools.

### The problem

Imagine an agent has 50 tools. The main model may need to consider many
tool descriptions to decide which tools are relevant. This can add token
overhead.

### Proposed approach

1.  Use a smaller, cheaper model to inspect the user query and tool
    descriptions.
2.  Shortlist the most relevant tools.
3.  Give the selected tools to the main model.
4.  Let the main model perform reasoning and tool execution.

### Flow

``` text
User query
    |
    v
Available tools (e.g. 50)
    |
    v
Smaller model selects relevant tools
    |
    v
Shortlist (e.g. 2 tools)
    |
    v
Main model reasons and executes tools
    |
    v
Final response
```

### Example

``` typescript
llmToolSelectorMiddleware({
  model: "gpt-4o-mini",
  maxTools: 2
})
```

  Property     Meaning
  ------------ --------------------------------------
  `model`      Model used to select relevant tools
  `maxTools`   Maximum number of tools to shortlist

**Important:** Tool selection is not tool execution. The selector
narrows the available tools; the main agent still handles reasoning and
execution. A smaller selector model may reduce overhead, but actual
savings depend on the number of tools, prompts, and model costs.

## 6. PII Redaction Middleware

**PII** means Personally Identifiable Information: information that can
identify an individual.

Examples: - Credit card numbers. - Phone numbers. - Social Security
Numbers (SSNs). - Email addresses. - Personal addresses.

**Purpose:** Detect and redact sensitive information before it is passed
to downstream model processing, according to the configured rules and
middleware behavior.

### Example

``` typescript
piiRedactionMiddleware({
  rules: {
    credit_card:
      /\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/g,

    ssn:
      /\b\d{3}-\d{2}-\d{4}\b/g,

    phone:
      /\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/g
  }
})
```

### Example patterns

  Rule            Example
  --------------- -----------------------
  `credit_card`   `4532-1234-5678-9010`
  `ssn`           `123-45-6789`
  `phone`         `555-123-4567`

Illustrative transformation:

``` text
Before:
My SSN is 123-45-6789 and my phone number is 555-123-4567.

After:
My SSN is [REDACTED] and my phone number is [REDACTED].
```

### Security considerations

-   Regex detection is not foolproof.
-   Email addresses need their own rule if they must be redacted.
-   Test multiple input formats and edge cases.
-   Protect logs, tool arguments, and other downstream paths too.
-   Do not assume redaction alone makes an application compliant.

## 7. Middleware comparison

  -----------------------------------------------------------------------
  Middleware              Main responsibility     Main benefit
  ----------------------- ----------------------- -----------------------
  Dynamic Model Selection Select model based on   Cost and routing
                          conditions              optimization

  Model Fallback          Try another model after Reliability
                          eligible failures       

  Summarization           Summarize older         Context management
                          conversation history    

  LLM Tool Selector       Shortlist relevant      Tool-selection
                          tools                   efficiency

  PII Redaction           Detect and redact       Privacy protection
                          sensitive data          
  -----------------------------------------------------------------------

### Which middleware fits which scenario?

  Scenario                                    Middleware
  ------------------------------------------- -------------------------
  Simple queries should use a cheaper model   Dynamic Model Selection
  Primary model fails                         Model Fallback
  Conversation becomes long                   Summarization
  Agent has dozens of tools                   LLM Tool Selector
  User enters sensitive information           PII Redaction

## 8. Combining middleware in an agent

The following combines the middleware patterns discussed in the lessons.
Confirm the exact imports, configuration options, ordering, and
compatibility against the LangChain version installed in your project.

``` typescript
import {
  createAgent,
  llmToolSelectorMiddleware,
  modelFallbackMiddleware,
  piiRedactionMiddleware,
  summarizationMiddleware
} from "langchain";

const agent = createAgent({
  model: "gpt-4o",

  tools: [
    searchTool,
    emailTool,
    getWeather
  ],

  middleware: [
    modelFallbackMiddleware(
      "gpt-4o-mini",
      "gpt-3.5-turbo"
    ),

    summarizationMiddleware({
      model: "gpt-4o",
      maxTokensBeforeSummary: 8000,
      messagesToKeep: 20
    }),

    llmToolSelectorMiddleware({
      model: "gpt-4o-mini",
      maxTools: 2
    }),

    piiRedactionMiddleware({
      rules: {
        credit_card:
          /\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/g
      }
    })
  ]
});
```

Middleware execution order and hook behavior matter. Do not assume every
middleware runs in simple top-to-bottom order for every hook. In
particular, verify that sensitive information is redacted before it can
reach any model or tool.

## 9. Supporting LangChain concepts from the code

### A. `createAgent()`

Creates an agent using a model, tools, optional system prompt,
checkpointer, and middleware.

``` typescript
const agent = createAgent({
  model,
  tools: [getUserLocation, getWeather],
  systemPrompt,
  checkpointer,
  middleware: [dynamicModelSelection]
});
```

### B. `createMiddleware()`

Creates custom middleware.

``` typescript
const myMiddleware = createMiddleware({
  name: "MyMiddleware",

  wrapModelCall: (request, handler) => {
    return handler(request);
  }
});
```

### C. `tool()`

Defines a tool that an agent can use.

``` typescript
const searchTool = tool(
  ({ query }) => {
    return `Search results for ${query}`;
  },
  {
    name: "search",
    description: "Search the internet",
    schema: z.object({
      query: z.string()
    })
  }
);
```

The tool function contains the implementation. The name and description
help the model understand the tool. The schema describes the expected
input.

### D. Zod schema

Zod defines and validates the structure of tool inputs.

``` typescript
schema: z.object({
  recipient: z.string(),
  subject: z.string()
})
```

This schema expects `recipient` and `subject` to be strings.

### E. Checkpointer and `thread_id`

Example:

``` typescript
const checkpointer = new MemorySaver();

const config = {
  configurable: {
    thread_id: "1"
  }
};
```

-   A checkpointer saves agent state across interactions.
-   `thread_id` identifies a conversation thread.
-   Reusing the same thread ID allows the agent to continue that thread,
    subject to the checkpointer configuration.

## 10. Important debugging lessons from Video 1

The practical exercise exposed several issues worth remembering:

1.  **API credentials:** A model provider needs the correct API key in
    the environment. Check the variable name and ensure the environment
    file is loaded.
2.  **Save files:** Ensure edits are saved before running the
    application.
3.  **JSON syntax:** A stray comma or other syntax error in
    `package.json` can prevent the project from running.
4.  **Declaration order:** Ensure model instances are initialized before
    middleware uses them.
5.  **Response format compatibility:** The instructor encountered a
    compatibility issue between response formatting and model-request
    middleware in the LangChain version used in the lesson. Removing the
    response format resolved that particular exercise. Treat this as
    version-specific, not a universal limitation.
6.  **Inspect response metadata:** Model response metadata can help
    verify which model handled a request.
7.  **Inspect tool calls:** An agent may call multiple tools before
    producing its final answer. For example, a weather query may first
    retrieve the user's location and then fetch weather for that
    location.

## 11. Corrections to the lecture/code

1.  **Token limit vs subscription limit:** A model context window, API
    usage quota, and billing limit are different things. The
    10,000-token example is illustrative.
2.  **Summarization vs response length:** Summarization manages
    conversation context. It does not inherently force the model to
    produce shorter answers.
3.  **Fallback vs model selection:** Fallback responds to eligible
    failures; dynamic selection routes requests based on conditions.
4.  **Tool selector vs tool execution:** Tool selection narrows the
    available tools. The main agent performs the reasoning and
    execution.
5.  **Middleware order:** Verify version-specific hook ordering and
    interactions instead of assuming array order describes every
    execution stage.
6.  **Email tool return value:** The original Code 2 callback omitted
    `return`. Corrected version:

``` typescript
const emailTool = tool(
  ({ recipient, subject }) => {
    return `Email sent to ${recipient} with subject ${subject}`;
  },
  {
    name: "send_email",
    description: "Send an email to someone",
    schema: z.object({
      recipient: z.string(),
      subject: z.string()
    })
  }
);
```

7.  **Dynamic model initialization:** Ensure `model` and `basicModel`
    are initialized before the middleware is invoked, and that both are
    compatible model instances.

## 12. Interview questions for revision

1.  What is middleware in LangChain?
2.  How does `wrapModelCall` work?
3.  What is the difference between dynamic model selection and model
    fallback?
4.  How can dynamic model selection reduce LLM costs?
5.  What happens when the primary model fails and fallback is
    configured?
6.  What does `maxTokensBeforeSummary` control?
7.  What is the purpose of `messagesToKeep`?
8.  Does summarization simply delete old messages?
9.  Why is an LLM tool selector useful when an agent has many tools?
10. What is the difference between tool selection and tool execution?
11. What does PII stand for?
12. Why is regex-only PII detection insufficient for production
    security?
13. Why must middleware be registered in `createAgent()`?
14. What is the purpose of a checkpointer and `thread_id`?
15. Why should you inspect response metadata while debugging model
    selection?

## 13. Quick revision sheet

Remember these five responsibilities:

1.  **Dynamic Model Selection** → Which model should I use?
2.  **Model Fallback** → What if the primary model fails?
3.  **Summarization** → How do I manage long conversations?
4.  **LLM Tool Selector** → Which tools are relevant?
5.  **PII Redaction** → How do I protect sensitive information?

### Practical learning exercise

Build one small agent and progressively add: 1. Dynamic model selection.
2. Model fallback. 3. Summarization. 4. PII redaction. 5. 10--15 tools
and tool selection.

Test each middleware independently before combining them. Log the
selected model, tool calls, errors, and token usage so you can verify
the actual behavior.

------------------------------------------------------------------------

**Core takeaway:** Middleware helps make LangChain agents more
cost-efficient, reliable, context-aware, and safer. Understand the
problem each middleware solves, then verify its behavior in your
installed LangChain version.
