# Security audit triage, 2026-10-21

## Confirmed findings and remediation issues

The parent audit issue reports **0 confirmed findings**. There are no confirmed findings to prioritize and no evidence-backed remediation issues to file. The eight items below are execution- or deployment-dependent leads, not confirmed vulnerabilities. Do not create remediation issues for them until their stated missing facts are validated.

## Needs-validation issue draft

**Title**

`[security][needs-validation] Audit leads to validate`

**Body**

Parent issue: [Security audit 2026-10-21 — prioritize findings and create remediation issues](https://github.com/Mike-Demo/jobsearch-apply-mcp/issues/1)

The 2026-10-21 audit reported zero confirmed findings and eight needs-validation leads. Resolve each checkbox by establishing the exact missing fact below. These leads have no severity assigned; do not treat them as confirmed vulnerabilities.

- [ ] **Company-research cache filename normalization allows path separators and dot segments from an untrusted company name**
  - Fingerprint: `company-research-cache-path-traversal`
  - Claimed root cause: Cache filename normalization in the job-evaluation guidance only lowercases, trims, and replaces spaces with hyphens, without dropping or rejecting '/' and '.' characters, unlike the stricter single-component rule in documents/README.md.
  - Missing fact: Whether the executing agent runtime additionally sanitizes or rejects '/' and '..' in the normalized company name when resolving company_research/<normalized-company-name>.json — no such rule appears in the workflow files and execution was not performed.

- [ ] **Unauthenticated HTTP mode is decided by bind address, and the shipped guidance sanctions reaching it through a reverse proxy**
  - Fingerprint: `localhost-none-auth-proxy-exposure`
  - Claimed root cause: auth.py decides unauthenticated mode solely from the MCP_HOST bind address, and the refusal message suggests reaching a localhost bind through an SSH tunnel or reverse proxy, without requiring the proxy itself to authenticate callers.
  - Missing fact: Whether a given user's reverse proxy/tunnel in front of a localhost-bound MCP_AUTH=none instance itself authenticates callers — a deployment fact not determinable from this repo.

- [ ] **Entra OAuth disables the authorization consent screen while client registration stays open and redirect extras are unvalidated**
  - Fingerprint: `oauth-consent-disabled-open-dcr`
  - Claimed root cause: auth.py configures the provider with require_authorization_consent set to False and appends MCP_ALLOWED_REDIRECT_URIS extras verbatim with no validation, relying on the redirect allowlist and email allowlist instead of an owner-visible consent step.
  - Missing fact: Provider-side facts not in this repo: how FastMCP/AzureProvider matches the 'http://localhost:*' wildcard redirect patterns, and whether an attacker-initiated authorization flow can complete without owner interaction/credentials for an allowlisted identity — no live endpoint was probed.

- [ ] **Untrusted posting text shares a session and tool set with ungated persistent profile writes (memory poisoning)**
  - Fingerprint: `prompt-injection-profile-poisoning-no-write-gate`
  - Claimed root cause: add_profile_fact checks only non-empty text and update_profile_section checks only a minimum length, with no code-level confirmation, provenance, or content gate, while untrusted posting text and profile writes share one session and tool set.
  - Missing fact: Whether a target MCP client/model actually follows an instruction embedded in job_detail/search_jobs/ingest_jobs posting text to call add_profile_fact or update_profile_section — requires model execution, prohibited in this run.

- [ ] **Workflow Bash templates interpolate untrusted company names and URLs inside shell quotes with no escaping rule**
  - Fingerprint: `shell-interpolation-of-untrusted-company-and-url`
  - Claimed root cause: The salary lookup, robots check, and curl retry templates interpolate untrusted company names and URLs inside shell quotes without any escaping, sanitization, or argument-vector invocation rule in source.
  - Missing fact: Whether the executing model/agent runtime escapes or rejects quote/shell-metacharacter values when interpolating company names and URLs into these Bash templates in practice — not in source and not executed in this run.

- [ ] **Workflows fetch stored third-party URLs from the user's machine with no scheme, host or private-address restriction**
  - Fingerprint: `stored-posting-url-ssrf-no-destination-allowlist`
  - Claimed root cause: The ingest and store path validates only that a URL is non-empty after stripping the query string, and the fetch workflows impose no destination allowlist or private-address block, including after redirect resolution; the robots gate governs permission to retry, not destination safety.
  - Missing fact: The target runtime's SSRF protections for WebFetch and for the curl fallback — specifically whether either blocks loopback, link-local (169.254.0.0/16, ::1, fe80::/10) and private-range destinations after redirect resolution.

- [ ] **A custom template's manifest carries a persistent arbitrary compile command that later /apply runs execute**
  - Fingerprint: `template-manifest-persistent-compile-command`
  - Claimed root cause: The template activation path extracts and persists the full declared compile command from TEMPLATE.md without restricting it to known toolchains, and the apply workflow executes that persisted command.
  - Missing fact: Whether the agent runtime prompts for / displays the exact declared compile command at --use activation or at later /apply execution — .claude/settings.json does not pre-approve tectonic/typst/arbitrary commands, so a prompt may occur in Claude Code but is not guaranteed across the other runtimes the repo supports.

- [ ] **search_jobs and list_jobs return third-party job fields without the untrusted-content labelling applied to job_detail**
  - Fingerprint: `unlabelled-untrusted-job-fields`
  - Claimed root cause: Only job_detail applies the untrusted-content wrapper; search_jobs result rows carry no warning key and list_jobs carries neither a docstring warning nor a payload warning for stored third-party fields.
  - Missing fact: Whether a target MCP client/model treats an instruction embedded in an unlabelled title/company field — in particular via list_jobs, which carries no docstring or payload warning — as an instruction and acts on it; requires model execution, prohibited in this run.

Rejected during independent verification (do not re-raise without new evidence): `unbounded-ingest-and-unpruned-profile-backups`.
