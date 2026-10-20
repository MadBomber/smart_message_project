# SmartMessage Project

A workspace for SmartMessage, a Ruby messaging framework, together with the transport gems and demo applications that grow up around it.

SmartMessage is in alpha. Its APIs are still changing, and the transport gems here are earlier still.

## The Idea

Most messaging code ties the meaning of a message to the system that carries it. The publisher has to know about Redis channels or RabbitMQ exchanges. The subscriber has to know how the bytes were encoded. Switching brokers means rewriting the application.

SmartMessage separates the two. A message is a plain business object. It describes its own contents, validates itself, and records who sent it, who it is for, and where replies should go. How the message travels is a pluggable transport. How it is encoded is a pluggable serializer. The goal is to do for messaging what an object-relational mapper does for databases: write the application once and choose the delivery mechanism by configuration.

## Goals

- **Keep messages independent of transport.** Delivering a message by Redis, a broker, a local network, a pipe on the same machine or plain memory should not change the message or the code that handles it.
- **Make messages self-describing.** A message carries its own schema, validation rules, version and documentation, so publishers and subscribers agree on what it means.
- **Support real conversations between entities.** Built-in sender, recipient and reply-to addressing covers broadcast, point-to-point and request/reply without extra plumbing.
- **Be dependable in production.** Deduplication, a dead letter queue for failed messages, circuit breakers, structured logging and processing statistics are part of the framework, not add-ons.
- **Let the transport list grow.** A small, stable contract between the core and its transports means a new delivery mechanism can ship as its own gem, without changing the core.
- **Show it working.** Demo applications exercise the framework the way a real system would, not just in unit tests.

## Components

### Core

| Repository | Role |
|------------|------|
| [smart_message](https://github.com/MadBomber/smart_message) | The framework: message definition, validation, addressing, versioning, dispatching, deduplication, logging, and the built-in transports for development, testing and Redis |

### Transport Extensions

Each extension is a separate gem that adds one delivery mechanism, so an application only takes on the dependencies it needs.

| Repository | Delivery mechanism |
|------------|--------------------|
| [smart_message-transport-rabbitmq](https://github.com/MadBomber/smart_message-transport-rabbitmq) | RabbitMQ, using topic routing, durable queues and broker-level reliability |
| [smart_message-transport-nats](https://github.com/MadBomber/smart_message-transport-nats) | NATS, a lightweight, high-throughput messaging system |
| [smart_message-transport-lanet](https://github.com/MadBomber/smart_message-transport-lanet) | Peer-to-peer messaging on a local network, with automatic node discovery and no central broker |
| [smart_message-transport-named_pipes](https://github.com/MadBomber/smart_message-transport-named_pipes) | Unix named pipes, for fast communication between processes on the same machine |

### Demonstrations

| Repository | Role |
|------------|------|
| [smart_message_demos](https://github.com/MadBomber/smart_message_demos) | Example applications, the largest of which simulates a city's emergency services as independent departments that coordinate through messages, organized using the Viable Systems Model |

## How This Workspace Is Organized

This repository does not hold the gems themselves. Each component above is an independent repository with its own history, tests and release cycle. This repository is the workspace that ties them together. It holds the shared environment settings and the project-level task definitions that let one command act on every component at once: checking status, pulling changes, running test suites and quality checks, keeping versions in step, and building and installing the gems in dependency order.

Those tasks are run by Asgard, a task runner whose tasks are defined in `.loki` files. The environment is managed with direnv. Both are shared with the other project workspaces in this family, so the same habits work across all of them.

## Where Things Stand

The core gem is preparing for a 0.1.0 release, which is allowed to break the earlier 0.0.x interface in favor of a cleaner design. The transport extensions are early implementations. Wiring them to the current core, registering them with it, and bringing their versions into step with the core are the next stages of work.

## License

Each component repository carries its own license.
