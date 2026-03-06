# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Overview

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework.

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues beyond what was reported.

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas of the code that may behave differently under cross-platform .NET.

---

## 3. Run the Unit Tests

Execute the test project to verify that existing business logic behaves as expected after the migration.

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

- Review any failing tests carefully. A test failure after migration may indicate a behavioral difference between .NET Framework and cross-platform .NET.
- Pay attention to tests that involve file I/O, culture-sensitive operations, or reflection, as these areas are common sources of cross-platform differences.

---

## 4. Validate the Data Layer

Since `Bookstore.Data` handles data access, verify the following:

- **Database Migrations**: If Entity Framework Core is in use, confirm that all migrations are up to date.
  ```bash
  dotnet ef migrations list --project app/Bookstore.Data
  ```
- **Connection Strings**: Ensure connection strings in configuration files (`appsettings.json`) are correct for the target environment.
- **Provider Compatibility**: Confirm that the database provider package (e.g., `Microsoft.EntityFrameworkCore.SqlServer` or `Npgsql`) targets a version compatible with the new framework.

---

## 5. Run and Validate the Web Application

Start the web application locally and perform manual validation of core functionality.

```bash
dotnet run --project app/Bookstore.Web/Bookstore.Web.csproj --configuration Release
```

Check the following areas:

- Application startup completes without exceptions.
- Key routes and pages load correctly.
- Authentication and authorization flows work as expected.
- Static files (CSS, JavaScript, images) are served correctly.
- Any middleware configured in `Program.cs` or `Startup.cs` behaves as intended.

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project is likely used for infrastructure definition. Verify the following:

- All AWS CDK construct library packages are compatible with the current .NET version.
- The CDK project synthesizes correctly by running:
  ```bash
  dotnet build app/Bookstore.Cdk/Bookstore.Cdk.csproj --configuration Release
  ```
- Review any infrastructure configuration that references environment-specific values to ensure they are accurate for the target deployment environment.

---

## 7. Check for Platform-Specific Code

Search the solution for any APIs that are not supported on cross-platform .NET. Common areas to inspect include:

- `System.Web` namespace usage (not available in cross-platform .NET).
- Windows Registry access (`Microsoft.Win32.Registry`).
- Windows-specific authentication mechanisms.
- `AppDomain` usage beyond what is supported in .NET Core and later.

Use the .NET Upgrade Assistant compatibility analyzer or the `Microsoft.DotNet.PlatformAbstractions` tooling to assist with this review if needed.

---

## 8. Deploy the Application

Once all validation steps pass, deploy the application to the target environment.

```bash
dotnet publish app/Bookstore.Web/Bookstore.Web.csproj --configuration Release --output ./publish
```

- Confirm the published output contains all expected files.
- Verify that the runtime identifier (RID) is set correctly in the `.csproj` if a self-contained deployment is required.
- Ensure the target server has the correct .NET runtime installed if deploying a framework-dependent application.