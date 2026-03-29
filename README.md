# shared-build-components (GitHub Actions)

This repository contains shared actions and reusable workflows which are used across my repositories to standardize and simplify build and deploy pipelines.

Shared actions in this repository: 
- Compile SCSS assets in place: [scss-compile](.github/actions/scss-compile/action.yml)
- Restore, build, test, and publish .NET projects: [dotnet-build](.github/actions/dotnet-build/action.yml)
- Deploy published .NET web apps via SSH with optional health checks and rollback: [dotnet-deploy-web](.github/actions/dotnet-deploy-web/action.yml)

Reusable workflows in this repository:
- Build .NET web applications: [dotnet_web_build](.github/workflows/dotnet_web_build.yml)
- Build and deploy .NET web applications via SSH: [dotnet_web_build_deploy_ssh](.github/workflows/dotnet_web_build_deploy_ssh.yml)
- Build Docker images, with optional registry push and deployment compose artifact generation: [docker_build](.github/workflows/docker_build.yml)
- Build and test Docker images for .NET applications, with optional registry push: [docker_dotnet_build](.github/workflows/docker_dotnet_build.yml)
- Build, test, and publish multi-platform Docker images for .NET applications: [docker_dotnet_build_multi_platform](.github/workflows/docker_dotnet_build_multi_platform.yml)
- Deploy a Docker Compose file from repository source via SSH: [docker_compose_deploy_from_source](.github/workflows/docker_compose_deploy_from_source.yml)
- Deploy a Docker Compose artifact via SSH: [docker_compose_deploy_from_artifacts](.github/workflows/docker_compose_deploy_from_artifacts.yml)

# Example usage

## Using shared actions

```yaml
    - name: Build dotnet projects
      uses: hcspieker/shared-build-components/.github/actions/dotnet-build@v3.0.0
      with:
        build-filter: 'MySolution/MySolution.sln'
```
## Using reusable workflows

```yaml
jobs:
  call-dotnet_web_build:
    uses: hcspieker/shared-build-components/.github/workflows/dotnet_web_build.yml@v3.0.0
    with:
      enable-scss-compile: true
      css-directory: 'MySolution/MyWebProject/wwwroot/css'
      build-filter: 'MySolution/MyWebProject/MyWebProject.csproj'
      enable-tests: 'false'
```

## Versioning and stability

- For production workflows pin to a tag or commit SHA, e.g. `uses: hcspieker/shared-build-components/.github/actions/dotnet-build@v3.0.0` or `@<sha>`.
- Using `@main` is convenient for development but can introduce breaking changes to consumers.

