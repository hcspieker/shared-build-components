# shared-build-components

This repository contains shared build components which are used across my repositories to standardize and simplify build and deploy pipelines.

This repository contains the following actions:
- `dotnet-build` — build, test, publish .NET projects.
- `dotnet-deploy-web` — deploy published web artifacts to a remote host via SSH.
- `scss-compile` — compile SCSS files in-place to CSS and minified CSS.

Each action lives under `.github/actions/<action-name>` and is intended to be referenced from consumer workflows (pin to a tag/sha for production stability).

## Action: `.github/actions/dotnet-build`
Description: Builds, tests and optionally publishes .NET projects and uploads the published output as an artifact.

Inputs:
- `dotnet-version` (optional, default: `9.0.x`) — .NET SDK version to install via `actions/setup-dotnet`.
- `build-configuration` (optional, default: `Release`) — build configuration passed to `dotnet build`/`dotnet test`/`dotnet publish`.
- `build-filter` (optional, default: `**/*.csproj`) — path or glob filter used for `dotnet restore` and `dotnet build`.
- `test-filter` (optional, default: `**/*[Tt]ests/*.csproj`) — path or glob filter used for `dotnet test`.
- `publish-filter` (optional, default: `**/*.csproj`) — path or glob filter used for `dotnet publish`.
- `enable-tests` (optional, default: `true`) — if `'true'`, runs `dotnet test`.
- `enable-publish` (optional, default: `false`) — if `'true'`, runs `dotnet publish` and uploads the `./publish` directory.
- `artifact-name` (optional, default: `dotNetApp`) — name used with `actions/upload-artifact` when `enable-publish` is `'true'`.

Outputs / side-effects:
- This action does not declare GitHub Action `outputs` in `action.yml`.
- If `enable-publish` is `'true'`, it produces published files under `./publish` and uploads them with `actions/upload-artifact` using the `artifact-name`. Consumers can use `actions/download-artifact` with the same `artifact-name`.

Example usage:
```yaml
  - name: Dotnet build
    uses: hcspieker/shared-build-components/.github/actions/dotnet-build@main
    with:
      dotnet-version: '9.0.x'
      build-configuration: 'Release'
      enable-tests: 'true'
      enable-publish: 'true'
      artifact-name: 'my-app-artifact'
```

## Action: `.github/actions/dotnet-deploy-web`
Description: Downloads a published artifact (from `dotnet-build`) and deploys it to a remote host via `scp`/`ssh`. Performs an in-place "next -> current -> old" switch and can run a health check with automatic rollback.

Inputs:
- `artifact-name` (optional, default: `dotNetApp`) — name used to `download-artifact`.
- `target-directory` (required) — remote path where app versions are stored; the action uploads into `<target-directory>/next` and rotates directories to `current`.
- `ssh-host` (required) — SSH host or IP.
- `ssh-username` (required) — SSH username.
- `ssh-key` (required) — SSH private key (PEM) content; used by `appleboy/scp-action` and `appleboy/ssh-action`.
- `kestrel-service` (required) — systemd service name to restart (used for Kestrel-hosted apps).
- `internal-health-endpoint` (optional, default: `''`) — internal URL to perform health checks after deployment. Leave empty to skip health checks.
- `health-check-retry-delay` (optional, default: `10`) — retry delay in seconds between health-check attempts.
- `health-check-retry-amount` (optional, default: `6`) — number of health-check retry attempts.

Outputs / side-effects:
- No declared `outputs` in `action.yml`.
- Side effects: downloads the artifact named `artifact-name` to the job workspace, copies files to `<target-directory>/next` on remote host, atomically rotates directories to `current`, corrects permissions/ownership, restarts the provided `kestrel-service`, and optionally performs health checks. On failing health checks (after retries) the action attempts an automatic rollback to the previous `old` version and exits with failure.

Example usage:
```yaml
 - name: Deploy web artifact
    uses: hcspieker/shared-build-components/.github/actions/dotnet-deploy-web@main
    with:
      artifact-name: 'my-app-artifact'
      target-directory: '/opt/myapp'
      ssh-host: ${{ secrets.DEPLOY_HOST }}
      ssh-username: ${{ secrets.DEPLOY_USER }}
      ssh-key: ${{ secrets.DEPLOY_SSH_KEY }}
      kestrel-service: 'myapp.service'
      internal-health-endpoint: 'http://127.0.0.1:5000/health'
      health-check-retry-delay: '10'
      health-check-retry-amount: '6'
```


Security note: supply `ssh-key` via repository or org `Secrets` and avoid printing secrets into logs.

## Action: `.github/actions/scss-compile`
Description: Finds `.scss` files in the given directory and compiles them to `.css` and minified `.min.css` using the Sass CLI. Skips partial files whose names start with `_`.

Inputs:
- `node-version` (optional, default: `20.x`) — Node.js version installed via `actions/setup-node`.
- `css-directory` (optional, default: `wwwroot/css`) — directory where `.scss` files are located and where compiled `.css` files will be written.

Outputs / side-effects:
- No declared `outputs` in `action.yml`.
- Side effects: Compiled `.css` and `.min.css` files are created in the same `css-directory`. Partial files (leading underscore) are skipped.

Example usage:
```yaml
  - name: Compile SCSS
    uses: hcspieker/shared-build-components/.github/actions/scss-compile@main
    with:
      node-version: '20.x'
      css-directory: 'src/wwwroot/css'
```

## Versioning and stability
- For production workflows pin to a tag or commit SHA, e.g. `uses: hcspieker/shared-build-components/.github/actions/dotnet-build@v1.0.0` or `@<sha>`.
- Using `@main` is convenient for development but can introduce breaking changes to consumers.
