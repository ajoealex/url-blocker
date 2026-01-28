# Tab URL Blocker 404 -- Architecture and Security Documentation

## Source Code Repository

https://github.com/ajoealex/url-blocker

## Chrome Extension Download

https://chromewebstore.google.com/detail/tab-url-blocker-404/kokjfejdghjdinfjhhpbihljmcfpjaci

## URL Blocker Listener (Binary Download)

https://github.com/ajoealex/url-blocker/releases

------------------------------------------------------------------------

## Solution Overview

Tab URL Blocker 404 is a Chrome Manifest V3 extension that enforces
website blocking using Chrome's declarativeNetRequest engine. It
supports optional audit and reporting through a local or centrally
hosted listener service.

The solution consists of two components:

### Chrome Extension

Whitelisted and managed within the organization. Responsible for URL
blocking and optional reporting.

### URL Blocker Listener

A standalone service (distributed as a platform executable) that
receives tab block events over HTTP and stores them in memory.

For ACCELQ automation use cases, it is recommended to start the listener
service on the same machine where the ACCELQ agent is installed and
running.

------------------------------------------------------------------------

## Architecture Diagram

<img width="3840" height="2160" alt="URL Blocker Plugin (1)" src="https://github.com/user-attachments/assets/d0a54855-d639-4669-8cff-3962fd028ffb" />


------------------------------------------------------------------------

## Extension Capabilities

### Core Functionality

-   Blocks navigation requests that match user-defined URL patterns\
-   Uses Manifest V3 and Chrome's declarative rules engine\
-   No request interception proxies\
-   No content-level traffic inspection\
-   Blocking is enforced natively by Chrome

### Optional Capabilities

**Reporting (Optional)**\
Sends blocked URL events to a user-configured URL Blocker Listener
endpoint over HTTP POST.\
Reporting is enabled only when the listener service is running,
configured, and reachable.

**Auto-close Tabs (Optional)**\
Automatically closes tabs after a configurable delay when a blocked URL
is accessed.

------------------------------------------------------------------------

## Data Flow Model

### Blocking Flow (Local-Only)

-   User configures URL patterns in the extension\
-   Extension updates declarativeNetRequest rules\
-   Chrome blocks matching requests locally within the browser

**Result:**\
All blocking is performed inside the browser. No network traffic is
generated.

### Reporting Flow (Opt-In)

-   Extension validates listener availability via `/ping`\
-   If enabled:
    -   Block events are sent to the listener via HTTP POST\
    -   Listener stores events in memory for retrieval

**Security boundary:**\
Reporting introduces a network path. If reporting is disabled, no block
data leaves the browser.

------------------------------------------------------------------------

## Permissions and Security Posture

  Permission                      Purpose
  ------------------------------- -------------------------------------
  storage                         Store patterns and configuration
  declarativeNetRequest           Apply blocking rules
  declarativeNetRequestFeedback   Detect blocked requests
  tabs                            Enable tab auto-close functionality
  `<all_urls>`                    Apply rules across all sites

**Security rationale:**\
These permissions are broad by necessity for network-level blocking.
Risk is mitigated architecturally by using MV3 declarative rules, not
runtime interception, content injection, or script-based scraping.

------------------------------------------------------------------------

## URL Blocker Listener -- Security Model

### What the Listener Is

The listener is distributed as a prebuilt standalone executable for
Windows, Linux, and macOS. It does not require Node.js installation on
endpoints.

### Functional Behavior

-   Starts an HTTP service on a configurable port\
-   Receives blocked URL events from the extension\
-   Stores events in a FIFO in-memory queue\
-   Exposes endpoints for health checks and event queries

### Configuration Model

-   `app.properties` must be placed alongside the executable\
-   Configuration is file-based and local to the deployment

### Enterprise Security Controls (Recommended)

-   Bind service to `127.0.0.1` by default\
-   For central reporting:
    -   Terminate TLS via reverse proxy\
    -   Restrict inbound traffic by corporate IP allowlists\
-   Enforce authentication (mTLS or token-based auth) for non-local
    traffic\
-   Keep memory-only storage unless persistence is explicitly required

------------------------------------------------------------------------

## Chrome Enterprise Management

### Extension Allowlisting

Admins can allowlist the extension via Google Admin Console.

**Extension ID:**\
`kokjfejdghjdinfjhhpbihljmcfpjaci`

### Policy Management Best Practice

Use `ExtensionSettings` for centralized lifecycle control: -
Installation mode\
- Policy enforcement\
- Permission governance

------------------------------------------------------------------------

## Network and Privacy Disclosure

### Network Behavior

-   Default: No outbound network calls\
-   Optional: When reporting is enabled:
    -   HTTP POST to listener endpoint\
    -   `/ping` used for health validation

### Data Captured (Reporting Only)

-   URL\
-   Timestamp\
-   Tab ID\
-   Request type\
-   Initiator

### Data Retention

-   In-memory only\
-   Non-persistent by default\
-   Persistence requires explicit implementation

------------------------------------------------------------------------

## Listener API Surface

### APIs Invoked by the Chrome Extension

These APIs are called only when reporting is enabled.

#### GET /ping

**Purpose:** Connectivity and health validation before enabling
reporting\
**Invoked by:** Chrome extension\
**Usage:** Server availability check

Response:

``` json
{
  "status": "ok",
  "message": "Server is running"
}
```

#### POST /

**Purpose:** Report blocked URL event\
**Invoked by:** Chrome extension\
**Usage:** Event delivery

Request Body:

``` json
{
  "blockedUrl": {
    "url": "https://example.com",
    "timestamp": "2026-01-01T10:00:00.000Z",
    "tabId": 123,
    "type": "main_frame",
    "initiator": "https://google.com"
  },
  "reportedAt": "2026-01-01T10:00:00.000Z"
}
```

Response:

``` json
{
  "message": "Blocked URL recorded successfully",
  "totalRequests": 5
}
```

------------------------------------------------------------------------

### APIs Exposed by the Listener Service

These APIs are not used by the plugin and are intended for
administrators, monitoring tools, or integrations.

#### GET /

**Purpose:** Retrieve all stored block events

Response:

``` json
{
  "requests": [ ... ],
  "totalRequests": 10
}
```

#### GET /?latest=true

**Purpose:** Retrieve the most recent block event

Response:

``` json
{
  "latest": { ... },
  "totalRequests": 10
}
```

#### DELETE /cleanup

**Purpose:** Clear in-memory event store

Response:

``` json
{
  "message": "All blocked URL requests cleared",
  "clearedCount": 10
}
```

#### GET /ping

**Purpose:** Health check endpoint\
**Consumers:** Extension, monitoring systems

Response:

``` json
{
  "status": "ok",
  "message": "Server is running"
}
```
