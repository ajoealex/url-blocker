# URL Blocker Project

A comprehensive URL blocking solution consisting of a Chrome browser extension and a reporting server. This project enables users to block websites based on customizable URL patterns, with optional reporting capabilities and automatic tab closing functionality.

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Features](#features)
- [Components](#components)
  - [URL Blocker Chrome Extension](#url-blocker-chrome-extension)
  - [URL Blocker Listener Server](#url-blocker-listener-server)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [API Reference](#api-reference)
- [Development](#development)
- [Building](#building)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Overview

The URL Blocker Project provides a two-component system for controlling web access in Chrome/Chromium browsers:

1. **Chrome Extension (url-blocker-plugin)**: A manifest v3 extension that blocks websites based on URL patterns using Chrome's declarativeNetRequest API
2. **Reporting Server (url-blocker-listener)**: A Node.js Express server that receives and stores reports about blocked URLs

Together, these components allow users to:
- Define URL patterns to block specific websites or domains
- Automatically report blocked access attempts to a central server
- Automatically close tabs that attempt to access blocked content
- Monitor and analyze blocked URL requests in real-time

## Architecture

```
┌─────────────────────────────────────┐
│   Chrome Browser                    │
│                                     │
│  ┌──────────────────────────────┐   │
│  │  URL Blocker Extension       │   │
│  │  - Pattern Matching          │   │
│  │  - Request Blocking          │   │
│  │  - Tab Management            │   │
│  └──────────────┬───────────────┘   │
│                 │                   │
└─────────────────┼───────────────────┘
                  │ HTTP POST
                  │ (Blocked URL Reports)
                  ▼
         ┌────────────────────┐
         │  Listener Server   │
         │  - Receives Reports│
         │  - Stores in Memory│
         │  - Provides API    │
         └────────────────────┘
```

## Project Structure

```
url-blocker-plugin/
├── README.md                          # This file
├── url-blocker-plugin/               # Chrome extension source
│   ├── manifest.json                 # Extension manifest (v3)
│   ├── background.js                 # Service worker for blocking logic
│   ├── popup.html                    # Extension popup UI
│   ├── popup.js                      # Popup interface logic
│   └── popup.css                     # Popup styling
│
└── url-blocker-listener/             # Reporting server
    ├── server.js                     # Express server implementation
    ├── app.properties                # Server configuration
    ├── package.json                  # Node.js dependencies
    ├── README.md                     # Server-specific documentation
    └── dist/                         # Built executables (after build)
```

## Features

### Chrome Extension Features

- **Pattern-based URL Blocking**: Block websites using flexible URL patterns with wildcard support
- **Manifest V3 Compliance**: Uses the latest Chrome extension standard with declarativeNetRequest API
- **Real-time Rule Updates**: Dynamically add/remove blocking rules without restarting the browser
- **Cross-device Synchronization**: Patterns sync across devices via Chrome Sync storage
- **Blocked Resource Types**: Blocks both main frames (direct navigation) and sub-frames (iframes)
- **Optional Reporting**: Send blocked URL details to a configured reporting server
- **Server Validation**: Automatic ping check to validate server connectivity before enabling reporting
- **Auto-Close Tabs**: Automatically close tabs that attempt to access blocked content after a configurable delay
- **Flexible Configuration**: Separate enable/disable controls for reporting and tab closing features

### Server Features

- **RESTful API**: Simple HTTP endpoints for receiving and querying blocked URL data
- **In-Memory Storage**: Fast, lightweight storage with configurable retention limits
- **Health Monitoring**: Ping endpoint for server health checks
- **Configurable Limits**: Control how many blocked URL entries to retain
- **FIFO Queue**: Automatically removes oldest entries when limit is reached
- **Standalone Executables**: Build platform-specific executables (Windows, Linux, macOS)
- **Cross-Platform**: Runs on any platform with Node.js or as standalone executable
- **No Database Required**: Simple in-memory storage for easy deployment

## Components

### URL Blocker Chrome Extension

#### Technical Details

- **Manifest Version**: 3
- **Permissions Required**:
  - `storage` - For saving blocked patterns
  - `declarativeNetRequest` - For blocking URLs
  - `declarativeNetRequestFeedback` - For tracking blocked requests
  - `tabs` - For auto-closing tabs
  - `<all_urls>` - For applying rules to all websites

#### Key Files

- **[manifest.json](url-blocker-plugin/manifest.json)** - Extension configuration and permissions
- **[background.js](url-blocker-plugin/background.js)** - Service worker handling:
  - Dynamic rule management
  - Blocked request detection
  - Reporting system
  - Tab auto-closing
  - Settings synchronization
- **[popup.js](url-blocker-plugin/popup.js)** - User interface logic:
  - Pattern management (add/delete)
  - Pattern validation
  - Reporting configuration
  - Tab closing configuration
  - Server connectivity testing
- **[popup.html](url-blocker-plugin/popup.html)** - Extension popup interface
- **[popup.css](url-blocker-plugin/popup.css)** - UI styling

#### Pattern Format

URL patterns follow Chrome's match pattern format:

```
*://*.example.com/*        # Block all of example.com
*://facebook.com/*         # Block specific domain
*://*.youtube.com/watch*   # Block YouTube videos
https://example.com/*      # HTTPS only
http://example.com/*       # HTTP only
```

**Pattern Requirements**:
- Must start with protocol (`*://`, `http://`, or `https://`)
- Must include at least one forward slash
- Minimum length of 5 characters
- Supports wildcards (`*`) for flexible matching

### URL Blocker Listener Server

#### Technical Details

- **Runtime**: Node.js 18+
- **Framework**: Express.js 4.18.2
- **Configuration**: Properties file
- **Storage**: In-memory (non-persistent)

#### Key Files

- **[server.js](url-blocker-listener/server.js)** - Main server implementation
- **[app.properties](url-blocker-listener/app.properties)** - Configuration file
- **[package.json](url-blocker-listener/package.json)** - Dependencies and build scripts

#### Data Model

Each blocked URL request contains:

```json
{
  "url": "https://example.com",           // Blocked URL
  "timestamp": "2025-12-28T02:44:45.437Z", // When it was blocked
  "tabId": 1772131031,                    // Chrome tab ID
  "type": "sub_frame",                    // Resource type
  "initiator": "https://www.youtube.com", // Originating page
  "reportedAt": "2025-12-28T02:44:45.437Z" // When reported to server
}
```

## Installation

### Chrome Extension Installation

#### Option 1: Developer Mode (Recommended for Development)

1. Open Chrome and navigate to `chrome://extensions/`
2. Enable "Developer mode" in the top right
3. Click "Load unpacked"
4. Select the `url-blocker-plugin` directory
5. The extension icon should appear in your toolbar

#### Option 2: Package as CRX (For Distribution)

1. Go to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Pack extension"
4. Select the `url-blocker-plugin` directory
5. Distribute the generated `.crx` file

### Server Installation

#### Option 1: Run from Source

1. Navigate to the server directory:
   ```bash
   cd url-blocker-listener
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Configure the server (optional):
   Edit `app.properties`:
   ```properties
   port=9874
   max_requests=10
   ```

4. Start the server:
   ```bash
   npm start
   ```

   Or for development with auto-reload:
   ```bash
   npm run dev
   ```

#### Option 2: Use Standalone Executable

1. Build the executable (see [Building](#building) section)
2. Copy the executable from `dist/` folder
3. Copy `app.properties` to the same directory as the executable
4. Run the executable:

   **Windows:**
   ```bash
   .\url-blocker-listener.exe
   ```

   **Linux/macOS:**
   ```bash
   ./url-blocker-listener
   ```

## Configuration

### Extension Configuration

All configuration is done through the extension popup interface:

1. Click the extension icon in Chrome toolbar
2. Add URL patterns to block in the input field
3. Enable/disable reporting and configure endpoint URL
4. Enable/disable auto-close tabs and configure delay interval

Settings are automatically synced across devices via Chrome Sync.

### Server Configuration

Edit `app.properties` in the server directory:

```properties
# Server port (default: 9874)
port=9874

# Maximum number of blocked URL requests to retain in memory
# Older requests are automatically removed when this limit is reached
max_requests=10
```

**Configuration Notes**:
- When running as source, the file is loaded from the script directory
- When running as executable, the file must be in the current working directory
- If the file is missing, defaults are used (port=3000, max_requests=10)

### Reporting Setup

To enable reporting from the extension to the server:

1. Start the URL Blocker Listener server (default port 9874)
2. Open the extension popup
3. Check "Report blocked URLs"
4. Enter the endpoint URL (e.g., `http://localhost:9874`)
5. Click "Save Settings"
6. The extension will validate the server connection with a ping test
7. If successful, reporting will be enabled

**Important**: The server must be running and accessible before enabling reporting. The extension automatically tests connectivity using the `/ping` endpoint.

### Auto-Close Tabs Setup

To automatically close tabs that try to access blocked content:

1. Open the extension popup
2. Check "Close blocked tabs after interval"
3. Enter the delay in seconds (1-3600)
4. Click "Save Settings"

When enabled, any tab that attempts to navigate to a blocked URL will be automatically closed after the configured delay.

## Usage

### Blocking Websites

1. Click the URL Blocker extension icon
2. Enter a URL pattern (e.g., `*://*.facebook.com/*`)
3. Click "Add Pattern" or press Enter
4. The website will be immediately blocked across all tabs

### Viewing Blocked Patterns

All active patterns are displayed in the extension popup. Click "Delete" next to any pattern to remove it.

### Pattern Examples

| Pattern | What It Blocks |
|---------|----------------|
| `*://*.facebook.com/*` | All Facebook pages (HTTP/HTTPS) |
| `*://*.youtube.com/watch*` | YouTube video pages only |
| `*://example.com/*` | Specific domain only |
| `https://*.gambling-site.com/*` | HTTPS only for gambling sites |
| `*://*.ad-network.com/*` | Entire ad network domain |

### Monitoring Blocked URLs

#### View All Blocked URLs

```bash
curl http://localhost:9874/
```

Response:
```json
{
  "requests": [
    {
      "url": "https://example.com",
      "timestamp": "2025-12-28T02:44:45.437Z",
      "tabId": 1772131031,
      "type": "sub_frame",
      "initiator": "https://www.youtube.com",
      "reportedAt": "2025-12-28T02:44:45.437Z"
    }
  ],
  "totalRequests": 1
}
```

#### View Latest Blocked URL

```bash
curl http://localhost:9874/?latest=true
```

Response:
```json
{
  "latest": {
    "url": "https://example.com",
    "timestamp": "2025-12-28T02:44:45.437Z",
    "tabId": 1772131031,
    "type": "sub_frame",
    "initiator": "https://www.youtube.com",
    "reportedAt": "2025-12-28T02:44:45.437Z"
  },
  "totalRequests": 1
}
```

#### Clear All Stored Requests

```bash
curl -X DELETE http://localhost:9874/cleanup
```

Response:
```json
{
  "message": "All blocked URL requests cleared",
  "clearedCount": 5
}
```

#### Health Check

```bash
curl http://localhost:9874/ping
```

Response:
```json
{
  "status": "ok",
  "message": "Server is running",
  "timestamp": "2025-12-28T02:44:45.437Z"
}
```

## API Reference

### POST /

Receive a blocked URL report from the Chrome extension.

**Request Body:**
```json
{
  "blockedUrl": {
    "url": "https://example.com",
    "timestamp": "2025-12-28T02:44:45.437Z",
    "tabId": 1772131031,
    "type": "sub_frame",
    "initiator": "https://www.youtube.com"
  },
  "reportedAt": "2025-12-28T02:44:45.437Z"
}
```

**Response (200 OK):**
```json
{
  "message": "Blocked URL recorded successfully",
  "totalRequests": 5
}
```

**Error Response (400 Bad Request):**
```json
{
  "error": "Missing required fields: blockedUrl and reportedAt"
}
```

### GET /

Retrieve stored blocked URL requests.

**Query Parameters:**
- `latest=true` (optional) - Return only the most recent request

**Response (All Requests):**
```json
{
  "requests": [...],
  "totalRequests": 10
}
```

**Response (Latest Only):**
```json
{
  "latest": {...},
  "totalRequests": 10
}
```

### GET /ping

Health check endpoint to verify server is running.

**Response (200 OK):**
```json
{
  "status": "ok",
  "message": "Server is running",
  "timestamp": "2025-12-28T02:44:45.437Z"
}
```

### DELETE /cleanup

Clear all stored blocked URL requests from memory.

**Response (200 OK):**
```json
{
  "message": "All blocked URL requests cleared",
  "clearedCount": 10
}
```

## Development

### Extension Development

#### File Watching

Chrome automatically reloads unpacked extensions when files change. During development:

1. Make changes to extension files
2. Click the extension reload button in `chrome://extensions/`
3. Test changes immediately

#### Debugging

- **Background Script**: Right-click extension icon → "Inspect service worker"
- **Popup**: Right-click popup → "Inspect"
- **Console Logs**: All blocking events are logged to the service worker console

### Server Development

#### Development Mode

```bash
cd url-blocker-listener
npm run dev
```

Uses `nodemon` for automatic server restart on file changes.

#### Production Mode

```bash
npm start
```

Runs the server without auto-reload.

#### Testing

Test the server endpoints with curl:

```bash
# Health check
curl http://localhost:9874/ping

# Send test blocked URL
curl -X POST http://localhost:9874/ \
  -H "Content-Type: application/json" \
  -d '{
    "blockedUrl": {
      "url": "https://test.com",
      "timestamp": "2025-12-28T10:00:00.000Z",
      "tabId": 123,
      "type": "main_frame",
      "initiator": "https://google.com"
    },
    "reportedAt": "2025-12-28T10:00:00.000Z"
  }'

# Retrieve all
curl http://localhost:9874/

# Get latest
curl http://localhost:9874/?latest=true

# Clean up
curl -X DELETE http://localhost:9874/cleanup
```

## Building

### Building Server Executables

The server can be packaged as standalone executables for distribution without requiring Node.js installation.

#### Build for Current Platform

```bash
cd url-blocker-listener
npm run build
```

#### Build for Specific Platforms

**Windows (x64):**
```bash
npm run build:win
```

**Linux (x64):**
```bash
npm run build:linux
```

**macOS (x64):**
```bash
npm run build:macos
```

**All Platforms:**
```bash
npm run build:all
```

#### Build Output

Executables are created in the `dist/` directory:
- `url-blocker-listener-win.exe` (Windows)
- `url-blocker-listener-linux` (Linux)
- `url-blocker-listener-macos` (macOS)

#### Distribution

When distributing executables:
1. Include the executable
2. Include `app.properties` configuration file
3. Place both files in the same directory
4. Provide instructions to run the executable

## Troubleshooting

### Extension Issues

#### Extension Not Blocking Sites

1. **Check patterns are active**: Open extension popup and verify patterns are listed
2. **Verify pattern format**: Patterns must include protocol and path (e.g., `*://*.example.com/*`)
3. **Check resource type**: Extension only blocks `main_frame` and `sub_frame` requests
4. **Reload extension**: Go to `chrome://extensions/` and click reload
5. **Check console**: Inspect service worker for error messages

#### Reporting Not Working

1. **Server running**: Verify server is accessible at configured endpoint
2. **Network connectivity**: Check firewall/network settings
3. **Endpoint URL**: Ensure URL is correct (include protocol and port)
4. **Server validation**: The extension pings the server at `/ping` - check this works manually
5. **Check console**: Look for fetch errors in service worker console

#### Tab Not Auto-Closing

1. **Feature enabled**: Verify "Close blocked tabs after interval" is checked
2. **Interval configured**: Ensure interval is set and saved
3. **Tab ID valid**: Some special tabs (like chrome:// pages) cannot be closed
4. **Check console**: Service worker logs tab closing attempts and errors

### Server Issues

#### Server Won't Start

1. **Port in use**: Check if port 9874 (or configured port) is already in use
2. **Properties file**: Verify `app.properties` exists and is readable
3. **Node.js version**: Ensure Node.js 18+ is installed (`node --version`)
4. **Dependencies**: Run `npm install` to ensure all packages are installed

#### Can't Connect to Server

1. **Server running**: Check server console shows "Server is running" message
2. **Correct port**: Verify port in `app.properties` matches endpoint URL
3. **Firewall**: Check firewall allows connections on the configured port
4. **Localhost vs 0.0.0.0**: Server binds to all interfaces, but you may need to use correct IP
5. **HTTPS**: If extension uses HTTPS, you may need to configure SSL

#### Requests Not Being Stored

1. **Check POST body**: Ensure requests include required fields (`blockedUrl`, `reportedAt`)
2. **Content-Type**: Must be `application/json`
3. **Check logs**: Server logs errors to console
4. **Memory limit**: Check if `max_requests` limit has been reached (older entries are removed)

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| "Invalid pattern format" | Pattern doesn't match required format | Use `*://domain.com/*` format |
| "Pattern already exists" | Duplicate pattern | Remove existing pattern first |
| "Server validation failed" | Can't connect to server | Verify server is running and accessible |
| "Missing required fields" | Invalid POST body | Check JSON structure matches API spec |
| "Port already in use" | Another process using the port | Change port in app.properties or stop other process |

### Debug Mode

#### Extension Debug Mode

Enable detailed logging in [background.js](url-blocker-plugin/background.js#L92) - all events are already logged to console. View via:
1. Go to `chrome://extensions/`
2. Click "Inspect service worker" under URL Blocker
3. View Console tab for real-time logs

#### Server Debug Mode

Server logs all requests and errors to console by default. For more details, modify [server.js](url-blocker-listener/server.js) to add additional logging.

## Security Considerations

### Extension Permissions

The extension requests broad permissions for blocking functionality:
- **`<all_urls>`**: Required to apply blocking rules to any website
- **`declarativeNetRequest`**: Required to block network requests
- **`tabs`**: Required for auto-closing functionality
- **`storage`**: Required to save patterns and settings

These permissions are necessary for the extension's core functionality and cannot be reduced.

### Server Security

The server currently:
- **Does not use authentication**: Anyone with access can send/retrieve data
- **Stores data in memory**: All data is lost on restart (intentional design)
- **No HTTPS**: Uses HTTP by default
- **No rate limiting**: Could be flooded with requests

For production use, consider:
- Adding authentication/authorization
- Implementing rate limiting
- Using HTTPS with proper certificates
- Implementing persistent storage if needed
- Restricting CORS to known origins

### Privacy

The reporting feature sends:
- Blocked URLs
- Timestamps
- Tab IDs (numeric identifiers)
- Resource types
- Initiating domains

This information is sent to the configured endpoint. Only enable reporting if you control the server or trust the recipient.

## Known Limitations

1. **In-memory storage**: Server data is lost on restart (by design for simplicity)
2. **Resource types**: Extension only blocks main_frame and sub_frame (not images, scripts, etc.)
3. **Pattern syntax**: Limited to Chrome's URL match pattern format (no full regex)
4. **No persistent logs**: Extension doesn't maintain a local log of blocked requests
5. **Network dependent**: Reporting requires network connectivity to server
6. **Tab closing**: Cannot close certain special tabs (chrome://, edge://, etc.)

## Future Enhancements

Potential improvements for future versions:

- Persistent storage option for server (database support)
- Authentication for server API
- Scheduled pattern rules (time-based blocking)
- Allow/block lists (exceptions to patterns)
- Statistics and analytics dashboard
- Export/import pattern configurations
- Browser action badge showing blocked count
- Notification system for blocked attempts
- Mobile browser support (where possible)
- Pattern testing/validation tool

## License

ISC

