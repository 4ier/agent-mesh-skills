---
name: oss-upload
description: "Upload generated artifacts to MinIO-backed OSS via the gateway and return a permanent download URL."
license: Proprietary. LICENSE.txt has complete terms
---

# OSS upload skill

## Purpose
Use this skill whenever you generate a file that the user should download. Upload the file to the gateway OSS service and return ONLY the permanent download URL. Do not return upload details or file metadata to the user.

## Endpoint
- Upload (multipart): `POST http://gateway:3000/oss/upload`
- Upload (raw): `PUT http://gateway:3000/oss/upload/<filename>`
- Download: `GET http://gateway:3000/oss/<objectKey>`

## Size limit
The service enforces a configurable maximum file size (default 100MB). If your file is larger, reduce content or split the output before uploading.

## Upload workflow (recommended)
1) Confirm the output file exists and is within size limits.
2) Upload with multipart form data.
3) Parse the JSON response and return the `url` only (no extra text).

### Example
```bash
# Assume the output is at /app/session/output.pptx
stat /app/session/output.pptx

# Upload
curl -sS -F "file=@/app/session/output.pptx" \
  "http://gateway:3000/oss/upload?name=output.pptx" \
  -o /tmp/oss-upload.json

# Extract URL (print only the URL)
node -e "const fs=require('fs');const data=JSON.parse(fs.readFileSync('/tmp/oss-upload.json','utf8'));process.stdout.write(String(data.url||''));"
```

## Raw upload alternative
```bash
curl -sS -X PUT --data-binary @/app/session/output.pptx \
  -H "Content-Type: application/vnd.openxmlformats-officedocument.presentationml.presentation" \
  "http://gateway:3000/oss/upload/output.pptx"
```

## Response format (do not echo this JSON to the user)
```json
{
  "bucket": "agent-outputs",
  "key": "2025/12/24/uuid-output.pptx",
  "url": "http://localhost:3000/oss/2025/12/24/uuid-output.pptx"
}
```

## Failure handling
If the upload fails:
- respond with a short failure notice (no file details),
- do not include local paths or upload diagnostics.
