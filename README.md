# Mythic Editor

---

## Highlights

- **Piece table storage** — an augmented red-black tree over immutable string buffers gives `O(log N)` inserts and deletes without copying the whole document.
- **Virtualized rendering** — only the lines inside the viewport (plus an overscan buffer) exist in the DOM; line elements are recycled while scrolling.
- **Mobile-first input** — an invisible textarea bridge handles keyboard, Android Gboard, and IME composition, so typing works on phones as well as desktops.
- **Unicode-correct** — positions never split a surrogate pair; astral characters (emoji, Devanagari, CJK, RTL) stay intact across edits.
- **Framework-free core** — the buffer and model layers run in plain ES2022 with no DOM dependency, so they can later move to WebAssembly or a worker.

---

## Getting Started

Requirements: Node.js 18+ and npm.

```bash
npm install      # install dev dependencies
npm run dev      # start the interactive playground (http://localhost:5173)
```

Other scripts:

| Command             | Description                                    |
| ------------------- | ---------------------------------------------- |
| `npm run build`     | Build the library bundle into `dist/`          |
| `npm test`          | Run the Vitest suite once                      |
| `npm run test:watch`| Run tests in watch mode                        |
| `npm run typecheck` | Type-check the project with `tsc --noEmit`     |

The playground (`index.html` + `src/playground/`) is a live demo of the engine: it mounts the editor view, shows buffer metrics and cursor/selection state, and lets you insert/delete text and convert offsets to positions.

---

## Usage

### Text buffer (no DOM)

```ts
import { PieceTreeTextBuffer } from '@mythic/core';

const buffer = new PieceTreeTextBuffer('const x = 1;\n', '\n');

buffer.insert(0, '// header\n');
buffer.deleteRange({ startLineNumber: 2, startColumn: 1, endLineNumber: 2, endColumn: 6 });
buffer.applyEdit({ startLineNumber: 1, startColumn: 1, endLineNumber: 1, endColumn: 3 }, 'let');

buffer.getText();                 // full document
buffer.getLineContent(1);         // first line, without EOL
buffer.getLength();               // length in UTF-16 code units
buffer.getPositionAt(8);          // { lineNumber, column }
buffer.getOffsetAt({ lineNumber: 1, column: 9 });
```

### Editor view (browser)

```ts
import { PieceTreeTextBuffer } from '@mythic/core';
import { EditorView } from '@mythic/view';

const buffer = new PieceTreeTextBuffer('hello world\n', '\n');

const editor = new EditorView(document.getElementById('editor-mount')!, buffer, {
  fontSize: 14,
  fontFamily: 'ui-monospace, Menlo, monospace',
  onDidChangeContent: (b) => console.log(b.getLength()),
  onSave: (text) => console.log('save', text)
});

editor.focus();
editor.setSelection(/* Selection */);
editor.render();
```

Call `editor.layout()` and `editor.render()` after the container is resized.

---

## Coordinate Conventions

All subsystems follow one set of rules:

| Concept  | Convention                                                        |
| -------- | ----------------------------------------------------------------- |
| Lines    | 1-based (`lineNumber >= 1`)                                       |
| Columns  | 1-based (`column >= 1`)                                           |
| Offsets  | 0-based, in UTF-16 code units (matches JS strings and LSP)        |
| Ranges   | Normalized (`start <= end`)                                       |
| Selection| Directional range with anchor and active cursor (LTR or RTL)      |
| Surrogates | String offsets never land between a high and low surrogate      |

---

More detail on design decisions and invariants is in [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Testing

```bash
npm test
```

The suite covers the piece-tree buffer (insert, delete, ranges), Unicode and surrogate handling, the core model types, and viewport virtualization math.

---

## Roadmap

- **Phase 1** — Piece-tree buffer and core model types. ✅
- **Phase 2** — Virtualized view, DOM line rendering, cursor/selection overlays, touch and IME input. ✅
- **Next** — Syntax highlighting, multi-cursor editing, undo/redo, and an optional WebAssembly storage backend.

---

## Attribution

The piece table, red-black tree, font measurement, line recycling, cursor, selection, and input bridge implementations are adapted from the MIT-licensed [Monaco Editor](https://github.com/microsoft/monaco-editor) / [VS Code](https://github.com/microsoft/vscode) source. Component-by-component attribution is in [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md).

## License

MIT © Crest-Editor. See [LICENSE](LICENSE).
