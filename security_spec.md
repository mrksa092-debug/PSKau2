# Security Specification - رحلة أمومة

## Data Invariants
1. A Chat message must belong to the authenticated user.
2. A User can only access one Patient record (Linked via fileNumber).
3. Patient records are read-only for users (Hospital controlled).
4. Appointments are read-only for users.
5. All timestamps must be server-generated.

## The Dirty Dozen Payloads (Rejection Tests)
1. **Identity Spoofing**: `GET /users/other-user-id`
2. **PII Leak**: `GET /patients/any-file-number` without linking.
3. **Shadow Update**: `PATCH /users/my-id { isAdmin: true }`
4. **ID Poisoning**: `GET /patients/very-long-garbage-id-123456789...`
5. **Orphaned Write**: `POST /users/my-id/chats { userId: 'someone-else' }`
6. **Future Timestamp**: `POST /users/my-id/chats { timestamp: '2099-01-01' }`
7. **Size Explosion**: `POST /users/my-id/chats { text: 'a'.repeat(100000) }`
8. **Malicious Enum**: `POST /users/my-id/chats { sender: 'hacker' }`
9. **State Skip**: `PATCH /patients/file123 { pregnancyStartDate: 'last-month' }` (ReadOnly check)
10. **Unauthorized Appt**: `GET /patients/other-file/appointments`
11. **Malicious Link**: `PATCH /users/my-id { patientFileNumber: 'admin-file' }` (If controlled)
12. **Anonymous PII Scrape**: `LIST /patients`

## Test Runner (firestore.rules.test.ts)
(Simulated logic in rules)
