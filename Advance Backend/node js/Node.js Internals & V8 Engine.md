# Node.js Internals & V8 Engine

> Handwritten notes — cleaned and structured with diagrams

---

## 1. What is Node.js?

**Node** is an **open source**, **cross-platform** runtime environment.

> 📌 Cross-platform = runs on Windows, Mac, Linux

```mermaid
graph LR
    JS[JavaScript\n🔤 Programming Language]
    RE[Runtime Environment\n⚙️ Software layer]
    CAP[Extra Capabilities\ngiven to JS]

    JS -->|needs| RE
    RE -->|provides| CAP

    CAP --> F[📁 Read / Write Files]
    CAP --> P[⚙️ Access Processes & RAM]
    CAP --> T[⏱️ Timers]
    CAP --> N[🌐 Timers & Network]

    style JS fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style RE fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style CAP fill:#E6F1FB,stroke:#185FA5,color:#0C447C
```

---

## 2. What is a Runtime Environment?

> **Runtime Env** = a **software** that provides **extra capabilities** to JS

- JS is just a programming language — it has no power on its own
- A runtime environment wraps JS and gives it abilities

### Two Runtime Environments for JS

```mermaid
graph TD
    JS[JavaScript Code]

    JS --> B[🌐 Browser\nRuntime Env]
    JS --> N[🟢 Node.js\nRuntime Env]

    B --> B1[Timers]
    B --> B2[Read & modify HTML DOM]
    B --> B3[Networking - client only]

    N --> N1[Read / Write Files]
    N --> N2[Access Processes & RAM]
    N --> N3[Timers & Network\nclient + server]

    style B fill:#FAECE7,stroke:#993C1D,color:#712B13
    style N fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

> 💡 Browser is **also** a runtime env — it provides timers, reading and modifying HTML to JS.

---

## 3. History — Why Node.js was Created?

```mermaid
timeline
    title Node.js Origin
    2009 : Ryan Dahl created Node.js
         : Goal — give JS OS-related capabilities
         : (like reading files, accessing processes, running on RAM)
         : Instead of being limited to browser-only abilities
```

**Ryan Dahl's idea:**

- JS was stuck inside the browser
- He wanted JS to also work **outside the browser** — on servers and machines
- So he built Node.js, giving JS: read/write files, access processes, timers, networking

---

## 4. Internals of Node.js

Node.js is not magic — it is built from layers:

```mermaid
graph TB
    subgraph Node.js Runtime
        JS[🟨 JS Layer\nYour JavaScript code]
        CPP[⚙️ C++ Layer\nBridge between JS and OS]
        EL[🔄 Event Loop\nHandles async tasks]
        V8[🔵 V8 Engine\nExecutes JS]
        LIB[📦 libuv\nAsync I/O, Networking, Timers]
    end

    JS --> CPP
    CPP --> V8
    CPP --> EL
    EL --> LIB
    LIB --> OS[🖥️ Operating System]

    style JS fill:#FAEEDA,stroke:#854F0B,color:#633806
    style CPP fill:#FAECE7,stroke:#993C1D,color:#712B13
    style EL fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style V8 fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style LIB fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

### The Layers Explained

|Layer|What it does|
|---|---|
|**JS Layer**|Your code — written in JavaScript|
|**C++ Layer**|Translates JS calls into system-level operations|
|**V8 Engine**|Executes and compiles JS into machine code|
|**Event Loop**|Manages async tasks (timers, I/O callbacks)|
|**libuv**|Handles file system, networking, timers at OS level|

---

## 5. V8 Engine — Deep Dive

> V8 is a **JS engine** — it makes running JS code possible on a machine. V8 is majorly written in **C++**.

