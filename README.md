# How to Build a Multi-Agent System with Awesome Open Source Agents using Coral Protocol

This guide provides a step-by-step guide to build and run a complete **multi-agent system** using [Coral Protocol](https://github.com/Coral-Protocol), open-source agents, and Coral Studio so you can view all the interactions visually.

## Introduction

### What is Coral?

Coral Protocol provides a collaboration infrastructure for AI agents. It allows agent developers to publish agent advertisements that any other agent or any multi-agent application can immediately use on demand.

Agent developers earn incentives when their agents are used.
Application developers can mix and match from Coral’s growing library of agents to assemble advanced systems faster and without vendor lock-in.

In this scenario, you would be an application developer, using Coral Protocol's local mode to build a multi-agent system by bringing together existing open source agents.


---

## Prerequisites

Before you set up and run Coral, make sure your local environment has the following tools installed.

These are required to run agents, Coral Server, Coral Studio, and external LLMs like OpenAI.

### Required Tools & Versions

| Tool | Version | Why You Need It                                             |
|------|---------|-------------------------------------------------------------|
| **Python** | 3.10+ | Needed for the agents in this guide         |
| **uv** | latest | Python environment & dependency manager ([install guide](https://docs.astral.sh/uv/getting-started/installation/)). **Ensure it's installed globally, not as part of a specific python context.** |
| **Node.js** | 18+ | Required to run Coral Studio (the UI)                       |
| **yarn** | Comes with Node | Used to install and run Studio dependencies                 |
| **Git** | latest | To clone agent and Coral repos                              |
| **OpenAI API Key** | Any | Needed for agents using OpenAI models (GPT)                 |

### Recommended Tools

| Tool | Reason |
|------|--------|
| **Visual Studio Code** | IDE for editing agent code and config |

---

## Note on Docker
All of these components are possible to run in Docker containers, but for this guide we will be running them locally.

See our Docker guide for more details on how to run Coral in Docker: [Coral Docker Guide](./docker-guide.md)

## Note on Windows
If you're on Windows, you may need to use Git Bash or WSL (Windows Subsystem for Linux) to run the commands in this guide. PowerShell may not work correctly with some of the commands.

WSL 2 works, but WSL1 performs better.

Alternatively, you may use Docker, but Windows users may suffer from performance issues with Docker Desktop since windows forces all containers to run in WSL2.

# Getting Started
First clone this repository:
```bash
git clone https://github.com/Coral-Protocol/MultiAgent-Quickstart.git
cd MultiAgent-Quickstart
```

After cloning that, cd in and clone the server:

```bash
git clone -b stabletutorial https://github.com/Coral-Protocol/coral-server
```

Lastly you will need the agents. cd into `agents` and clone the agent repos.

```bash
cd agents
git clone -b stabletutorial https://github.com/Coral-Protocol/Coral-GithubMCP-Agent.git github
git clone -b stabletutorial https://github.com/Coral-Protocol/Coral-FirecrawlMCP-Agent.git firecrawl
git clone -b stable https://github.com/Coral-Protocol/Coral-Interface-Agent.git interface
```

Once you've cloned everything, your file structure should look like this:

```
MultiAgent-Quickstart/
├── coral-server/
├── agents/
│   ├── github/
│   ├── firecrawl/
│   └── interface/
├── ...
```


## Run Coral Server

**Coral Server** is the engine that runs your multi-agent sessions, executes agent logic, and facilitates communication between agents.

In this step, you'll set up and start Coral Server locally, with the config path pointed at the repository root, so it uses our provided `application.yaml` config. Without specifying a path, the internal example application yml would be used, which won't have the agents we want on it.

### 1. Run the Coral Server from source

```bash
cd coral-server
CONFIG_PATH=.. ./gradlew run
```

> **NOTE**: this will appear to get stuck at 86% - this is a gradle quirk, if the logs say the server has started then it has started.

This will launch the server,
which acts as a control plane that manages networks of agents, and facilitates their communication and collaboration.

---

## Start Coral Studio (UI)

**Coral Studio** is a web-based UI for managing sessions, agents, and threads visually.

This step walks you through installing and running Coral Studio locally on your machine.

### 1. Run with yarn

Open your terminal (Git Bash or PowerShell) and run:

First, clone coral studio:
```bash
git clone https://github.com/Coral-Protocol/coral-studio
cd coral-studio
git checkout f438ff26893948b07bdd76a1733749f256b18636
```

Then install the dependencies:
```bash
yarn install
```

Then to run it from source, do this:
```
yarn dev
```

This will start the Studio UI at [`http://127.0.0.1:5173`](http://127.0.0.1:5173)

Open this URL in your web browser to access the Coral Studio interface.

### 2. Confirm it's Working
You should see:
- A dashboard for Coral Studio
- An option to create a session or connect to Coral Server
- A visual interface to observe and interact with threads and agents

---

## Creating a Session
### What is a Session?
In short, sessions are what applications work with to create and manage the lifecycle of a given graph of agents. If you're familiar with Kubernetes, they could be thought of as custom resources that create agents all in a unique shared namespace.

Sessions are created through a REST interface on the coral server, usually on-demand in response to a user's interaction or other event from the core business logic of a software service.

Read more about sessions on Coral's docs [here](https://docs.coralprotocol.org/CoralDoc/CoreConcepts/Sessions).

### Creating a Session via Coral Studio
In production, you would typically just make a POST request using your preferred HTTP client library from your application's backend code.

For development purposes it makes sense to use Curl, Postman, or the Coral Studio UI to create sessions.

Let's use the Coral Studio UI to create a session.

First, we need to connect to our Coral Server:

- Click on the server selector, and press 'Add a server'

![The server selector](images/server-selector.png)

- For the host, enter `localhost:5555`, and press 'Add'.

![Add a server dialog box](images/add-a-server.png)

Now we can create the session:

- Click on 'Select session', and then 'New session'

  ![select session](images/select-session.png)
- Make sure 'Application ID' and 'Privacy Key' match what you have in your `application.yaml`
    - If you're using our provided config, `app` and `priv` work.

    ![app id and priv key inputs](images/app-priv.png)

Now we pick our agents:

- Click the 'New agent' under, and select 'interface'

 ![new agent dropdown](images/new-agent.png)
- Fill in the needed API keys
    - For OpenAI, see [this page](https://platform.openai.com/api-keys)
    - For Firecrawl, see [their website](https://www.firecrawl.dev/)
    - For GitHub, see [this guide](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) (make sure to *only* allow read-only permissions!)
- For the 'interface' agent specifically, we need to add the user input custom tools:
    - Go to the 'Custom Tools' section, and in the dropdown, select both 'request-question' and 'answer-question'
 
    ![custom tools dropdown](images/add-custom-tools.png)
- Repeat for the other 2 agents ('github' & 'firecrawl')

> Feel free to add some extra prompts to fine-tune agent behaviour!

With all our agents ready, we need to make a group - to indicate that all of these agents can interact:

- Go to the 'Groups' section, and click 'New group'

![new group button](images/new-group-button.png)
- Click on 'Empty group', and select all of our agents

![all agents selected in our group](images/agent-groups.png)

> You can also copy the resulting JSON from the 'Export' section - to easily import all of these settings again in future.


Click on 'Create', and your session should spin up!

### Interact with them through Coral Studio

Coral Studio is filling in for the interface a production application would have here. You can see in the `/sessions` POST that gets sent when you create a session (observable via the browser's network tab), that the custom tools are brought in this way:

```json
{
  "agentGraph": {
    "agents": {
      "interface": {
        "options": {
          "MODEL_API_KEY": "..."
        },
        "type": "local",
        "agentType": "interface",
        "tools": [
          "user-input-respond",
          "user-input-request"
        ]
      },
      "github": {
        "options": {
          "MODEL_API_KEY": "...",
          "GITHUB_PERSONAL_ACCESS_TOKEN": "..."
        },
        "type": "local",
        "agentType": "github",
        "tools": []
      },
      "firecrawl": {
        "options": {
          "MODEL_API_KEY": "...",
          "FIRECRAWL_API_KEY": "..."
        },
        "type": "local",
        "agentType": "firecrawl",
        "tools": []
      }
    },
    "links": [
      [
        "firecrawl",
        "github",
        "interface"
      ]
    ],
    "tools": {
      "user-input-respond": {
        "transport": {
          "type": "http",
          "url": "http://localhost:5173/api/mcp-tools/user-input-respond"
        },
        "toolSchema": {
          "name": "answer-question",
          "description": "Answer the last question you requested from the user. You can only respond once, and will have to request more input later.",
          "inputSchema": {
            "type": "object",
            "properties": {
              "response": {
                "type": "string",
                "description": "Answer to show to the user."
              }
            },
            "required": [
              "response"
            ]
          }
        }
      },
      "user-input-request": {
        "transport": {
          "type": "http",
          "url": "http://localhost:5173/api/mcp-tools/user-input-request"
        },
        "toolSchema": {
          "name": "request-question",
          "description": "Request a question from the user. Hangs until input is received.",
          "inputSchema": {
            "type": "object",
            "properties": {
              "message": {
                "type": "string",
                "description": "Message to show to the user."
              }
            }
          }
        }
      }
    }
  }
}
```
Note the `"tools"` object. 

Since the agents are being made with an environment variable pointing to their personal MCP server address on their Coral server, other tools are also possible to be brought in for individual agents.

In this case, this custom input tool which maps to a HTTP request back to coral studio is being provided to the agent.

#### Navigating to Input Tool Queries
Once you created the session, each agent was instantiated and began iterating in their loops, freely communicating and taking actions.

When the interface agent is ready, it'll call our custom 'request-question' tool, and a notification will appear in the 'Tools > User Input' tab in your sidebar.
![user input tool page](images/user-input.png)

After sending your response, under the hood the blocking tool call will finally return.

The interface agent then will continue operating in its loop until it requests input again or shuts down.

#### Observing Agent Collaboration

You can see the agents collaborating to fulfill the user query by selecting a session, expanding the "Threads" collapsible section and clicking into an individual thread.

Since we gave them an 'answer-question' tool, the interface agent will use that once it is satisfied with a response to give the user.

### Next Steps
By now, you should have a working multi-agent system that can interact with users and each other.

Since the agents are running from source, you can modify their code and configuration to change their behavior by directly editing the agent source code in the `agents` directory.

For quickly iterating individual agent changes, check out [Devmode](https://docs.coralprotocol.org/CoralDoc/Introduction/UsingAgents#devmode) in the Coral docs.

To add a new agent:
1. Clone the agent repository into the `agents` directory (Or wherever you want to keep your agent sources).
2. Add the agent to the `application.yaml` config file under the `agents` section.
3. Modify your Session POST request to include the new agent in the `agentGraph` section.

You can ask for help and share what you have made in the [Coral Protocol Discord](https://discord.gg/MqcwYy6gxV)

You may be interested in the [Production Deployments](#production-deployments) section below, which covers how to run Coral in production environments.

---

## Production Deployments

### Runtimes
So far we've been using the Executable runtime, which is great for development and testing to run source code directly.
But as mentioned earlier, in production, you should use another runtime.

Using the Docker runtime is detailed [here](https://docs.coralprotocol.org/CoralDoc/Introduction/UsingAgents#docker).

The only difference is that a docker image tag is used instead of a source code path. Coral Server when then interact with the Docker daemon to run the agent in a container when a session is created.

### Backend
In production, you would typically have a backend service that manages sessions and interacts with a Coral Server via its REST API.
Coral imposes no restrictions on how you implement your backend, so you can use any language or framework you're comfortable with. It just needs to make HTTP requests to a Coral Server's endpoints.

At deployment time, the coral server needs to be deployed alongside your backend service, and the backend service needs to be able to connect to it. With Kubernetes, this means adding a Service to your cluster that points to the Coral Server pod. Coral Server instances for your application should not be exposed to the public internet, as they are not meant to be directly interacted with by users.
The Coral Server will also need a Docker socket to run agents in containers, so you will need to mount a Docker socket into the Coral Server pod. 

A Kubernetes runtime is in development along with other convenient runtime options, so keep an eye on the [Coral Server GitHub](https://github.com/Coral-Protocol/coral-server).

### Frontends
Coral also does not impose any restrictions on how you implement your frontend. You can use any framework or library you're comfortable with.

It is important to note that unlike Coral Studio, a frontend should not directly interact with the Coral Server. Instead, it should interact with your backend service, which in turn interacts with the Coral Server. Coral Studio is a development tool, and does actually have its own backend service that it uses to interact with the Coral Server, though it assumes that the Coral Server is running in the same private network.
