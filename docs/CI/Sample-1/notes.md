# Enterprise Backend CI Pipeline (Azure DevOps / .NET)

This repository contains a production-grade **Continuous Integration (CI)** pipeline written in YAML for **Azure DevOps Server**.  
It automates the build and artifact workflow for large-scale .NET backend systems distributed across **more than 40 repositories**.  
For demonstration, this YAML file illustrates the configuration for **three representative repositories**.

---

## Overview

The pipeline provides an end-to-end automation process covering:

- Conditional repository checkout and workspace cleanup  
- Environment setup and authentication  
- NuGet and .NET package restoration  
- Solution build and artifact preparation  
- Branch-aware artifact publishing to shared junction paths  

---

## Pipeline Flow

### 1. Repository Setup
- The YAML defines multiple repositories under the `resources:` block:
  - `Logistics.Inventory`
  - `Logistics.Common`
  - `Logistics.Commercial`
- Each triggers on `master` and `Development` branches.
- The pipeline conditions ensure **only the active repository** (determined by `$(Build.Repository.Name)`) is checked out.

### 2. Authentication and Variables
- The variable `myorgBranchName` resolves from `$(Build.SourceBranchName)`.  
- Authentication to the DevOps server is established using:
  ```powershell
  cmdkey /add:devopsserver.myorg.net ...
  ```

### 3. Workspace Cleanup
- Removes old build and junction directories before restoring dependencies:
  ```powershell
  rmdir /q /s C:\\agent\\_work\\Web\\Junction
  ```
- Ensures a clean state for each build run.

### 4. Junction Creation
- The pipeline creates symbolic directory links to shared artifact paths, for example:
  ```
  C:\\agent\\_work\\Web\\Junction → C:\\myproductArtifacts\\Web\\<branch>
  ```
- These junctions centralize compiled binaries for use by related repositories.

### 5. Dependency Restoration
- Executes both NuGet and dotnet restore commands:
  ```powershell
  nuget restore <solution> -ConfigFile C:\\NuGet\\NuGet.Config
  dotnet restore <solution>
  ```

### 6. Build Execution
- Compiles the .NET solution via:
  ```powershell
  MSBuild <solution>
  ```
- Build conditions depend on the current branch (`master` or `Development`).

### 7. Artifact Copying
- After successful compilation, the output is synchronized to selected locations using `robocopy`:
  ```
  C:\\myproductArtifacts\\<Category>\\master
  C:\\myproductArtifacts\\<Category>\\Development
  ```
- This ensures consistent, versioned artifacts for downstream usage.

### 8. Final Cleanup
- Deletes temporary work files and directories to maintain agent stability.

---

## Key Benefits

- Unified YAML CI design for **40+ repositories**  
- Independent, isolated build environments per repository  
- Efficient artifact sharing across backend services  
- Reliable branch-based artifact separation  
- Simplified maintenance and scalability; new repositories can be added easily to the `resources:` section  

---

## File Reference

| File | Description |
|------|--------------|
| `azure-pipelines.yml` | The complete CI pipeline configuration for repository processing, build, and artifact distribution. |

---

## Technical Keywords

Azure DevOps, CI/CD, .NET Core, MSBuild, NuGet Restore, dotnet restore,  
PowerShell automation, conditional checkout, multi-repository pipeline,  
artifact management, build synchronization, enterprise DevOps practices

---

**License:** Internal Use Only  
**Maintained by:** DevOps Engineering Team
