# Saivage Container Setup For getrich

This deployment wires the `saivage` container to the sibling `getrich` repository in this workspace:

- host source repo: `/home/salva/g/ml/saivage`
- host target repo: `/home/salva/g/ml/getrich`
- container source mount: `/opt/saivage`
- container target mount: `/work/getrich`

The target repo already contains `.saivage/config.json`, so the container only needs to mount the repo and start the server against it.

Data acquisition support is enabled for this workspace: Saivage includes a Data Agent that the Manager can invoke with `run_data_agent()`. The Data Agent has built-in MCP tools for web search, URL fetches, metadata checks, bounded downloads, and fallback downloads with attempt logs. Downloads are not tied to one fixed project directory; the agent should choose the project-relative output path that fits the task, record provenance, and account for unreliable sources by trying alternate URLs or methods before failing. `getrich/.saivage/saivage.json` configures the Playwright MCP server in headless mode for JavaScript-heavy data-source pages. Headless mode is preferred because the service runs inside LXC without a normal desktop display; the provisioner also installs `xvfb` and Chromium browser dependencies as a fallback base.

Review support is also enabled: the Manager can invoke a Reviewer with `run_reviewer()` after the main stage work completes. The Reviewer validates the stage objective, acceptance criteria, task reports, changed artifacts, data provenance, leakage controls, benchmark comparisons, and statistical evidence. The Manager is expected to loop through review, targeted fixing tasks, and re-review until blocking issues are resolved, warnings are accepted as explicit residual risk, or escalation is justified.

## One-time setup

Run these commands from `/home/salva/g/ml/saivage`:

```bash
cd /home/salva/g/ml/saivage
make -C deploy create
make -C deploy provision
make -C deploy start
```

What this does:

1. Creates the LXC container.
2. Bind-mounts `/home/salva/g/ml/saivage` into the container as `/opt/saivage`.
3. Bind-mounts `/home/salva/g/ml/getrich` into the container as `/work/getrich`.
4. Installs Node.js and builds Saivage inside the container.
5. Installs a systemd unit that runs `node dist/cli.js serve /work/getrich`.

## Verify

```bash
cd /home/salva/g/ml/saivage
make -C deploy status
make -C deploy logs
make -C deploy ip
```

Inside the container, the service should report the project as `/work/getrich`.

## Updating after code changes

The Saivage source tree is bind-mounted, so rebuild in place:

```bash
cd /home/salva/g/ml/saivage
make -C deploy deploy
```

## Custom target project

If you want the same container flow to target a different repo, override the deploy variables:

```bash
cd /home/salva/g/ml/saivage
TARGET_PROJECT_ROOT=/absolute/path/to/other-project \
TARGET_PROJECT_MOUNT=/work/other-project \
make -C deploy create provision start
```

If the container already exists and you change `TARGET_PROJECT_ROOT` or `TARGET_PROJECT_MOUNT`, rerun `make -C deploy create` so the LXC bind mount is updated.