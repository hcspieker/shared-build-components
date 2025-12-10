# shared-build-components (GitHub Actions)

This repository contains shared actions and reusable workflows which are used across my repositories to standardize and simplify build and deploy pipelines.

Shared actions in this repository: 
- SCSS compilation: [scss-compile](.github/actions/scss-compile/action.yml)
- .NET build, test, publish: [dotnet-build](.github/actions/dotnet-build/action.yml)
- .NET web app deployment via SSH: [dotnet-deploy-web](.github/actions/dotnet-deploy-web/action.yml)

Reusable workflows in this repository:
- Build .NET Web Applications: [dotnet_web_build](.github/workflows/dotnet_web_build.yml)
- Build and Deploy .NET Web Applications via SSH: [dotnet_web_build_deploy_ssh](.github/workflows/dotnet_web_build_deploy_ssh.yml)

# Example usage

## Using shared actions

```yaml
    - name: Build dotnet projects
      uses: hcspieker/shared-build-components/.github/actions/dotnet-build@v2.0.0
      with:
        build-filter: 'MySolution/MySolution.sln'
```
## Using reusable workflows

```yaml
jobs:
  call-dotnet_web_build:
    uses: hcspieker/shared-build-components/.github/workflows/dotnet_web_build.yml@v2.0.0
    with:
      enable-scss-compile: true
      css-directory: 'MySolution/MyWebProject/wwwroot/css'
      build-filter: 'MySolution/MyWebProject/MyWebProject.csproj'
      enable-tests: 'false'
```

## Versioning and stability

- For production workflows pin to a tag or commit SHA, e.g. `uses: hcspieker/shared-build-components/.github/actions/dotnet-build@v1.0.0` or `@<sha>`.
- Using `@main` is convenient for development but can introduce breaking changes to consumers.

