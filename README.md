# slidev-addon-second-screen

A [Slidev](https://sli.dev) addon that adds a button to the presenter view for opening the presentation on a second screen, similar to PowerPoint's presenter view.

## Features

- **Open on Second Screen**: Opens the slide presentation on an external display, always on the screen where the presenter is NOT running
- **Fullscreen Prompt**: Automatically requests fullscreen mode with a fallback dialog (supports Enter/Escape/X-button)
- **Visual Status**: Button icon shows whether the second screen is active (filled) or inactive (outline)
- **Auto-Detect Close**: Button resets when the second screen window is manually closed
- **Feature Detection**: Gracefully handles browsers without Window Management API support
- **Automatic Sync**: Leverages Slidev's built-in navigation sync between windows

## Requirements

- **Browser**: Chrome 100+ or Edge 100+ (Window Management API required)
- **Secure Context**: HTTPS or `localhost` (required for Window Management API)
- **Multiple Screens**: At least two connected displays

## Installation

```bash
npm install slidev-addon-second-screen
```

## Usage

Add the addon to your Slidev presentation:

### Option 1: In `slides.md` frontmatter

```yaml
---
addons:
  - slidev-addon-second-screen
---
```

### Option 2: In `package.json`

```json
{
  "slidev": {
    "addons": ["slidev-addon-second-screen"]
  }
}
```

## How It Works

1. **Open on Second Screen**: When you click the button in the presenter view, the addon:
   - Requests permission to access screen information
   - Detects the screen where the presenter is NOT running
   - Opens a new window on that screen with the slide presentation
   - Automatically requests fullscreen mode on the second screen

2. **Fullscreen Prompt**: If automatic fullscreen fails, a dialog appears with:
   - "Enter Fullscreen" button
   - Keyboard shortcuts: Enter (fullscreen), Escape (close dialog)
   - X button to dismiss the dialog

3. **Navigation Sync**: Slidev automatically syncs navigation between all open windows, so when you advance slides in the presenter view, the second screen updates accordingly.

## Browser Support

| Feature | Chrome | Firefox | Safari |
|---------|--------|---------|--------|
| `getScreenDetails()` | 100+ | ❌ | ❌ |
| `requestFullscreen()` | 100+ | ❌ | ❌ |

The addon will:
- Show the button only in supported browsers
- Display a disabled icon in unsupported browsers
- Gracefully hide functionality when features are unavailable

## HTTPS Requirement

The Window Management API requires a **Secure Context**:
- `http://localhost` and `http://127.0.0.1` are considered secure ✅
- Any other hostname requires HTTPS
- For hosted slides (GitHub Pages, Netlify, etc.), HTTPS is automatic

**No HTTPS configuration needed for local development** when using `localhost`.

## Permissions

On first use, the browser will prompt for permission to:
- Access information about your displays
- Open and place windows on specific screens

You can manage these permissions in your browser's site settings.

## Troubleshooting

### Button appears disabled
- Ensure you're using Chrome 100+ or Edge 100+
- Check that you're accessing via `localhost` or HTTPS
- Verify that multiple screens are connected

### Popup blocked
- Allow popups for your Slidev site in browser settings
- The addon needs to open a new window for the second screen

### Fullscreen not working
- Fullscreen must be triggered by a user gesture (clicking the button)
- Some browsers may require additional interaction before allowing fullscreen
- Use the fullscreen prompt dialog as a fallback

### Screens not syncing
- Ensure both windows are connected to the same Slidev dev server
- In build mode, sync uses BroadcastChannel (same browser, same device)
- In dev mode, sync uses the Vite dev server

## Development

For local development:

```bash
# Clone the repository
git clone https://github.com/yourusername/slidev-addon-second-screen.git
cd slidev-addon-second-screen

# Create a test slides.md
cat > slides.md << 'EOF'
---
addons:
  - ./
---

# Test Slide

This is a test presentation for the second screen addon.

---

# Second Slide

Navigate between slides to test sync.
EOF

# Start Slidev
npm install
npm run dev
```

## License

MIT

## Author

Denis Sowa

## Contributing

Contributions are welcome! Please open an issue or pull request on GitHub.
