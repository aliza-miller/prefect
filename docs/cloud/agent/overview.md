# Overview

The Prefect Agent is a small process spun up to orchestrate Flow runs. The agent queries Prefect Cloud for new or incomplete Flow runs and allocates resources for them on the deployment's platform of choice.

Prefect Cloud follows a hybrid approach to workflow execution. This means that Prefect processes run inside the tenant's infrastructure and only send requests _out_ to Prefect Cloud. Both the Prefect Agent and all Prefect Flows which run using Cloud follow this communication pattern.

[[toc]]

### Agent Process

Agents start by first querying Prefect Cloud for their respective tenant ID (inferred from the API token that the agent is given). The agent then continually queries Prefect Cloud for Flow runs to be started on that agent's platform.

Flow runs can be created through the [GraphQL API](../concepts/graphql.html), [CLI](../concepts/cli.html), [programatically](../concepts/flow_runs.html#creating-a-flow-run), or [UI](../concepts/ui.html). Once the Flow run is created, the agent detects it, retrieves the Flow run's metadata, and creates a unit of execution on the agent's platform. Examples of this include a Docker container for a Local Agent or a job for a Kubernetes Agent. Once the Agent submits the Flow run for execution, that Flow run is set to a `Submitted` state.

### Installation

If Prefect is already [installed](../../core/getting_started/installation.html) no additional work is required to begin using Prefect Agents!

### Usage

Prefect Agents can easily be configured through the CLI.

```
$ prefect agent
Usage: prefect agent [OPTIONS] COMMAND [ARGS]...

  Manage Prefect agents.

  Usage:
      $ prefect agent [COMMAND]

  Arguments:
      start       Start a Prefect agent
      install     Output platform-specific agent installation configs

  Examples:
      $ prefect agent start
      ...agent begins running in process...

      $ prefect agent start kubernetes --token MY_TOKEN
      ...agent begins running in process...

      $ prefect agent install --token MY_TOKEN --namespace metrics
      ...k8s yaml output...

Options:
  -h, --help  Show this message and exit.
```

All Prefect Agents are also extendable as Python objects and can be used programatically!

```python
from prefect.agent import LocalAgent

LocalAgent().start()
```

### Tokens

Prefect Agents rely on the use of a `RUNNER` token from Prefect Cloud. For information on tokens and how they are used visit the [Tokens](../concepts/tokens.html) page.

### Flow Affinity: Labels

Agents have an optional `labels` argument which allows for separation of execution when using multiple Agents. This is especially useful for teams wanting to run specific Flows on different clusters. For more information on labels and how to use them visit [Environments](../execution/overview.html#labels).

By default, Agents have no set labels and will only pick up runs from Flows with no specified Environment Labels. Labels can be provided to an agent through a few methods:

- Initialization of the Agent class:

```python
from prefect.agent import LocalAgent

LocalAgent(labels=["dev", "staging"]).start()
```

- Arguments to the CLI:

```
$ prefect agent start --label dev --label staging
```

- As an environment variable:

```
$ export PREFECT__CLOUD__AGENT__LABELS='["dev", "staging"]'
```

:::tip Environment Variable
Setting labels through the `PREFECT__CLOUD__AGENT__LABELS` environment variable will make those labels the default unless overridden through initialization of an Agent class or through the CLI's `agent start` command.
:::
