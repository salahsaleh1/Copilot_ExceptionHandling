# Exception Handling in Copilot Studio
## A Practical Guide to Building Reliable Agents

---

## Introduction

When building agents in **Microsoft Copilot Studio**, failures are unavoidable. APIs may be unavailable, data may be incomplete, permissions can break, or Power Automate flows might fail unexpectedly.

Exception handling is not just a technical requirement — it directly impacts:
- User trust
- Conversation quality
- Maintainability
- Supportability of your agent

A well-designed agent should fail **gracefully**, communicate clearly, and provide enough information for developers to diagnose issues quickly.

This guide explains how to implement effective exception handling strategies in Copilot Studio using connectors, flows, and tools.

---

## Why Exception Handling Matters

Copilot Studio agents frequently depend on:
- External APIs
- Dataverse or SharePoint data
- Power Automate flows
- Authentication and network connectivity

Without proper exception handling:
- Topics may terminate suddenly
- Users may receive confusing or generic messages
- Developers lack context when troubleshooting issues

Good exception handling ensures:
- Clear and friendly user messaging
- Predictable agent behavior
- Detailed logging for support teams
- Better long-term reliability

---

## Handling Errors in Connector Calls

Connectors are often used inside topics to fetch or send data. These calls can fail for several reasons:
- Timeouts
- Authorization issues
- Invalid input
- Unexpected response formats

Copilot Studio provides **two main ways** to handle connector failures.

---

## Option 1: Raise an Error (Fail Fast)

This is the default and recommended approach for **critical failures**.

<img width="353" height="492" alt="image" src="https://github.com/user-attachments/assets/8ba0f534-d794-4589-9b02-f6e5303dc93b" />


### How it Works
- If the connector fails, the topic execution stops
- Control is transferred to the system **On Error** topic
- You can present a customized message to the user

<img width="1146" height="628" alt="image" src="https://github.com/user-attachments/assets/189993e2-fdb3-4d31-adbc-495a48879f07" />


### Best Practices
- Show a clear, user-friendly message
- Notify your team whenever an error occurs.
- Choose an appropriate notification method:
  - Send a message to a Microsoft Teams channel
  - Send an email
  - Create a ServiceNow incident
- Prompt alerting enables the team to begin troubleshooting quickly.


### Advanced Tip
- If you have sufficient Copilot Studio credits, you can enhance exception handling by:
  - Sending the support team a brief summary of the issue
  - Including suggestions for how to resolve it

- To implement this approach:
  - Save the connector name and action name in a global variable before the action runs
  - This allows you to reference them later in the **On Error** topic
    
 ````
"The connector action named " & Global.sAction & " has thrown an error during execution.
The error code is: " & System.Error.Code & "
The error message is: " & System.Error.Message & "

Based on this information, generate a concise and professional summary suitable for a support team email. Please include:
- A brief explanation of the issue
- Suggested troubleshooting steps
- Relevant documentation links or keywords to investigate further"
````

- In the **On Error** topic:
  - Before sending the email, add a **Create generative answers** node
  - Provide the required input to this node
  - Uncheck the **Send a message** option so the agent does not post the response in the chat
  - Store the generated response in a variable

- Since the response will be sent via email:
  - Update the prompt to generate the output in **HTML** instead of Markdown
  - Use the generated HTML content as the body of the exception email
<img width="650" height="791" alt="image" src="https://github.com/user-attachments/assets/6449f7d7-2a4f-4463-a844-93c9d8d7a300" />

---

## Application Insights

If your bot is already connected to **Azure Application Insights**, or you plan to connect it, you should log a **custom telemetry event** to capture error details. This makes it easier to track and analyse issues over time.

### Custom Telemetry Event Properties

The **Custom Telemetry Event Properties** menu allows you to:

- Define the **event name**
- Specify the event **properties** in **JSON format**

<img width="581" height="240" alt="image" src="https://github.com/user-attachments/assets/6d850412-bd32-441a-a7d6-d6277b717d58" />


```json
{
  "propertyName": "propertyValue",
  "errorCode": "string",
  "description": "string"
}
```

---

## Option 2: Continue on Error (Graceful Degradation)

This approach allows the topic to continue even if the connector fails.

### How it Works
- The connector does not interrupt the topic flow
- The agent can inspect:
  - HTTP status codes
  - Response payloads
- Logic can branch based on the error type

### When to Use It
- Non-critical data lookups
- Optional enrichment steps
- Situations where fallback behavior is acceptable

### Best Practices
- Display a friendly message explaining the limitation
- Offer retry options or alternative paths
- Log error details silently for support teams

This method helps maintain conversational flow while still handling failures responsibly.

---

## Handling Errors in Power Automate Flows Used in Topics

Flows called from Copilot Studio topics behave similarly to connectors when errors occur.

### Recommended Pattern

1. Wrap flow logic inside a **Scope**
2. Add a parallel **Exception Scope**
3. Use condition-based branching to detect failures
4. Return a structured response to the agent

### Key Considerations
- Always return a success or failure flag
- Include meaningful error messages
- Pass the conversation ID for traceability

The agent should never need to guess whether a flow succeeded.

---

## Using Flows as Tools

When Power Automate flows are used as **tools**, error handling becomes even more important.

### Best Practices
- Clearly define output contracts
- Include explicit error fields in responses
- Avoid relying on implicit failures

### Example Output Structure
- `status`: success / failure
- `message`: human-readable explanation
- `errorCode`: optional technical identifier

This allows the agent to interpret results accurately and respond appropriately.

---

## Designing a Consistent Error Strategy

To build production-ready agents:
- Use consistent error patterns across topics
- Standardize user-facing error messages
- Centralize logging and notifications
- Treat failures as first-class design scenarios

Planning for failure is a key part of designing reliable conversational agents.

---

## Conclusion

Exception handling is not an afterthought — it is a core design principle in Copilot Studio.

By:
- Choosing the right error-handling strategy
- Designing flows defensively
- Communicating clearly with users
- Logging meaningful diagnostics

You can build agents that are resilient, trustworthy, and easy to maintain.

A well-handled failure is often invisible to the user — and that is exactly the goal.
