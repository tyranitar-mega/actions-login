# actions-login

A GitHub Action that bridges a job's OIDC token to the [ScopeDB CLI](https://github.com/scopedb/scopedb-cli): it requests a token from GitHub, then exports `SCOPEDB_OIDC_TOKEN` and `SCOPEDB_WORKSPACE_ID` so data commands (`scope query`, `scope catalog`, `scope doctor`) authenticate without a stored login.

## Usage

```yaml
permissions:
  contents: read
  id-token: write

steps:
  - uses: scopedb/actions-login@v1
    with:
      workspace-id: "0123456789ab"
  - run: scope query 'SELECT 1 AS ready'
```

## Inputs

| Input          | Required | Default                         | Description                                            |
| -------------- | -------- | ------------------------------- | ------------------------------------------------------ |
| `workspace-id` | yes      |                                 | ScopeDB workspace ID data commands authenticate against |
| `audience`     | no       | `https://control.scopedb.cloud` | OIDC token audience: the deployment's control-plane origin |

## Requirements

- The job or workflow must grant `permissions: id-token: write`; without it GitHub does not expose a token endpoint and the step fails.
- A workspace owner must create an OpenID Connect trust policy in the ScopeDB Console whose subject matches the token's `sub` claim, for example `repo:org/repo:environment:production`. Jobs whose subject matches no policy are rejected at the exchange.

## Notes

- The token is masked before it is exported and is never written to disk by the action or the CLI.
- Each `scope` invocation exchanges the token for a short-lived workspace credential; nothing is persisted between steps.
- Outside GitHub Actions, export `SCOPEDB_OIDC_TOKEN` (or `SCOPEDB_OIDC_TOKEN_FILE`) and `SCOPEDB_WORKSPACE_ID` directly; this action only bridges GitHub's token endpoint.
