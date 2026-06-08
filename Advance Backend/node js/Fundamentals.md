# Node.js — Fundamentals

> Notes from AlgoCamp · Topics: Runtime, Package Management, Versioning, Environment Variables

---

## 1. What is Node.js?

JavaScript originally only ran inside browsers. Node.js is a **runtime environment** that takes JS outside the browser and runs it directly on a machine.

> Created in **2009 by Ryan Dahl** as an open source project.

### Browser JS vs Node.js

```mermaid
graph LR
    A[JavaScript] -->|runs inside| B[Browser]
    A -->|runs inside| C[Node.js Runtime]

    B --> B1[DOM / HTML / CSS]
    B --> B2[Timers]
    B --> B3[Network - client only]

    C --> C1[Filesystem access]
    C --> C2[Process control]
    C --> C3[Timers]
    C --> C4[Network - client + server]

    style B fill:#FAECE7,stroke:#993C1D,color:#712B13
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

### Node.js Internal Architecture

```mermaid
graph TB
    subgraph Node.js Runtime
        V8[V8 Engine\nExecutes JS]
        libuv[libuv\nAsync I/O, Timers, Networking]
        JS[Your JavaScript Code]
    end

    JS --> V8
    V8 --> libuv
    libuv --> OS[Operating System]

    style V8 fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style libuv fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style JS fill:#EEEDFE,stroke:#534AB7,color:#3C3489
```

---

## 2. REPL

Node.js ships with a **REPL** console — an interactive shell to run JS code directly.

```mermaid
graph LR
    R[Read] --> E[Evaluate] --> P[Print] --> L[Loop] --> R

    style R fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style P fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style L fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

Run it by typing `node` in your terminal.

---

## 3. Package Management

### Key Terms

|Term|Meaning|
|---|---|
|**Package**|A pre-written piece of code intended to do a specific task|
|**Library**|A package that does one focused thing|
|**Framework**|A collection of libraries that form a complete ecosystem|

```mermaid
graph TD
    Framework[Framework\ne.g. Express]
    Framework --> LibA[Library A]
    Framework --> LibB[Library B]
    Framework --> LibC[Library C]
    LibA --> PkgA[Package]
    LibB --> PkgB[Package]
    LibC --> PkgC[Package]

    style Framework fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style LibA fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style LibB fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style LibC fill:#E6F1FB,stroke:#185FA5,color:#0C447C
```

### Package Managers

- **NPM** — Node Package Manager (default, comes with Node.js)
- **PNPM** — saves disk space by using symlinks instead of copying packages per project
- **Yarn** — another alternative

### Setting Up a Project

```bash
npm init     # creates package.json
pnpm init    # same, using pnpm
```

---

## 4. `package.json`

The metadata file for your project.

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "dev": "node --watch index.js"
  },
  "dependencies": {
    "axios": "^1.16.0",
    "dotenv": "~17.4.2"
  }
}
```

**Key fields:**

- `scripts` — alias long commands, run with `npm run <name>`
- `dependencies` — packages your project needs
- `main` — entry point file

### Hot Reloading

The `--watch` flag (built into modern Node.js) auto-restarts when a file changes. Previously needed the external package `nodemon`.

---

## 5. Dependency Graph

When you install a package, it may depend on other packages — which depend on more. NPM/PNPM resolves and downloads the **entire graph** automatically.

```mermaid
graph TD
    A[Your App] --> B[axios]
    A --> C[dotenv]
    B --> D[follow-redirects]
    B --> E[form-data]
    B --> F[proxy-from-env]
    D --> G[debug]
    E --> H[mime-types]

    style A fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style B fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style C fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style D fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style E fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style F fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style G fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style H fill:#F1EFE8,stroke:#5F5E5A,color:#444441
