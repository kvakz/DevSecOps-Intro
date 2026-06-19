# Lab 3 — Submission

## Task 1: SSH Commit Signing

### Local configuration
- `git config --global gpg.format` → ssh
- `git config --global user.signingkey` → /home/uld16/.ssh/id_ed25519.pub
- `git config --global commit.gpgsign` → true

### Local verification
Output of `git log --show-signature -1`:
```
commit 77d8df8f756f3ef06261dac7cbc90aa334e76628
Good "git" signature for uld163@icloud.com with ED25519 key SHA256:2WLOjA/IllIpMrO7nH1fTGRcPAhb9dxdZAKaW/evEYY
Author: kvakz <uld163@icloud.com>
Date:   Fri Jun 19 14:03:28 2026 +0300

    test: first signed commit
```

### GitHub verification
- Direct link to your most recent commit on GitHub: https://github.com/inno-devops-labs/DevSecOps-Intro/commit/77d8df8f756f3ef06261dac7cbc90aa334e76628
- Screenshot of the Verified badge: 

### One-paragraph reflection (2-3 sentences)
A forged-author commit allows an attacker to commit malicious code while impersonating a trusted contributor, enabling them to repudiate their actions or frame others. The Verified badge prevents this by cryptographically linking the commit to a trusted SSH key, making any unsigned or improperly signed impersonation attempt immediately visible as "Unverified" on GitHub.

---

## Task 2: Pre-commit + gitleaks

### `.pre-commit-config.yaml` (paste the full content)
```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.21.0
    hooks:
      - id: gitleaks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.6.0
    hooks:
      - id: detect-private-key
      - id: check-added-large-files
```

### `pre-commit install` output
```
pre-commit installed at .git/hooks/pre-commit
```

### The blocked commit
Output of the `git commit` that gitleaks blocked (the failing hook output):
```
Detect hardcoded secrets.................................................Failed
- hook id: gitleaks
- exit code: 1

Finding:     GH_PAT= REDACTED
Secret:       REDACTED
RuleID:      github-pat
Entropy:     4.143943
File:        submissions/leak-attempt.txt
Line:        2
Fingerprint: submissions/leak-attempt.txt:github-pat:2

INF 1 commits scanned.
INF scan completed in 60.9ms
WRN leaks found: 1
```

### Tune-out exercise
1. **Inline allowlist** — `[allowlist]` block in `.gitleaks.toml`. This is appropriate when specific, known-safe strings (e.g., canonical example secrets in documentation) need to be ignored regardless of their location in the repo.
2. **Path exclusion** — `paths: [docs/]` in `.gitleaks.toml`. This is risky because it creates a blind spot; any real secret accidentally committed to the excluded directory will be ignored by the scanner.

---

## Bonus: History Rewrite

### Before
```
d5fdf77 docs: add usage notes
6193702 feat: empty log
d30c082 feat: add config
ffcdd4e init
```
Output of `git log -p | grep -c 'ghp_'`: **2**

### After
```
4c8a1fd docs: add usage notes
326e75a feat: empty log
008ca6d feat: add config
b15f651 init
```
Output of `git log -p | grep -c 'ghp_'`: **0**
Output of `git log -p | grep -c 'REDACTED'`: **2**

### The two-step pattern in real life
1. `git filter-repo --replace-text replacements.txt` — rewrite locally
2. **Secret Rotation** — In a real incident, the leaked secret must be revoked and rotated immediately. History rewriting only removes the evidence from the repo, it doesn't invalidate the secret if it was already compromised.

### Two real-world gotchas you discovered (2 sentences each)
1. `git-filter-repo` refuses to run on any repository that isn't a fresh clone by default to prevent accidental data loss. I had to use the `--force` flag to bypass this check in the sandbox repo.
2. History rewriting modifies every subsequent commit hash from the point of the first change. In a real team environment, this would require a coordinated force-push and would force all other developers to rebase their work.
