# Test Principles

## Summary

The goal is to ensure both quality assurance and a smooth development flow.

Perform effective testing. Unstable or needless tests do not constitute quality assurance.

## Criteria

* All tests must pass without exception.
* All user scenarios are covered.
* Maintain independence, parallel execution capability, and reproducibility.
* Avoid overly fragile tests that break with implementation changes.

## Boundary

* Basically, mocks are limited to external dependencies (avoid mocking internal implementations).
* Fixed-time waits are prohibited.
* Sharing state between tests is prohibited.
* Non-important tests should not be executed.
    * A non-important test is one where the cost outweighs the value (e.g., unrelated to user scenarios, purely design-related concerns).
