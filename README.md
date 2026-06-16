# BaRS and Wayfinder

Design thinking, gap analysis, and proposals for the intersection of the Booking and Referral Standard (BaRS) and the Wayfinder / Patient Care Aggregator programme.

---

## Context

**BaRS** (Booking and Referral Standard) is a FHIR R4 API standard that enables interoperable booking and referral workflows across NHS organisations. It defines how systems exchange ServiceRequests, Appointments, and Slots via a standard RESTful interface.

**Wayfinder** (Patient Care Aggregator) surfaces referral, appointment, and pathway information to patients via the NHS App. It collates data from provider systems and presents a unified view of a patient's care journey — upcoming appointments, referral status, and actions they can take.

The overlap between these two programmes is significant: BaRS defines how referrals and bookings flow between organisations, and Wayfinder is how patients see and interact with that data. Aligning them means patients get real-time, accurate visibility into their care without providers needing to build bespoke patient-facing integrations.

---

## Documents

| Document | Description |
|---|---|
| [BaRS API and Wayfinder Gap Analysis (PDF)](./BaRS%20API%20Standard%20and%20Wayfinder%20Producer%20Specification%20Gap%20Analysis-v4-20260616_142243.pdf) | Formal gap analysis comparing the BaRS API standard with the Wayfinder Producer Specification — identifies where they align, where they diverge, and what changes are needed |
| [Adding Tasking via FHIR Task](./bars-tasking-fhir-task.md) | Design for adding `/Task` operations to the BaRS API — enabling workflow tracking, patient-facing tasks, and operational visibility across the referral pathway |

---

## Key Ideas

### 1. Task as the Bridge Between BaRS and Wayfinder

Today, Wayfinder can show patients "you have a referral" and "you have an appointment" — but not the steps in between. Adding FHIR Task to BaRS fills this gap:

- Patient sees their referral is "being triaged"
- Patient is asked to "complete a questionnaire before your appointment"
- Patient can "confirm attendance"
- Patient sees "your GP has been sent the outcome"

Tasks make the invisible operational work visible to patients, without exposing clinical complexity.

### 2. BaRS as the Wayfinder Data Source

Rather than each Trust building bespoke integrations to feed Wayfinder, the BaRS API can serve as the standard data source:

- `GET /ServiceRequest?patient:identifier={nhs-number}` — patient's referrals
- `GET /Appointment?patient:identifier={nhs-number}` — patient's bookings
- `GET /Task?patient:identifier={nhs-number}&owner={patient}` — patient's actionable tasks

Wayfinder aggregators query these standard endpoints rather than proprietary provider feeds.

### 3. Aligning the Producer Specification with BaRS

The Wayfinder Producer Specification defines what data providers must surface. The gap analysis identifies where this specification can be satisfied by BaRS-standard endpoints — reducing the implementation burden on providers who already support BaRS.

### 4. Patient-Initiated Actions via BaRS

Wayfinder is not just read-only. Patients can:

- Cancel or reschedule appointments → `PUT /Appointment` (status: cancelled) or Rebook flow
- Confirm attendance → `PATCH /Task` (complete the "confirm attendance" task)
- Submit questionnaires → `POST /QuestionnaireResponse` linked to a Task
- Request a callback → `POST /Task` (code: contact-patient, requester: patient)

These are standard BaRS operations, now triggered by the patient via the NHS App rather than by a clinician or admin user.

---

## Strategic Direction

The long-term vision is:

1. **BaRS becomes the standard integration layer** for referral and booking data across the NHS
2. **Wayfinder consumes BaRS endpoints** rather than requiring providers to build separate patient-facing feeds
3. **FHIR Task enables workflow visibility** — patients and clinicians see the same operational state
4. **Providers implement once** — supporting BaRS satisfies both inter-org interoperability and patient-facing requirements

This reduces duplication, lowers the integration burden on providers, and ensures patients see consistent, real-time data regardless of which Trust they're referred to.

---

## Related Repositories

| Repository | Description |
|---|---|
| [BaRS-and-e-RS](https://github.com/RichardWardNHSD/BaRS-and-e-RS) | BaRS and e-Referral Service intersection — translation layer, worklists, HL7 V3 replacement |
| [BaRS-Appointments-StandardPattern](https://github.com/RichardWardNHSD/BaRS-Appointments-StandardPattern) | Standard Pattern for Appointment management (Slot, Book, Cancel, Reschedule, Rebook) |
| [BaRS-Onboarding](https://github.com/RichardWardNHSD/BaRS-Onboarding) | Onboarding documentation and processes |
