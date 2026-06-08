# Node.js — Paths & File I/O

---
## 1. Absolute vs Relative Path

| | Absolute Path | Relative Path |
|---|---|---|
| **Starts from** | Root of the filesystem (`/` on Unix, `C:\` on Windows) | Current working directory |
| **Example** | `/home/binod/project/index.html` | `../index.html` |
| **Portability** | ❌ Machine-specific | ✅ Portable across machines |
| **Predictability** | ✅ Always the same | ⚠️ Depends on *where* you run the script |
```js
// Absolute
const path = '/home/binod/project/index.html'

// Relative (resolved from process.cwd())
const path = '../index.html'
```

> **Rule of thumb:** Use relative paths for project files, absolute paths when you need a guaranteed location (e.g. system configs).

---

## 2. `__dirname` Doesn't Work in ES Modules

`__dirname` and `__filename` are **CommonJS-only** globals. They are not available when using `"type": "module"` in `package.json` or `.mjs` files.

```js
// ❌ CommonJS only — breaks in ES Modules
const path = __dirname + '/index.html'
```

### ✅ The ES Module fix — `import.meta.url`

```js
import { fileURLToPath } from 'url'
import { dirname } from 'path'

// Recreate __filename and __dirname manually
const __filename = fileURLToPath(import.meta.url)
const __dirname  = dirname(__filename)

// Or use URL directly (simpler)
const filePath = new URL('../index.html', import.meta.url)
```

| | CommonJS | ES Module |
|---|---|---|
| Current file | `__filename` | `fileURLToPath(import.meta.url)` |
| Current dir | `__dirname` | `dirname(fileURLToPath(import.meta.url))` |
| Relative file | `path.join(__dirname, '../file')` | `new URL('../file', import.meta.url)` 

---

## 3. File I/O in Node.js

### Reading a file

```js
import { readFile } from 'fs/promises'

// As a string
const contents = await readFile('./index.html', { encoding: 'utf-8' })

// As a Buffer (binary)
const buffer = await readFile('./image.png')
```

### Writing a file

```js
import { writeFile } from 'fs/promises'

await writeFile('./output.html', contents)            // default: utf-8
await writeFile('./output.html', contents, 'utf-8')   // explicit encoding
```

### Appending to a file

```js
import { appendFile } from 'fs/promises'

await appendFile('./log.txt', 'New log entry\n')
```

### Checking if a file exists

```js
import { access } from 'fs/promises'

try {
  await access('./index.html')
  console.log('File exists')
} catch {
  console.log('File not found')
}
```

### Quick reference

|Task|Method|
|---|---|
|Read file|`readFile(path, { encoding: 'utf-8' })`|
|Write file|`writeFile(path, data)`|
|Append to file|`appendFile(path, data)`|
|Delete file|`unlink(path)`|
|List directory|`readdir(path)`|
|File metadata|`stat(path)`|

> Always use `fs/promises` with `await` — avoids callback hell and is the modern Node.js standard.

---

## Putting it all together

```js
import { readFile, writeFile } from 'fs/promises'

// Safe path using import.meta.url (works in ES Modules)
const inputPath  = new URL('../index.html',    import.meta.url)
const outputPath = new URL('../newindex.html', import.meta.url)

let contents = await readFile(inputPath, { encoding: 'utf-8' })

const data = { name: 'Binod', message: 'Happy coding' }

for (const [key, value] of Object.entries(data)) {
  contents = contents.replaceAll(`{{${key}}}`, value)
}

await writeFile(outputPath, contents)
```