```

---

## 6. Lock Files

|File|Created by|
|---|---|
|`package-lock.json`|npm|
|`pnpm-lock.yaml`|pnpm|

```mermaid
graph LR
    PJ[package.json\nversion ranges] -->|npm install| LF[lock file\nexact versions]
    LF -->|shared with team| D1[Dev 1\nsame versions]
    LF --> D2[Dev 2\nsame versions]
    LF --> CI[CI/CD\nsame versions]

    style PJ fill:#FAEEDA,stroke:#854F0B,color:#633806
    style LF fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style D1 fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style D2 fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style CI fill:#F1EFE8,stroke:#5F5E5A,color:#444441
```

**Why important:** Ensures every team member and deployment gets the exact same versions. Prevents "works on my machine" bugs.

---

## 7. Version Numbers — `major.minor.patch`

```
1  .  16  .  0
↑      ↑     ↑
major minor patch
```

```mermaid
graph TD
    V[axios version] --> E[1.16.0\nExact — install only this]
    V --> T[~1.16.0\nTilde — patch updates only\n≥ 1.16.0 and < 1.17.0]
    V --> C[^1.16.0\nCaret — minor updates allowed\n≥ 1.16.0 and < 2.0.0]

    style V fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style E fill:#FAECE7,stroke:#993C1D,color:#712B13
    style T fill:#FAEEDA,stroke:#854F0B,color:#633806
    style C fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

> **Rule of thumb:** Use `^` for most packages. Use exact versions when stability is critical.

---

## 8. Data Formats — XML vs JSON vs YAML

```mermaid
graph LR
    Data[Structured Data] --> XML
    Data --> JSON
    Data --> YAML

    XML --> X1[Verbose\nTag-based\nLegacy systems]
    JSON --> J1[Clean syntax\nAPIs and config\nMost common]
    YAML --> Y1[Human-readable\nConfig files\nDocker, CI/CD]

    style XML fill:#FAECE7,stroke:#993C1D,color:#712B13
    style JSON fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style YAML fill:#E6F1FB,stroke:#185FA5,color:#0C447C
```

**Same data, three formats:**

```xml
<!-- XML -->
<Product>
  <name>Iphone</name>
  <price>50000</price>
</Product>
```

```json
// JSON
{ "product": { "name": "Iphone", "price": 50000 } }
```

```yaml
# YAML
- name: "Iphone"
  price: 50000
  currency: [USD, INR]
```

---

## 9. Node.js Globals

Node provides global variables automatically — just like how browsers provide `document` or `window`.

```mermaid
graph LR
    Browser[Browser globals] --> BD[document]
    Browser --> BW[window]
    Browser --> BN[navigator]

    Node[Node.js globals] --> NP[process]
    Node --> NB[__dirname]
    Node --> NF[__filename]

    NP --> PE[process.env\nenv variables]
    NP --> PA[process.argv\nCLI arguments]
    NP --> PX[process.exit\nexit the process]

    style Browser fill:#FAECE7,stroke:#993C1D,color:#712B13
    style Node fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style NP fill:#EEEDFE,stroke:#534AB7,color:#3C3489
```

---

## 10. Environment Variables

**What:** Key-value pairs stored at the OS level. Any process on the machine can read them.

```mermaid
graph TD
    OS[OS-level env store\nPORT=3000\nDB_PASS=secret] --> P1[Process 1\nNode app]
    OS --> P2[Process 2\nAnother app]
    OS --> P3[Process 3\nAny process]

    style OS fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style P1 fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style P2 fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style P3 fill:#EEEDFE,stroke:#534AB7,color:#3C3489
```

**Why use them:**

- Store sensitive info (API keys, DB passwords) outside your code
- Share common config across multiple processes

### How to Set Them

```mermaid
graph LR
    A[How to set env vars] --> B[Persistent\nAdd to ~/.bashrc or ~/.zshrc\nexport KEY=Value\nSurvives terminal restarts]
    A --> C[Temporary\nRun in terminal\nexport KEY=Value\nGone when terminal closes]

    style B fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

### Access in Node.js

```js
process.env.PORT        // reads the PORT env variable
process.env.DB_PASSWORD // reads a DB password
```

---

## Quick Reference

```bash
node                    # open REPL
node index.js           # run a file
node --watch index.js   # run with hot reload

npm init                # create package.json
npm install             # install all dependencies
npm run dev             # run the "dev" script
```