layout: true

---

class: center, middle, title

# 12-Factor Applications

Benjamin R. Haskell

benizi -
[.fa.fa-github[]][github]
[.fa.fa-twitter[]][twitter]
[.fa.fa-facebook[]][facebook]
[.fa.fa-globe[]](http://benizi.com)

[twitter]: https://twitter.com/benizi
[github]: https://github.com/benizi
[facebook]: https://facebook.com/benizi

---

# What is 12-Factor?

12-Factor is a set of guiding principles for developing horizontally scalable apps.

- The Twelve-Factor App: [12factor.net][12factor]
    - Created by Adam Wiggins, cofounder of Heroku

Type of scaling gives answer to "We have more traffic. What do we do?"
- Vertically scalable => use a bigger server
- Horizontally scalable => add more servers

Heroku is a Platform-as-a-Service (PaaS):

- Containerized Linux
- Easy to Deploy
- Easy to Scale
- Very nice integrations
- Tradeoff: $$$, SPOF (but so is AWS)

[12factor]: http://12factor.net

---

class: roman

# The 12 Factors

1. [Codebase][factor1] - Use revision control, single app is a single repo.
2. [Dependencies][factor2] - Keep dependencies explicit and isolated.
3. [Config][factor3] - Use environment variables for configuration.
4. [Backing services][factor4] - Treat backing services as resources.
5. [Build, release, run][factor5] - Keep build and run stages separate.
6. [Processes][factor6] - App is one or more stateless processes.
7. [Port binding][factor7] - Export services by binding ports.
8. [Concurrency][factor8] - Scale out by creating more processes.
9. [Disposability][factor9] - Processes should start fast and fail gracefully.
10. [Dev/prod parity][factor10] - Keep environments as similar as possible.
11. [Logs][factor11] - Logs are streams of events.
12. [Admin processes][factor12] - These are one-off processes.

[factor1]: http://12factor.net/codebase
[factor2]: http://12factor.net/dependencies
[factor3]: http://12factor.net/config
[factor4]: http://12factor.net/backing-services
[factor5]: http://12factor.net/build-release-run
[factor6]: http://12factor.net/processes
[factor7]: http://12factor.net/port-binding
[factor8]: http://12factor.net/concurrency
[factor9]: http://12factor.net/disposability
[factor10]: http://12factor.net/dev-prod-parity
[factor11]: http://12factor.net/logs
[factor12]: http://12factor.net/admin-processes

---

# [I. Codebase][factor1]

- Single codebase in a revision control system.
- Multiple codebases?
    - Not an app
    - A distributed system.
- Distributed system can be composed of multiple 12-factor apps.
- Many "deploys" of the same app.
    - At different versions.
    - With different backing services.
- Tools used to achieve this:
    - Git, Mercurial, Subversion, GitHub

???

- One codebase tracked in revision control, many deploys

- Use Git.

---

# [II. Dependencies][factor2]

- All dependencies should be explicitly declared and isolated.
    - Allows knowing the exact set of deps that will be used.
    - Prevents unknown implicit deps from "leaking" in.
- Never rely on system-wide dependencies.
    - All dependencies should be explicitly declared.
- No package manager gets you 100% of the way there.
    - Many rely on system-level dependencies being preinstalled.
- Tools used to achieve this:
    - Rubygems, npm, Leiningen, vendoring
- Interestingly, Python separates the two goals:
    - `pip` provides dependency declaration
    - `virtualenv` (now `venv`?) provides isolation

???

- Explicitly declare and isolate dependencies

- "Even C has Autoconf for dependency declaration."
- e.g. no implicit dependency on ImageMagick being installed.

---

# [III. Config][factor3]

- All configuration is done through environment variables.
    - Only applies to "external" configuration.
- Examples of internal configuration:
    - Routing configuration in Rails or Express.
    - Dependency injection configuration in Spring.
- Avoid brittle, language-specific configuration files.
    - E.g., Rails: `config/database.yml`, `config/secrets.yml`
- "Environments" of config vars are brittle and unscalable.
- Tools used to achieve this:
    - `dotenv`: useful for collecting vars for local use
    - `figaro`: useful for deploying vars to Heroku
    - [Figaro vs Dotenv][figaro-vs-dotenv]

[figaro-vs-dotenv]: https://github.com/laserlemon/figaro#is-figaro-like-dotenv

???

- Store config in the environment

- slight disagreement here, but only on the utility of "grouped" configs.

---

# [IV. Backing services][factor4]

- Anything the app consumes over the network as part of normal operation.
- 12-factor treats "local" and "3rd-party" services identically.
- Example backing services:
    - Databases: PostgreSQL, MySQL, Oracle
    - Error reporting: New Relic, AirBrake
    - Caching: Redis, Memcached
    - Queueing: RabbitMQ
    - Email management: SendGrid
    - Storage: Amazon S3
- Swapping out for a replacement should require no code changes.
- One of Heroku's biggest selling points:
    - Many "add-ons" available.
    - Once added, configuration is injected as environment variables.
- Tools used to achieve this:
    - Language support: "configuration" = "URI"

???

- Treat backing services as attached resources

---

# [V. Build, release, run][factor5]

- Strict separation between `build`, `release`, and `run` stages.
- Build: convert code into executable.
    - Different levels of complexity depending on language.
    - "High touch" (developer on hand, usually)
- Release: combine the build with current config.
    - Usually language-independent, but very platform-specific.
    - Range from "Git checkout" to "Build an Amazon AMI"
    - Always has a unique ID (useful for correlating issues).
- Run: start a set of processes from a release.
    - Depends entirely on platform.
- Biggest benefit of separate stages: easy rollbacks.
    - Deploy finishes and there's a bug? Restart against prior release.
- Tools used to achieve this:
    - Heroku itself: e.g., this is how buildpacks work
    - [`Terraform`][terraform] (HashiCorp): "Build, Combine, and Launch Infrastructure"

[terraform]: https://www.terraform.io/

???

- Strictly separate build and run stages

---

# [VI. Processes][factor6]

- 12-Factor processes are stateless and share-nothing.
- The app will usually have a specific desired configuration.
    - Expressed as a list of types with number of instances.
    - `Procfile` is the (YAML) format Heroku uses:
        ```yaml
        web: lein run -m com.benizi.ecommerce $PORT
        worker: /app/goworker -queues=process
        console: lein repl
        ```
- Anything that needs to be persisted needs a stateful backing service.
- Only a small amount of cache available for local operations.
    - Never assume the next request will be served from the same machine.
- 12-Factor apps prefer asset packaging as part of `build` step.
- Sticky sessions are the devil.

???

- Execute the app as one or more stateless processes

---

# [VII. Port binding][factor7]

- 12-Factor apps export services by binding ports.
- One app can be the backing service for another.
- Usually HTTP (web apps, after all), but certainly not exclusively:
    - E.g., export Memcached-compatible interface.
- Mostly seems to be an argument against monolithic servers
    - E.g., servlet-container architectures popular in Java-land.
- Heroku-specific: expects the `web` process to bind `$PORT`.
- Tools used to achieve this:
    - Programming language support (Golang `dial`).
    - `Docker`: definitely inspired by 12-Factor.
    - `socat`: like `nc`, but way more options.

???

- Export services via port binding

---

# [VIII. Concurrency][factor8]

- Processes are first-class citizens in 12-factor apps.
- Never daemonize.
    - Process manager should handle that.
- Don't handle log files.
    - Process manager should handle that.
- Don't handle crashing processes.
    - Process manager should handle that.
- Don't handle user-initiated restarts/shutdowns.
    - Process manager should handle that.  (Get the message?)
- Tools used to achieve this:
    - [Foreman][foreman] in dev
    - Something platform-specific (e.g. Upstart, Heroku)

[foreman]: https://ddollar.github.io/foreman/

???

- Scale out via the process model

---

# [IX. Disposability][factor9]

- 12-Factor app processes are disposable
- Minimize startup time
- Shutdown gracefully for SIGTERM
    - Web: stop listening for new requests, finish current ones.
    - Worker: return jobs to work queue.
- Everything should be transactional and/or idempotent.
    - Can start process again ...
        - Transactional: ... because nothing changed on failure.
        - Idempotent: ... because outcome will be identical.
- Logical conclusion: Crash-only design.
    - Don't assume your program will cleanly shutdown.
    - Not the same as lazily ignoring errors.
- Tools used to achieve this:
    - Robust queueing solutions: RabbitMQ.
    - Language/database support: CRDTs, distributed txns.

???

- Maximize robustness with fast startup and graceful shutdown

---

# [X. Dev/prod parity][factor10]

- Keeping environments identical leads to better continuous deployment.
- Three "gaps": time, personnel, and tools.
    - Time gap: between development and deployment.
    - Personnel: devs and ops vs DevOps.
    - Tools: Local vs "production" services.
- Adapters are the devil.
    - Resist their temptations.
    - But, nice when a backing service needs to be swapped out.
- "Lightweight" services for local dev are no longer compelling.
    - Open source is usually installable locally.
    - Proprietary servers may be exceptions.
- Tools used to achieve this:
    - Docker: great way to get identical service running in dev.

???

- Keep development, staging, and production as similar as possible

---

# [XI. Logs][factor11]

- Logs are streams of events.
- 12-Factor apps don't care:
    - Where their logs end up
    - How their logs are stored
- Write logs unbuffered to STDOUT.
- Tools used to achieve this:
    - Log routers: `Logplex`, `Fluentd`, `Logstash`
    - Analysis: `Splunk`

???

- Treat logs as event streams

---

# [XII. Admin processes][factor12]

- One-off tasks should be run in an identical environment.
    - Minimizes errors the same way as dev/prod parity.
- Example tasks:
    - Database migrations
    - Console to inspect live application's state
    - One-time cleanup scripts (committed to codebase).
- 12-Factor loves REPLs.
    - Read-Eval-Print-Loop.
- Tools used to achieve this:
    - REPL

???

- Run admin/management tasks as one-off processes

---

# Disclaimers

- 12-Factor is (Internet-)old:
    - Mostly written in June, 2011 (*last* updated in Jan, 2012)
- Predates Docker and Ansible, two tools I like that both:
    - Had 12-Factor as a widespread part of the ecosystem they inhabit.
    - Can/do help with many of the twelve factors.
- Not everything is a 12-Factor app:
    - Hadoop/Hive, Spark, and other "big data" applications
- Not all twelve factors are equally important:
    - E.g. [12-Factor Apps in Plain English][blogpost] chooses "Importance" of each
- Some are contentious:
    - \#11 (Logs): Problem is complex regardless of using STDOUT
- Complaints about Unix-centric nature of 12-Factor:
    - Especially `SIGTERM` in #9 (Disposability)
    - Also process model of #6 (Processes)

[blogpost]: http://www.clearlytech.com/2014/01/04/12-factor-apps-plain-english/
