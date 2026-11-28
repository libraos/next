# Product usage data

By default, LibraOS Next sends no product usage events, crash reports or connection
traces to its makers or to upstream. With no configuration, the server sends nothing to
PostHog and fetches nothing from upstream's servers.

The app still uses the network. Some requests start on their own: it checks which
providers are installed and signed in, and checks for provider updates unless you turn
that setting off. Listing, installing or removing Skills can fetch `catalog.json` from `libraos/skills` at `raw.githubusercontent.com`, normally no more than once every 24 hours for each server. Installing a Skill also fetches its source repository's file listing from `api.github.com` and its files from `raw.githubusercontent.com`. GitHub receives the server's outgoing IP address, the request time and the requested paths, along with HTTP headers including a User-Agent and tracing identifiers. An administrator can block these requests with the [managed policy](./managed-policy.md) limit `skillCatalog: false`. The rest follow from what you do: your
providers' CLIs talking to their vendors, source control, provider installs, speech
model downloads, and similar actions.

## Analytics you configure

The server sends product usage events only when both `T3CODE_POSTHOG_KEY` and
`T3CODE_POSTHOG_HOST` are set in its environment. It then sends them to that host;
whoever runs the host receives them. Events include the provider, model, reasoning
effort, permission mode, turn and request results, duration, release channel, and
token counts when available, associated with a hashed random installation identifier and, when available, hashed
client-device or hosted-relay account identifiers, never identifiers derived from your
Claude or Codex sign-in. Routine usage events do not include prompts, responses, file contents, authentication
tokens, conversation IDs, raw provider events, or child-agent output. With analytics configured, a feedback report you approve can include
scrubbed conversation text if you choose to include it.

To keep events off even when those variables are set, set
`T3CODE_TELEMETRY_ENABLED=false` in the server's environment before starting it. An
administrator can turn them off for the server and desktop app with a
[managed policy](./managed-policy.md#other-capabilities).

This setting covers product usage events only. Diagnostics you export to an
OpenTelemetry receiver (below) are separate: remove the endpoints, or set
`T3CODE_OTEL_SDK_DISABLED=true` and restart, to stop those.

The desktop app reads `T3CODE_TELEMETRY_ENABLED` from your shell profile (for
example `~/.zshrc`) on macOS and Linux, so export it there and restart the app. On
Windows, set it as a user environment variable.

## Hosted relay and mobile diagnostics

Upstream's hosted relay, its relay connection traces, mobile crash reports and agent
activity for mobile notifications are not part of LibraOS Next: the releases have no
relay configured, so none of these are sent.

The LibraOS Next phone apps send nothing by default: no analytics, crash reports or
traces. They talk only to the computers you pair them with. Notifications arrive while
the app is open.

## Export diagnostics to your own receiver

To send traces, metrics, or logs to an OpenTelemetry receiver, open
**Settings > Maintenance > Diagnostics**, select an environment, and enter each signal's OTLP HTTP
endpoint under **OpenTelemetry export** (for example `http://localhost:4318/v1/traces`). Save, then
restart that environment's server. The receiver must be reachable from the server's machine.
**Test** sends an empty export from the server to check an endpoint before or after you save it;
saved endpoints are checked when the section opens. Checks send authentication headers only to
the signal's currently running endpoint and do not follow redirects. To check a new endpoint
that requires authentication, save it and restart the server first.

`T3CODE_OTLP_*_URL` and standard `OTEL_EXPORTER_OTLP_*` environment variables override the saved
settings. An overridden signal shows **Set by environment** and names the variable that set it, and its
**Test** checks the running endpoint instead of the field. For endpoints saved here or set with `T3CODE_OTLP_*_URL`, set `T3CODE_OTLP_HEADERS` for
receivers that need authentication and `T3CODE_OTLP_PROTOCOL=http/protobuf` for receivers that do
not accept JSON. An endpoint set with `OTEL_EXPORTER_OTLP_*` takes its headers and protocol from
`OTEL_EXPORTER_OTLP_HEADERS` and `OTEL_EXPORTER_OTLP_PROTOCOL` instead.
