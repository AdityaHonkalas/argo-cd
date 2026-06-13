# CI Error Testing Workflows

This document describes the controlled CI error workflows created for testing failure scenarios in GitHub Actions.

## Overview

These workflows are designed to intentionally fail in specific ways to test CI/CD error handling, monitoring, and alerting systems. Each workflow is triggered manually via `workflow_dispatch` to prevent accidental execution.

## Workflow Naming Convention

All error testing workflows follow this naming pattern:
```
<base-workflow-name>-<error-type>-error.<yaml|yml>
```

## Error Types

### 1. Authentication Errors (`auth-error`)
Simulates authentication failures with invalid credentials or tokens.

### 2. Package Version Errors (`package-error`)
Simulates dependency resolution failures with invalid package versions.

### 3. Syntax Errors (`syntax-error`)
Simulates YAML parsing errors with malformed workflow syntax.

### 4. Timeout Errors (`timeout-error`)
Simulates job timeouts with unrealistically short timeout settings.

---

## Workflow Details

### CI Build Workflows

#### 1. `ci-build-auth-error.yaml`
- **Base Workflow**: `ci-build.yaml`
- **Error Type**: Authentication Error
- **Error Location**: `test-go` job
- **Error Details**: Invalid `GITHUB_TOKEN` set to `ghp_INVALID_TOKEN_FOR_TESTING_AUTH_ERROR_1234567890`
- **Expected Failure**: Authentication failure when accessing GitHub API or private repositories
- **Trigger**: Manual (`workflow_dispatch`)

#### 2. `ci-build-package-error.yaml`
- **Base Workflow**: `ci-build.yaml`
- **Error Type**: Package Version Error
- **Error Location**: `build-go` job
- **Error Details**: Invalid `GOLANG_VERSION` set to `99.99.99`
- **Expected Failure**: `setup-go` action fails to find the specified Go version
- **Trigger**: Manual (`workflow_dispatch`)

#### 3. `ci-build-syntax-error.yaml`
- **Base Workflow**: `ci-build.yaml`
- **Error Type**: Syntax Error
- **Error Location**: `build-go` job, step "Restore go build and module cache"
- **Error Details**: Missing colon after `name` field (line 64)
- **Expected Failure**: YAML parsing error when workflow is loaded
- **Trigger**: Manual (`workflow_dispatch`)

#### 4. `ci-build-timeout-error.yaml`
- **Base Workflow**: `ci-build.yaml`
- **Error Type**: Timeout Error
- **Error Location**: `test-go` job
- **Error Details**: Job timeout set to 1 minute (normally takes 10+ minutes)
- **Expected Failure**: Job cancelled due to timeout
- **Trigger**: Manual (`workflow_dispatch`)

---

### Image Building Workflows

#### 5. `image-auth-error.yaml`
- **Base Workflow**: `image.yaml`
- **Error Type**: Authentication Error
- **Error Location**: `build-and-publish` job secrets
- **Error Details**: Invalid Quay.io credentials (`invalid_username_for_testing`, `invalid_password_for_testing_auth_error_12345`)
- **Expected Failure**: Docker login to Quay.io fails
- **Trigger**: Manual (`workflow_dispatch`)

#### 6. `image-package-error.yaml`
- **Base Workflow**: `image.yaml`
- **Error Type**: Package Version Error
- **Error Location**: `build-only` job workflow call
- **Error Details**: Invalid `go-version` set to `99.99.99`
- **Expected Failure**: Reusable workflow fails during Go setup
- **Trigger**: Manual (`workflow_dispatch`)

#### 7. `image-syntax-error.yaml`
- **Base Workflow**: `image.yaml`
- **Error Type**: Syntax Error
- **Error Location**: `build-only` job
- **Error Details**: Invalid job dependency reference `set-var` (should be `set-vars`)
- **Expected Failure**: Workflow validation error for missing job dependency
- **Trigger**: Manual (`workflow_dispatch`)

