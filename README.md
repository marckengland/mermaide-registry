# Mermaide package registry

This repository stores publisher-signed `.mpkg` submissions and the trusted inputs used to publish the static feed at `https://mermaide.marckengland.com/registry/v1/`.

The generated `registry-tool.mjs`, its notices, and `schemas/v1/` come from the reviewed Mermaide source. No npm install or submitted package code runs in registry CI. Protect these files, workflows, `registry.json`, `trust/`, and publisher enrollment with required maintainer review before accepting public submissions.

## Submit a version

1. Ask the maintainer to register your namespace and Ed25519 public key. Registration in `trust/publishers.json` is a separate maintainer action. A new self-proposed `publishers/<namespace>.json` does not authorize its own key.
2. Use `mermaide publish` with `--registry marckengland/mermaide-registry`. The CLI opens a package proposal; it does not approve or list the version.
3. Address scan findings. The PR workflow compares immutable base and head trees and downloads only added submission data using trusted base-branch tooling. Changes to code, workflows, existing versions or trust inputs fail admission.
4. After maintainer review and merge, the protected release workflow rescans the bytes, adds the marketplace signature, advances signed metadata and requests a Pages build. A green scan alone is not a release. A previously verified immutable version that is being revoked, or is already revoked, can remain in signed history despite newly blocked scan findings; this never admits a new unsafe version.

## Verify a domain

An enrolled publisher can ask the maintainer to add its domain to its protected authorization. Publish this TXT record, replacing the namespace and public key ID:

```text
_mermaide-publisher.publisher.example TXT "mermaide-publisher=marckengland/mermaide-registry:namespace:<public-key-sha256>"
```

The record binds the exact registry, namespace and key. DNS failures or mismatches leave the publisher unverified. A signed proof expires after 24 hours. Verification establishes control of the domain, not that the package is safe.

## Reports and revocations

Use the package report issue form for non-sensitive reports. Do not post credentials or working exploits. Enable GitHub private vulnerability reporting and use it for security disclosures.

Maintainers add exact publisher/name/version entries and a public reason to `trust/revocations.json`. The next protected release retains prior revocations, signs the complete list and excludes revoked versions from active results. Consumers use this list to disable installed versions on their next update check. Removal of an entry from the source request file cannot erase a published revocation.

## Release and recover

The `registry-release` environment must restrict deployment to `main` and contain `MARKETPLACE_SIGNING_KEY`. Require CODEOWNER review for changes to source, workflows and trust inputs before they enter `main`. After launch approval, scheduled refreshes run without a per-run environment review; otherwise an unattended feed expires while a job waits for approval. Its public key must match `registry.json` and the public key shipped in Mermaide. Never store the private key in this repository. Namespace and marketplace metadata accept only canonical public SPKI PEM values, with no private PEMs or appended material. Keep `gh-pages` protected against force pushes and other actors replacing its history. Approve the PR/push/six-hour schedule policy before publishing these workflows.

The release workflow verifies the previous deployment from the exact `gh-pages` commit, preserves immutable archives and refuses a changed branch before publication. It requests Pages explicitly because writes using `GITHUB_TOKEN` do not trigger a Pages build automatically. Configure Pages for the `gh-pages` branch, root directory, the exact custom domain and enforced HTTPS. The initial publication can prepare the branch before Pages is configured; that workflow reports the remaining setup and does not claim the site is live.

To recover from a bad deployment, prepare a new signed snapshot using the last valid deployment as `--previous`. Advance the sequence and retain every revocation. Verify the new site before publication. Do not force-push an older snapshot: consumers that have accepted a higher sequence reject it. A public archive cannot recall copies that users already downloaded.

The static registry records unmeasured install counts as `null`. It does not infer downloads from Git or mark them as zero. The later marketplace service owns measured statistics. Rule inspection is bounded and heuristic; it blocks recognized risks and uninspectable inputs but cannot prove arbitrary programs benign. All package execution still requires runtime capability enforcement and user review.
