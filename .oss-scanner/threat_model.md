# Threat model

## What this project does and where untrusted input enters
An uptime-monitoring server (Node.js/TypeScript, Koa + Vue). Untrusted input: the HTTP API
(adding/editing monitors, push messages, agent data), URLs to be fetched, and uploaded
authentication/basic-auth credentials. The server issues outbound HTTP requests to user-supplied
URLs (SSRF surface).

## Components that matter most / least
- most: the HTTP check engine (outbound requests to user URLs — SSRF), the API/auth layer, and
  the push/agent message handling.
- less: the Vue frontend.

## How to exercise it
Register monitors pointing at internal/metadata URLs (SSRF); send crafted push/agent payloads;
exercise the API with malformed input.

## How you rate severity
- RCE or auth bypass: critical. SSRF reaching internal services: high.
- Stored XSS via monitor names/notifications: medium-high.
- DoS: medium.

## Anything to leave alone
- Do not report the frontend bundling artifacts; focus on the server.
