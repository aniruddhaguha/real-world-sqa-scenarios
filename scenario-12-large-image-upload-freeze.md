# Scenario 12: Large Image Upload Freezes App with No Error Message

## Scenario

Uploading **small images works perfectly**, but uploading **large images freezes the app** without any error message. No crash logs appear on the server.

---

## Assumptions

- The app supports image uploads via a REST API (multipart/form-data).
- The server has a maximum file size limit (e.g., 10MB), but the client is not validating this before upload.
- The app reads the entire file into memory before sending — causing memory pressure on large files.
- "Freeze" means the UI becomes unresponsive — possibly due to a blocking operation on the main thread.
- No crash logs appear because the failure happens client-side (memory exhaustion, timeout) before the request reaches the server.
- No user-facing error is shown because there is no proper error handling for this case.


---

## Test Ideas

### File Size & Memory Testing
- Upload images of increasing sizes: 1MB, 5MB, 10MB, 20MB, 50MB, 100MB — identify the threshold at which the freeze occurs.
- Monitor device memory usage during upload using profiling tools (Android Profiler, Xcode Instruments).
- Check if the app loads the entire file into memory at once vs. using a streaming/chunked approach.
- Test on low-memory devices and high-memory devices — confirm if the issue is memory-constrained.

### API Limit Testing
- Check the server's configured max request body size (e.g., `client_max_body_size` in Nginx, `maxRequestSize` in Spring).
- Send a large file directly via Postman/cURL — observe server response (413 Request Entity Too Large, timeout, etc.).
- Verify the client does not have a pre-upload file size check.
- Test if the API returns an error for oversized files — and if the client handles it gracefully.

### Timeout Behavior Testing
- Set a known large upload and observe if a timeout fires on the client.
- Check the client's HTTP timeout configuration for upload requests.
- Test on slow network (3G simulation) — large uploads may time out before completing.
- Verify that upload progress is tracked — does the app show a progress bar or just freeze?

### Client-Side Error Handling Testing
- Deliberately trigger an upload failure (kill the server mid-upload) — verify the app shows a meaningful error.
- Test what happens when the upload is cancelled mid-way — does the app recover cleanly?
- Check if a retry mechanism exists — and if it works correctly after a failed large upload.
- Verify that the main thread is not blocked during upload (upload should run on a background thread).

---

## Edge Cases

- Image file is corrupt or partially downloaded — client tries to read it and crashes silently.
- User uploads a very wide/tall image (e.g., 10,000 x 10,000 px) — even if file size is under limit, memory needed to decode it is enormous.
- File format is unexpected (e.g., HEIC on iOS) — conversion adds memory overhead.
- Network disconnects mid-upload — no retry logic means the upload silently fails without user notification.
- Server processes the upload but takes so long that the client times out — server still stores the file; client shows error.
- Multiple large uploads queued simultaneously — cumulative memory usage causes device-level crash.
- Proxy or CDN has a different (smaller) request size limit than the origin server — request is rejected earlier.

---

## Risks

- **User experience risk:** App freeze with no error message is one of the worst UX failures — leaves users confused and frustrated.
- **Data loss risk:** If upload partially succeeds before freeze, the server may store an incomplete file.
- **Memory risk:** On low-end devices, large file processing may cause the OS to kill the app entirely.
- **Security risk:** No file size validation allows potential DoS attacks — a malicious user can send enormous payloads to overwhelm the server.
- **App store risk:** Persistent crashes or freezes can lead to negative reviews and potential app store rejection.
- **Support overhead risk:** Users unable to upload important content will flood support channels.
