# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workspace Overview

This directory is the **SmartMessage project workspace**: the `smart_message` core gem, four transport extension gems, and a demos repo. It is not itself a repo. It sits inside `madbomber/projects/` (the workspace-level Asgard toolkit, see `../README.md`), which gitignores every `*_project/` directory. Each subdirectory is its own git repo with its own Gemfile and tests. When working inside one, that repo's `CLAUDE.md` takes precedence.

| Repo | Purpose | Version |
|------|---------|---------|
| `smart_message` | Core: message classes, dispatcher, transports (stdout, memory, redis, file), serializers, DDQ, DLQ, circuit breakers | 0.0.18 |
| `smart_message-transport-lanet` | Transport over the `lanet` LAN messaging gem | 0.0.1 |
| `smart_message-transport-named_pipes` | Transport over OS named pipes (FIFOs) | 0.0.1 |
| `smart_message-transport-nats` | Transport over NATS (`nats-pure ~> 2.0`) | 0.0.1 |
| `smart_message-transport-rabbitmq` | Transport over RabbitMQ (`bunny ~> 2.20`), with its own circuit breaker, DLQ, metrics, and health modules | 0.0.1 |
| `smart_message_demos` | Demo apps (e.g. `city_services_with_vsm`) that use the released `smart_message` gem and `vsm` (Viable Systems Model) | n/a |

`smart_message` is on the `develop` branch (`main` is the release branch). It has in-progress planning docs dated 2026-10-19 (`review_*.md`, `analysis_*.md`, `fix_plan_*.md`) that target a 0.1.0 release allowed to break the 0.0.x API. Read `fix_plan_2026-10-19.md` before changing core behavior.

## Commands (run inside a sub-repo)

```bash
bundle exec rake test                # all tests (default task in the transport gems also runs rubocop)
ruby -Ilib:test test/some_test.rb    # single test file (Minitest)
bundle exec rubocop                  # lint (transport gems carry .rubocop.yml)
bin/console                          # IRB with the gem loaded
```

- Core `smart_message` `rake test` sets `SM_LOGGER_TEST=true` and runs `test/**/*_test.rb`. Redis-dependent tests skip themselves when no Redis is on `localhost:6379`.
- `smart_message-transport-rabbitmq` integration tests need a broker: `docker-compose.test.yml`, plus `TESTING_WITH_RABBITMQ.md` / `TESTING.md` in that repo.
- `smart_message-transport-nats` integration tests need a NATS server.
- `smart_message` uses Ruby `4.1.0-preview1` (`.ruby-version`). The transport gems require Ruby >= 3.2.

## How the Gems Fit Together

- A transport is a subclass of `SmartMessage::Transport::Base` (`smart_message/lib/smart_message/transport/base.rb`). It implements `do_publish(message_class, serialized_message)` plus `subscribe`/`unsubscribe`/`connect`/`disconnect`, and hands inbound data to `receive(message_class, serialized_message)`. That routes it through the core dispatcher.
- Transports are looked up by name through `SmartMessage::Transport::Registry` (`registry.register(name, klass)`). Only `:stdout`, `:memory`, and `:redis` are registered as built-ins. **None of the extension gems registers itself yet**, so callers have to instantiate the class directly (`SmartMessage::Transport::NATS`, `::RabbitMQ`, `::RabbitMQEnhanced`, `::Lanet`, `::NamedPipes`) or register it by hand.
- **Stub base classes shadow the core.** `smart_message-transport-named_pipes` and `smart_message-transport-rabbitmq` each ship their own `lib/smart_message/transport/base.rb` stub. Their own `lib/` comes first on the load path, so `require 'smart_message/transport/base'` loads the stub, not the core class. Their tests therefore do not test against the real core. `lanet` only falls back when `SmartMessage::Transport::Base` is undefined.
- Dependency wiring is inconsistent. `rabbitmq` has the `smart_message` dependency commented out of its gemspec. `named_pipes` requires `smart_message >= 0.1.0`, which the current core (0.0.18) does not satisfy. No transport Gemfile points at a local `path: "../smart_message"` yet.

## Workspace Tooling (Asgard)

The shared tasks live in `../workspace/*.loki`: `check`, `status`, `test_all`, `quality_all`, `wire_local`, `bump_versions`, and others. `../README.md` documents them. `ws_repos.loki` auto-detects `smart_message` as the core and `smart_message-transport-*` as extensions. Only gems whose gemspec declares a `smart_message` dependency count as extensions, so `rabbitmq` is currently excluded.

The project root has `.loki` (entry point, `default_task :check`), `.envrc` (exports `PROJECT`, `RR`, `WORKING`, and `BUNDLE_GEMFILE` from `.bundle_mode`), and `repo_dev.loki` (imports the `dev/*.loki` per-component tasks). `.bundle_mode` is `prod` because no repo here has a `Gemfile.local` yet. Don't run `asgard dev` until those files exist, or bundler will look for a Gemfile that isn't there.

The sub-repos have no `.loki` files of their own yet, so tasks that delegate into each repo (`fetch`, `pull`, `push`, `test_all`, `quality_all`) report them as skipped. Their `.envrc` files also lack `source_up`, so inside a component the project-level environment (`BUNDLE_GEMFILE`, `PROJECT`, `DEV`) is not inherited.

The workspace versioning convention (extensions lockstep with core on the first three segments) is not followed yet: the core is 0.0.18 and the transports are 0.0.1.

## Conventions

- Application code that defines `SmartMessage::Base` subclasses keeps them in a `messages/` directory.
- Most repos have a `COMMITS.md` for commit message style.
