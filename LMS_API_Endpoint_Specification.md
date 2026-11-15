# LMS — REST API Contract Outline (Illustrative)

**Scope:** University LMS video upload, view, notification and deletion. Composite design example — not a real institution or finalized OpenAPI contract.

## Access control
- OAuth 2.0 / OIDC authentication; short-lived JWTs and server-side role/ownership verification.
- Lecturers manage videos only for courses they teach; students view only courses with active enrollment.
- Short-lived object-storage upload URLs and signed CDN playback URLs are scoped to authorized access.

## Endpoints

| Method | URI | Authorization | Purpose |
|---|---|---|---|
| POST | `/courses/{id}/videos/upload-url` | Lecturer assigned to course | Request a pre-signed multipart upload URL |
| POST | `/courses/{id}/videos` | Lecturer assigned to course | Confirm upload and save metadata |
| GET | `/courses/{id}/videos` | Lecturer / enrolled student | List course videos |
| GET | `/videos/{id}/stream` | Lecturer / enrolled student | Get signed CDN URL |
| DELETE | `/videos/{id}` | Lecturer owner / admin | Delete video under retention rules |

## Suggested status lifecycle
`UPLOADING → PROCESSING → READY → DELETED`; processing errors enter `FAILED` for investigation or retry.

## Events
- `video.published`: `eventId`, `videoId`, `courseId`, `publishedAt`, `version`.
- Idempotency key for student notifications: `videoId + studentId + notificationChannel`.
- Dead-letter queue and retry policy for transcoding and notifications.

## Key validations and example error cases
- `401 Unauthorized`: login missing or token invalid.
- `403 Forbidden`: lecturer not assigned to the course or student not enrolled.
- `404 Not Found`: course/video does not exist or is concealed under the API's disclosure policy.
- `413 Payload Too Large`: media size exceeds policy (prefer to enforce prior to storage upload).
- `415 Unsupported Media Type`: unsupported file format.
- `409 Conflict`: video not READY, duplicate confirmation or already deleted depending on endpoint.
- `429 Too Many Requests`: quotas or rate limits exceeded.
- `500/503`: processing/service error; provide correlation ID and retry guidance.

## Nonfunctional requirements to validate
- Multipart resume behavior for intermittent connections and multi-GB files.
- Adaptive HLS playback; 360p / 720p / 1080p output examples.
- Malware scan, file MIME validation, HTTPS, audit trails, signed URL expiration.
- Notification batching and student email/push preferences.
- Soft vs permanent deletion, object storage/CDN cleanup, retention and restore policy.

**All implementation selections and thresholds are illustrative and subject to stakeholder approval.**
