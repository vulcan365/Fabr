# Fabr

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![NuGet](https://img.shields.io/nuget/v/Fabr.Core.svg)](https://www.nuget.org/packages/Fabr.Core)

**An Orleans-based framework for building distributed AI agent systems in .NET.**

Fabr provides the building blocks for creating, hosting, and connecting to AI agents that run as [Orleans](https://learn.microsoft.com/en-us/dotnet/orleans/) grains. Agents are durable, scalable, and communicate through a structured message-passing architecture.

## Architecture

```
┌─────────────────────────────────────────────────┐
│                  Your Application                │
├──────────────────┬──────────────────────────────┤
│   Fabr.Client    │         Fabr.Host            │
│  ClientContext   │   AgentGrain (Orleans)       │
│  ChatDock (UI)   │   API Controllers            │
│                  │   WebSocket Middleware        │
├──────────────────┴──────────────────────────────┤
│                   Fabr.Sdk                       │
│  FabrAgentProxy · TaskWorkingAgent · ChatClient  │
├─────────────────────────────────────────────────┤
│                  Fabr.Core                        │
│  Interfaces · Data Models · Grain Abstractions   │
└─────────────────────────────────────────────────┘
```

## Packages

| Package | Description |
|---------|-------------|
| **Fabr.Core** | Core interfaces and data models for the Fabr agent framework |
| **Fabr.Sdk** | SDK for building AI agents — `FabrAgentProxy`, `TaskWorkingAgent`, chat client extensions |
| **Fabr.Host** | Orleans server host — grains, API controllers, WebSocket middleware, streaming |
| **Fabr.Client** | Client library — `ClientContext`, `ChatDock` Blazor component |

## Quick Start

### 1. Install packages

```bash
dotnet add package Fabr.Host
```

`Fabr.Host` pulls in `Fabr.Sdk` and `Fabr.Core` transitively. For client applications, add `Fabr.Client` instead.

### 2. Create an agent

```csharp
using Fabr.Sdk;
using Fabr.Core;

[AgentAlias("my-assistant")]
public class MyAssistantAgent : FabrAgentProxy
{
    public override async Task<AgentMessage> OnMessage(AgentMessage message)
    {
        // Create an AI-powered chat agent
        var (agent, session) = await Host.CreateChatClientAgent(
            modelName: "AzureProd",
            instructions: "You are a helpful assistant."
        );

        var response = await agent.SendAsync(session, message.Text);
        return message.ToReply(response);
    }
}
```

### 3. Configure the host

```csharp
var builder = WebApplication.CreateBuilder(args);

// Configure Fabr with Orleans
builder.AddFabrServer();

var app = builder.Build();
app.UseFabrServer();
app.Run();
```

### 4. Configure model access

Copy `fabr.json.example` to `fabr.json` in your project root and fill in your API keys:

```json
{
  "ModelConfigurations": [
    {
      "Name": "AzureProd",
      "Provider": "Azure",
      "Uri": "https://your-resource.cognitiveservices.azure.com/",
      "Model": "gpt-4.1-mini",
      "ApiKeyAlias": "AZURE_KEY"
    }
  ],
  "ApiKeys": [
    {
      "Alias": "AZURE_KEY",
      "Value": "your-api-key-here"
    }
  ]
}
```

> **Note:** `fabr.json` is gitignored by default to prevent accidental secret commits.

## Configuration

### Orleans Clustering

Fabr supports multiple Orleans clustering providers out of the box:

- **Azure Storage** — `Microsoft.Orleans.Clustering.AzureStorage`
- **SQL Server (ADO.NET)** — `Microsoft.Orleans.Clustering.AdoNet`
- **Localhost** — for development

Configure clustering through standard Orleans configuration in your host's `appsettings.json`.

### WebSocket API

Fabr includes WebSocket middleware for real-time agent communication. See the [WebSocket documentation](src/Fabr.Host/WebSocket/README.md) for protocol details and usage.

### Client Library

The `Fabr.Client` package provides `ClientContext` for connecting to agents and a `ChatDock` Blazor component for building chat UIs. See the [Client documentation](src/Fabr.Client/README.md) for details.

## Building from Source

```bash
dotnet build src/Fabr.sln
```

## License

Licensed under the [Apache License, Version 2.0](LICENSE).

See [NOTICE](NOTICE) for attribution requirements.

## Contributing

Contributions are welcome! Please open an issue or pull request on [GitHub](https://github.com/vulcan365/Fabr).
