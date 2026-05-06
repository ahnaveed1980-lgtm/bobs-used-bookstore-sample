# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The solution build produced no errors across all five projects after transformation:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

This indicates the migration to cross-platform .NET was completed without introducing any compilation errors. The following steps outline how to validate, test, and deploy the solution.

---

## 1. Restore and Build the Solution

Run a full restore and build from the solution root to confirm a clean state:

```bash
dotnet restore
dotnet build --configuration Release
```

Ensure there are no warnings that could indicate deprecated APIs or compatibility issues that may surface at runtime.

---

## 2. Run the Unit Tests

Execute the test project to verify that existing business logic behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test output for:
- Any failing tests
- Any skipped tests that may have been conditionally excluded during migration
- Code coverage, if a coverage tool is configured

---

## 3. Verify Data Layer Functionality

Since `Bookstore.Data` handles persistence, confirm the following:

- **Database provider compatibility**: Ensure the configured database provider (e.g., SQL Server, SQLite, PostgreSQL) is supported by the target .NET version and that the correct NuGet packages are referenced.
- **Migrations**: If Entity Framework Core is used, verify existing migrations are intact and apply cleanly:

```bash
dotnet ef database update --project app/Bookstore.Data/Bookstore.Data.csproj --startup-project app/Bookstore.Web/Bookstore.Web.csproj
```

- **Connection strings**: Confirm that connection strings in `appsettings.json` or environment variables are correctly configured for the target environment.

---

## 4. Run and Smoke Test the Web Application

Start the web application locally and perform basic functional checks:

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Manually verify:
- Application starts without runtime exceptions
- Key pages and routes load correctly
- Data reads and writes function as expected
- Authentication and authorization flows work if applicable

Check the application logs for any runtime errors or deprecation warnings.

---

## 5. Validate the CDK Project

If `Bookstore.Cdk` defines infrastructure (e.g., AWS CDK), confirm the infrastructure definitions are still valid:

```bash
dotnet build app/Bookstore.Cdk/Bookstore.Cdk.csproj --configuration Release
```

Review any CDK constructs that may reference environment-specific values and ensure they are correctly parameterized for the target deployment environment.

---

## 6. Review Target Framework and Dependency Versions

Open each `.csproj` file and confirm:

- The `<TargetFramework>` element targets the intended .NET version (e.g., `net8.0`)
- All NuGet package versions are current and compatible with the target framework
- No packages reference `netstandard` or `net4x` only targets that could cause runtime issues

Use the following command to check for outdated packages:

```bash
dotnet list package --outdated
```

---

## 7. Check for Platform-Specific Code

Search the codebase for any APIs that may have been available on Windows but behave differently or are unavailable on Linux/macOS:

- `System.Drawing` (GDI+ is not fully supported cross-platform)
- Registry access (`Microsoft.Win32.Registry`)
- Windows-specific file path assumptions (backslashes, drive letters)

Replace any such usages with cross-platform alternatives where found.

---

## 8. Publish the Application

Once validation is complete, publish the application for the target runtime:

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

Verify the contents of the `./publish` directory and confirm all required assets, configuration files, and dependencies are present before deploying to the target environment.