#### 8. `image-timeout-error.yaml`
- **Base Workflow**: `image.yaml`
- **Error Type**: Timeout Error
- **Error Location**: `build-only` job
- **Error Details**: Job timeout set to 1 minute (normally takes 10+ minutes)
- **Expected Failure**: Job cancelled due to timeout during image build
- **Trigger**: Manual (`workflow_dispatch`)

---

### Image Reuse Workflows

#### 9. `image-reuse-auth-error.yaml`
- **Base Workflow**: `image-reuse.yaml`
- **Error Type**: Authentication Error
- **Error Location**: `publish` job, Docker login step
- **Error Details**: Invalid Quay.io credentials hardcoded in login step
- **Expected Failure**: Docker login fails with authentication error
- **Trigger**: Manual (`workflow_dispatch`)

#### 10. `image-reuse-package-error.yaml`
- **Base Workflow**: `image-reuse.yaml`
- **Error Type**: Package Version Error
- **Error Location**: `publish` job environment
- **Error Details**: Invalid `GOLANG_VERSION` set to `99.99.99`
- **Expected Failure**: `setup-go` action fails
- **Trigger**: Manual (`workflow_dispatch`)

#### 11. `image-reuse-syntax-error.yaml`
- **Base Workflow**: `image-reuse.yaml`
- **Error Type**: Syntax Error
- **Error Location**: `publish` job, QEMU setup step
- **Error Details**: Missing `name` field for the step (line 59)
- **Expected Failure**: YAML parsing error
- **Trigger**: Manual (`workflow_dispatch`)

#### 12. `image-reuse-timeout-error.yaml`
- **Base Workflow**: `image-reuse.yaml`
- **Error Type**: Timeout Error
- **Error Location**: `publish` job
- **Error Details**: Job timeout set to 1 minute
- **Expected Failure**: Job cancelled during image build
- **Trigger**: Manual (`workflow_dispatch`)

---

### CodeQL Workflows

#### 13. `codeql-auth-error.yml`
- **Base Workflow**: `codeql.yml`
- **Error Type**: Authentication Error
- **Error Location**: `CodeQL-Build` job environment
- **Error Details**: Invalid `GITHUB_TOKEN` set to `ghp_INVALID_TOKEN_FOR_TESTING_AUTH_ERROR_1234567890`
- **Expected Failure**: CodeQL initialization fails due to authentication error
- **Trigger**: Manual (`workflow_dispatch`)

#### 14. `codeql-package-error.yml`
- **Base Workflow**: `codeql.yml`
- **Error Type**: Package Version Error
- **Error Location**: `CodeQL-Build` job, Go setup step
- **Error Details**: Invalid `go-version-file` reference to `nonexistent-go-version-file.mod`
- **Expected Failure**: `setup-go` action fails to find the file
- **Trigger**: Manual (`workflow_dispatch`)

#### 15. `codeql-syntax-error.yml`
- **Base Workflow**: `codeql.yml`
- **Error Type**: Syntax Error
- **Error Location**: `CodeQL-Build` job, CodeQL init step
- **Error Details**: Missing `name` field for the step (line 47)
- **Expected Failure**: YAML parsing error
- **Trigger**: Manual (`workflow_dispatch`)

#### 16. `codeql-timeout-error.yml`
- **Base Workflow**: `codeql.yml`
- **Error Type**: Timeout Error
- **Error Location**: `CodeQL-Build` job
- **Error Details**: Job timeout set to 1 minute (normally takes 5+ minutes)
- **Expected Failure**: Job cancelled during CodeQL analysis
- **Trigger**: Manual (`workflow_dispatch`)

---

### PR Title Check Workflows

#### 17. `pr-title-check-auth-error.yml`
- **Base Workflow**: `pr-title-check.yml`
- **Error Type**: Authentication Error
- **Error Location**: `validate` job environment
- **Error Details**: Invalid `GITHUB_TOKEN` set to `ghp_INVALID_TOKEN_FOR_TESTING_AUTH_ERROR_1234567890`
- **Expected Failure**: PR title checker fails to authenticate
- **Trigger**: Manual (`workflow_dispatch`)

