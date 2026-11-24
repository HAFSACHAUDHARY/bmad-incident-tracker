- source_spec: `_bmad-output/implementation-artifacts/spec-1-2-github-actions-ci-and-pages-deploy.md`
  summary: PR-vs-main Pages deploy guards lack an executable act/live Actions check
  evidence: Workflow `if:` strings only; closing needs `act` or observing a real PR run does not upload/deploy; story treats live PR-deploy safety as out-of-band
