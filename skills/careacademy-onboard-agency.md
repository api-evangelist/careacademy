---
name: Onboard an agency and its caregivers to CareAcademy
description: >-
  Create a CareAcademy free-trial organization from your product, add its
  locations and caregivers, and hand the user off via Single Sign-On.
api: openapi/careacademy-openapi-original.yml
operations:
  - "POST /organizations/{organizationIntegrationId}"
  - "POST /locations/{organizationIntegrationId}"
  - "POST /practitioners"
  - "GET /sign_in_url"
---

# Onboard an agency and its caregivers

Use this when a customer indicates they do **not** already have a CareAcademy account.

## Auth
All calls use HTTP Basic authentication with the integration credentials provisioned by CareAcademy staff. Base URL: `https://staging.careacademy.com/api/v1` for testing.

## Steps
1. **Create the organization.** `POST /organizations/{organizationIntegrationId}` where `organizationIntegrationId` is the agency's unique ID in your system. Body requires `organizationName`, `organizationState`, `adminFirstName`, `adminLastName`, `adminEmail`, `adminIntegrationId`. A `201` creates the free trial and returns a temporary `signInUrl`; a `200` means the org already existed and was linked.
2. **Add locations (if multi-location).** If the agency has multiple branches, `POST /locations/{organizationIntegrationId}` with a single Location or an array. Capture each location `id` to assign caregivers. Single-location agencies can skip this and pass `state` on the practitioner instead.
3. **Add caregivers.** For each caregiver, `POST /practitioners` with required `integrationId`, `name` (FHIR HumanName), `telecom` (FHIR ContactPoint), `email`, `hireDate`, `organizationIntegrationId`, and optional `locationId`/`studentType`. Re-POSTing with a known `integrationId` links/updates rather than duplicating.
4. **Sign the user in.** `GET /sign_in_url?userIntegrationId=...` returns a temporary `signInUrl`; redirect the browser to it to drop the user into their CareAcademy dashboard. `userIntegrationId` must match the practitioner `integrationId` used in step 3.

## Rules
- Integration IDs are caller-owned and must be unique within your integration context; reuse them across calls.
- Dates (`hireDate`, `birthDate`, `registrationDate`) follow RFC3339 full-date.
- On errors, read `errors[]` (field problems) and `additionalInformation[]` (warnings); `parameterName: "_"` is a general message. See `errors/careacademy-problem-types.yml`.
- No idempotency-key header — rely on the integration-ID upsert behavior.
