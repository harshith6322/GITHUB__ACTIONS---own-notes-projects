Perfect question, Harshith 🔥 — this is one of the most **important distinctions** in GitHub Actions automation.

Let’s break down **`workflow_run` vs `workflow_call`** clearly 👇

---

## ⚔️ `workflow_run` vs `workflow_call` — The Core Difference

| Feature              | `workflow_run`                                               | `workflow_call`                                                    |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------ |
| **When it triggers** | When another workflow **finishes running** (based on events) | When **explicitly called** from another workflow                   |
| **Connection type**  | **Event-based (indirect)** trigger                           | **Direct call (like a function call)**                             |
| **Use case**         | To chain workflows (e.g. *Build → Test → Deploy*)            | To reuse common workflow logic (like *reusable jobs or templates*) |
| **Trigger type**     | Uses: `on.workflow_run`                                      | Uses: `on.workflow_call`                                           |
| **Data passing**     | Only limited data via the GitHub event context               | You can pass **inputs and secrets** directly                       |
| **Execution**        | Runs **after** another workflow finishes                     | Runs **inside** the caller workflow (synchronously)                |
| **Common analogy**   | Like saying “Run workflow B when workflow A completes.”      | Like saying “Call function B from inside A.”                       |

---

## 🧩 Example 1 — `workflow_run`

Used for **chaining independent workflows**.

### 🔹 `build.yml`

```yaml
name: Build

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building project..."
```

### 🔹 `deploy.yml`

```yaml
name: Deploy

on:
  workflow_run:
    workflows: ["Build"]
    types: [completed]

jobs:
  deploy:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying after Build succeeded!"
```

✅ Here:

* `deploy.yml` runs only **after** `build.yml` completes successfully.

---

## 🧩 Example 2 — `workflow_call`

Used for **reusable workflows** — like a shared template.

### 🔹 `reusable.yml`

```yaml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      MY_SECRET:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building for ${{ inputs.environment }}"
      - run: echo "Using secret: ${{ secrets.MY_SECRET }}"
```

### 🔹 `main.yml`

```yaml
name: Main

on:
  push:
    branches: [main]

jobs:
  call-reusable:
    uses: ./.github/workflows/reusable.yml
    with:
      environment: production
    secrets:
      MY_SECRET: ${{ secrets.MY_SECRET }}
```

✅ Here:

* `main.yml` **calls** `reusable.yml` directly like a **function**.
* `reusable.yml` can take **inputs** and **secrets**, making it modular.

---

## 🚀 When to Use What

| Scenario                                                                          | Use               |
| --------------------------------------------------------------------------------- | ----------------- |
| You want Workflow B to run **after Workflow A finishes**                          | ✅ `workflow_run`  |
| You want to **reuse logic** (build, deploy, test steps) across multiple workflows | ✅ `workflow_call` |
| You want to trigger a different workflow **conditionally after success/failure**  | ✅ `workflow_run`  |
| You want **reusable “template” workflows** with parameters                        | ✅ `workflow_call` |

---

Would you like me to show you a **real example combining both** —
like using `workflow_call` inside a `workflow_run` setup (so “Build” triggers “Reusable Deploy”)?
It’s a common advanced pattern in CI/CD pipelines.
