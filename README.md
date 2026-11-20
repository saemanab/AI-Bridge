# AI Bridge

Send your VS Code code to **Google AI Studio** in one click.

AI Bridge is two small extensions that talk to each other on your own machine:

- **VS Code extension**: runs a tiny local server (`127.0.0.1:8999`) that can read your active file or your whole project.
- **Chrome extension**: asks that server for the code and either copies it to your clipboard or pastes it straight into the AI Studio prompt box.

Nothing is sent anywhere except to the page you paste into. The server only listens on `localhost` and requires a secret token.

## Features

- **Copy Entire Project**: bundles all eligible source files into one Markdown block (path + code fences).
- **Copy Active File**: copies the file open in VS Code, including unsaved edits.
- **Paste Project / Paste File buttons inside AI Studio**, next to the *Tools* button. No clipboard step needed.

## Requirements

- Google Chrome (or a Chromium browser such as Edge or Brave)
- VS Code 1.85 or newer

## Setup

### 1. Choose a token

Pick a long random string. It must be **identical** in both places:

- `vscode-extension/src/extension.js` → `BRIDGE_TOKEN`
- `chrome-extension/config.js` → `BRIDGE_TOKEN`

Generate one with any of these (each prints a 64-character hex string):

**Windows (PowerShell)**

```powershell
$bytes = New-Object byte[] 32
$rng = [Security.Cryptography.RandomNumberGenerator]::Create()
$rng.GetBytes($bytes)
$token = [BitConverter]::ToString($bytes).Replace('-','').ToLower()
$token
```

**macOS / Linux**

```bash
openssl rand -hex 32
```

**Any OS with Node.js**

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Copy the output and paste it as the value of `BRIDGE_TOKEN` in both files, for example:

```js
const BRIDGE_TOKEN = "561a8d...your-own-64-char-token...";
```

> Never commit your real token. Keep the placeholder in the repo and only set the real value locally.

### 2. Start the VS Code side

1. Open the `vscode-extension` folder in VS Code.
2. Press **F5**. A second window opens (Extension Development Host).
3. In that second window, **open the project folder you want to share**.

The server starts automatically. If you see "set your token", you skipped step 1.

### 3. Load the Chrome side

1. Go to `chrome://extensions`.
2. Turn on **Developer mode**.
3. Click **Load unpacked** and select the `chrome-extension` folder.

After changing any file later, click the reload icon on the extension card, and refresh the AI Studio tab.

## Usage

### Option A: paste inside AI Studio (fastest)

1. Open <https://aistudio.google.com>.
2. In the prompt bar, click **Paste Project** or **Paste File** (next to *Tools*).
3. The code is inserted at the end of the prompt box. Type your question and press Run.

### Option B: use the toolbar popup

1. Click the AI Bridge icon in Chrome.
2. Click **Save & Reconnect** to check the connection.
3. Click **Copy Entire Project** or **Copy Active File**.
4. Press **Ctrl+V** in AI Studio.

## What gets included

| Rule | Value |
| --- | --- |
| Workspace | First folder open in the VS Code window |
| Max file size | 300 KB |
| Max total size | 2 MB |
| Max files | 250 |
| Skipped folders | `.git`, `node_modules`, `vendor`, `dist`, `build`, `out`, `.next`, `.nuxt`, `.cache`, `.idea`, `coverage`, `.venv`, `venv`, `__pycache__`, `target`, `pods`, `bin`, `obj` |
| Skipped files | `.env*`, private keys, certificates, and names containing `secret`, `credential`, `token`, `password`, `private` |
| Binary files | Ignored |

"Copy Entire Project" reads files **from disk**, so save first. "Copy Active File" reads the editor and includes unsaved changes.

If the limits are hit, the bundle includes a warning that it may be incomplete.

## Troubleshooting

| Message / symptom | Fix |
| --- | --- |
| `Unauthorized. Check the shared bridge token.` | The two tokens don't match. Make them identical and reload both extensions. |
| `Cannot reach the VS Code bridge` / `Offline` | The VS Code window isn't running (press F5 again) or another app is using port 8999. |
| `Open your project folder...` | Open a folder in the Extension Development Host window, not in your original VS Code window. |
| `No active text editor` | Click inside a file in VS Code first. |
| Buttons don't appear in AI Studio | Reload the extension, then refresh the AI Studio tab. If AI Studio changed its layout, the buttons fall back to a floating pair at the bottom-right. |
| `Prompt box not found` | Click on the AI Studio page and make sure the prompt box is visible. |
| `Extension was reloaded. Refresh this page.` | Refresh the AI Studio tab after reloading the extension. |

## Security notes

- The server binds to `127.0.0.1` only and rejects requests without the token.
- The token sits in plain text in two files. Treat it like a password and keep it out of Git.
- **Review the bundle before sending it to any AI service.** The secret-file filter is name-based and cannot catch keys hard-coded inside normal source files.

## Project layout

```
ai-bridge/
├── vscode-extension/     # local bridge server inside VS Code
│   ├── package.json
│   └── src/extension.js
└── chrome-extension/     # popup + AI Studio paste buttons
    ├── manifest.json
    ├── config.js         # bridge URL + token
    ├── popup.html / popup.css / popup.js
    ├── background.js     # talks to the bridge for the page
    └── content.js / content.css   # paste buttons in AI Studio
```

## Changelog

- **0.4.0**: Paste buttons inside AI Studio.
- **0.3.1**: Fixed copy buttons being permanently disabled; clipboard fallback.
- **0.3.0**: Popup no longer pings the bridge on open; 2.5 s request timeout.
- **0.2.0**: Added Copy Entire Project.

## License
abiaba mohamed
