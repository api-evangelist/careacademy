---
name: Pull an agency's CareAcademy compliance report
description: >-
  Retrieve an organization's course-completion and compliance data from
  CareAcademy as JSON or CSV, paging through large result sets.
api: openapi/careacademy-openapi-original.yml
operations:
  - "GET /compliance_report/{organizationIntegrationId}"
---

# Pull an agency's compliance report

Use this to surface CareAcademy training completions, exam results, and compliance status inside your product.

## Auth
HTTP Basic authentication with the provisioned integration credentials. Base URL `https://staging.careacademy.com/api/v1` for testing.

## Steps
1. **Request the report.** `GET /compliance_report/{organizationIntegrationId}` with required query params `filter[startDate]` (ISO 8601) and `filter[locationId]`. Optional: `filter[endDate]` (defaults to now), `filter[completeIncomplete]` (`complete` default, or `incomplete`).
2. **Choose a format.** Set the `Accept` header to `application/json` or `text/csv`. JSON returns a `data[]` array of course records (agencyName, courseName, completedDate, examResultPercentage, isInitial, isAnnual, ...).
3. **Page through results.** When more data exists the response includes a `links` object; follow `links.next.href` (a relative URL carrying `page[after]=N`, a zero-based page index) until it is absent.

## Rules
- `organizationIntegrationId` is the same value you used when creating the organization.
- A `404` means no organization matched the integration ID; a `400` means a filter param is missing or an unparseable date was supplied. See `errors/careacademy-problem-types.yml`.
- The report contains PII/PHI (names, birth dates, license/registration IDs) — handle per your HIPAA obligations.