```mermaid
graph LR
    YC[Your JS Code] --> V8

    subgraph V8 Engine
        PA[① Parser\nTokenizes code\nBuilds AST]
        IG[② Interpreter\nIgnition\nRuns AST line by line]
        CO[③ Compiler\nTurboFan\nOptimizes hot code]
    end

    PA --> IG --> CO
    CO -->|fast machine code| MC[🖥️ Machine Execution]

    style PA fill:#FAECE7,stroke:#993C1D,color:#712B13
    style IG fill:#FAEEDA,stroke:#854F0B,color:#633806
    style CO fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

### Components of V8 Engine

#### ① Parser

- Takes raw JS code
- **Tokenizes** it (breaks into small pieces)
- Builds an **AST** (Abstract Syntax Tree)

**Example — Tokenization:**

```
let a = 1;
→ [ let, a, =, 1, ; ]
```

**AST = Abstract Syntax Tree**

```mermaid
graph TD
    ROOT[Program]
    ROOT --> DEC[VariableDeclaration\nlet]
    DEC --> ID[Identifier\na]
    DEC --> LIT[Literal\n1]

    style ROOT fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style DEC fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    style ID fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style LIT fill:#E1F5EE,stroke:#0F6E56,color:#085041
```

> 💡 **Why is a Parser needed?** Computers don't understand JS directly. The parser converts human-readable code into a structured tree that the engine can process step by step.

---

#### ② Interpreter — Ignition

- Takes the AST produced by the parser
- Converts it to **bytecode**
- Runs code **line by line** (portable, cross-platform)

```mermaid
graph LR
    AST[AST\nfrom Parser] --> IGN[Ignition\nInterpreter]
    IGN --> BC[Bytecode\nPortable format]
    BC --> RUN[▶️ Runs on any platform]

    style IGN fill:#FAEEDA,stroke:#854F0B,color:#633806
    style BC fill:#E6F1FB,stroke:#185FA5,color:#0C447C
```

> 📌 Bytecode is **portable** — can run cross-platform (Windows, Mac, Linux)

---

#### ③ Compiler — TurboFan

- Watches for **"hot" code** (functions called many times)
- Compiles hot code into optimized **machine code**
- Makes repeated execution much faster

```mermaid
graph LR
    BC[Bytecode] --> TF[TurboFan\nCompiler]
    TF --> MC[⚡ Optimized\nMachine Code]
    MC --> FAST[Much Faster\nExecution]

    style TF fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style MC fill:#EEEDFE,stroke:#534AB7,color:#3C3489
```

---

### Full V8 Pipeline — Summary

```mermaid
flowchart LR
    CODE[📝 JS Source Code]
    --> PARSE[Parser\nTokenize + AST]
    --> IGN[Ignition\nBytecode]
    --> TF[TurboFan\nOptimized Machine Code]
    --> CPU[🖥️ CPU Executes]

    style CODE fill:#F1EFE8,stroke:#5F5E5A,color:#444441
    style PARSE fill:#FAECE7,stroke:#993C1D,color:#712B13
    style IGN fill:#FAEEDA,stroke:#854F0B,color:#633806
    style TF fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style CPU fill:#EEEDFE,stroke:#534AB7,color:#3C3489
```

---

## 6. The Interpret vs Compile Dilemma

Running JS involves a classic trade-off:

```mermaid
graph TD
    D[Dilemma\nHow to run JS code?]
    D --> I[Interpret\nLine by line]
    D --> C[Compile\nAll at once]

    I --> IA[✅ Portable\nworks cross-platform]
    I --> IB[❌ Slower\nno optimization]

    C --> CA[✅ Fast\noptimized machine code]
    C --> CB[❌ Not portable\nplatform-specific]

    D --> SOL[✅ V8 Solution\nHybrid approach\nIgnition + TurboFan]

    style D fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    style SOL fill:#E1F5EE,stroke:#0F6E56,color:#085041
    style I fill:#FAECE7,stroke:#993C1D,color:#712B13
    style C fill:#FAEEDA,stroke:#854F0B,color:#633806
```

> V8 **combines both** — interpret first for portability, then compile hot paths for speed.

---

## Quick Summary

```mermaid
mindmap
  root((Node.js))
    What is it
      Open source runtime
      Cross-platform
      Created 2009 by Ryan Dahl
    Runtime Env
      Browser gives JS DOM + timers
      Node gives JS files + processes + network
    Internals
      JS Layer
      C++ Layer
      Event Loop
      libuv
    V8 Engine
      Parser → AST + tokens
      Ignition → bytecode
      TurboFan → machine code
```