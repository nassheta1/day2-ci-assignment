# CI Pipeline Design

## 1. Events

The workflow will run on:
- Push to main, excluding Markdown-only changes and changes limited to docs/.
- Pull requests targeting main.
- Manual execution using workflow_dispatch.
- A scheduled run every Monday at 07:00 UTC.

These events allow continuous validation of changes, pull request checks, manual testing, and periodic verification.

## 2. Jobs and Dependencies

The workflow will contain the following jobs:

- test: Run the Python test suite.
- package: Build a versioned tar.gz package after test succeeds.
- verify: Download and test the package without checking out the repository.
- report: Report the results of test and package, even if one fails or is skipped.
- pr-comment: Comment on pull requests when the package job succeeds.

Job graph:

test -> package -> verify
test ----\
          -> report
package -/

package -> pr-comment (pull requests only)

## 3. Artifact

The main artifact is dist/temperature-<version>.tar.gz.
It contains temperature.py and README.md.

The test job also uploads a JUnit test report named test-report.

## 4. Pipeline Type

This is Continuous Integration because it automatically tests and packages changes without deploying the application to production.

## 5. Package Behavior on Test Failure

The package job must be skipped when tests fail because packaging should only happen after the application passes its tests. The report job must still run to record the failure.