#### 18. `pr-title-check-package-error.yml`
- **Base Workflow**: `pr-title-check.yml`
- **Error Type**: Package Version Error
- **Error Location**: `validate` job, action reference
- **Error Details**: Invalid action version `@v99.99.99`
- **Expected Failure**: Action resolution fails
- **Trigger**: Manual (`workflow_dispatch`)

#### 19. `pr-title-check-syntax-error.yml`
- **Base Workflow**: `pr-title-check.yml`
- **Error Type**: Syntax Error
- **Error Location**: `validate` job, PR title checker step
- **Error Details**: Missing `with` keyword before parameters (line 38)
- **Expected Failure**: YAML parsing error
- **Trigger**: Manual (`workflow_dispatch`)

#### 20. `pr-title-check-timeout-error.yml`
- **Base Workflow**: `pr-title-check.yml`
- **Error Type**: Timeout Error
- **Error Location**: `validate` job
- **Error Details**: Job timeout set to 0.017 minutes (~1 second)
- **Expected Failure**: Job cancelled immediately
- **Trigger**: Manual (`workflow_dispatch`)

---

## Usage Instructions

### Running Error Workflows

1. Navigate to the **Actions** tab in your GitHub repository
2. Select the desired error workflow from the left sidebar
3. Click **Run workflow** button
4. Select the branch (usually `master` or your test branch)
5. Click **Run workflow** to execute

### Expected Behavior

All error workflows are designed to fail. The failure should occur at the specific point where the error was injected:

- **Authentication Errors**: Fail during API calls or login attempts
- **Package Errors**: Fail during dependency resolution or setup
- **Syntax Errors**: Fail during workflow parsing (won't even start)
- **Timeout Errors**: Fail after the specified timeout period

### Monitoring and Testing

These workflows can be used to test:

1. **CI/CD Monitoring Systems**: Verify that monitoring tools detect and alert on failures
2. **Error Handling**: Test how your systems handle different types of CI failures
3. **Notification Systems**: Ensure failure notifications are sent correctly
4. **Recovery Procedures**: Practice incident response for CI failures
5. **Documentation**: Validate troubleshooting guides with real failure scenarios

---

## Important Notes

### Safety Measures

1. **Manual Trigger Only**: All workflows use `workflow_dispatch` to prevent automatic execution
2. **No Production Impact**: Workflows don't push to production registries or deploy code
3. **Clear Labeling**: All workflows are clearly marked with "ERROR TEST" in their names
4. **Documentation**: Each workflow includes comments explaining the intentional error

### Maintenance

When updating base workflows:

1. Review corresponding error workflows
2. Update error injection points if workflow structure changes
3. Test error workflows to ensure they still fail as expected
4. Update this documentation with any changes

### Cleanup

These workflows can be safely deleted if no longer needed. They don't affect production workflows or systems.

---

## Troubleshooting

### Workflow Doesn't Appear in Actions Tab

- Ensure the workflow file is in `.github/workflows/` directory
- Check that the YAML syntax is valid (except for syntax-error workflows)
- Verify the file has `.yaml` or `.yml` extension

### Workflow Succeeds Instead of Failing

- Review the error injection point in the workflow
- Check if the base workflow has changed
- Verify that the error condition is still valid

### Cannot Trigger Workflow

- Ensure you have write permissions to the repository
- Check that the workflow is on the correct branch
- Verify `workflow_dispatch` trigger is properly configured

---

## Summary

| Workflow Base | Auth Error | Package Error | Syntax Error | Timeout Error |
|---------------|------------|---------------|--------------|---------------|
| ci-build | ✅ | ✅ | ✅ | ✅ |
| image | ✅ | ✅ | ✅ | ✅ |
| image-reuse | ✅ | ✅ | ✅ | ✅ |
| codeql | ✅ | ✅ | ✅ | ✅ |
| pr-title-check | ✅ | ✅ | ✅ | ✅ |

**Total Error Workflows**: 20

---

## Contact

For questions or issues with these error testing workflows, please refer to the main project documentation or contact the DevOps team.

---

*Last Updated: 2026-06-13*
*Version: 1.0*