# Instagram Publishing Failure Analysis - Error 2207052

## Error Details
- **Error Code:** 2207052 (Media fetch failed)
- **Full Error:** "Only photo or video can be accepted as media type"
- **Error Subcode:** 9004
- **Media URL:** `https://postiz.pawalink.com/uploads/2026/04/25/100951099393419adf810101aca2f399dce.jpg`

## Root Cause

The error is a **Meta/Instagram Graph API issue** - Instagram's backend cannot fetch the image from your server. This is NOT a Postiz bug but rather a change in how Instagram fetches media from external URLs.

This issue began affecting self-hosted applications around March 2025. Multiple users on different platforms (Mixpost, other Instagram schedulers) have reported the same error.

## Verified Server Configuration

| Check | Status |
|-------|--------|
| Image accessible via HTTPS | ✅ Pass |
| HTTP 200 response | ✅ Pass |
| Content-Type: image/jpeg | ✅ Pass |
| Valid JPEG file | ✅ Pass (1000x1000, JFIF) |
| Content-Length present | ✅ Pass (649561 bytes) |
| CORS headers | ✅ Pass |

## Technical Details

Instagram uses `facebookexternalhit` bot to fetch media. This bot is failing to fetch from your server due to **TLS/SSL handshake issues**. The image itself is valid, but Instagram's server cannot download it.

Common symptoms:
- Error message shows the correct URL but fails to fetch
- All images fail regardless of format/size
- Works for Facebook posting but fails for Instagram

## Solutions

### Option 1: Use HTTP Instead of HTTPS (Quick Fix)

Modify docker-compose.yaml:

```yaml
environment:
  # Change from HTTPS to HTTP
  NEXT_PUBLIC_UPLOAD_DIRECTORY: 'http://postiz.pawalink.com:4007/uploads'
```

Requirements:
- Ensure HTTP port 4007 is accessible externally
- Or configure reverse proxy to serve HTTP

### Option 2: Use Cloudflare R2 Storage

Switch from local storage to S3-compatible storage:

```yaml
environment:
  STORAGE_PROVIDER: 'cloudflare'
  CLOUDFLARE_ACCOUNT_ID: 'your-account-id'
  CLOUDFLARE_ACCESS_KEY: 'your-access-key'
  CLOUDFLARE_SECRET_ACCESS_KEY: 'your-secret-access-key'
  CLOUDFLARE_BUCKETNAME: 'your-bucket-name'
  CLOUDFLARE_BUCKET_URL: 'https://your-bucket-url.r2.cloudflarestorage.com/'
  CLOUDFLARE_REGION: 'auto'
```

### Option 3: Use Different Reverse Proxy

Some users report success with Nginx or Traefik instead of Caddy.

### Option 4: Wait for Meta Fix

This is a known issue and Meta may fix it in the future. Monitor:
- https://developers.facebook.com/community/threads/1593241148449585/
- https://developers.facebook.com/community/threads/532451459921319/

## References

- Instagram Error Codes: https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/reference/error-codes/
- Community Thread: https://developers.facebook.com/community/threads/1593241148449585/
- Mixpost Issue: https://github.com/inovector/mixpost/issues/197

## Current Environment

- **Postiz Version:** ghcr.io/gitroomhq/postiz-app:latest
- **Storage:** Local (postiz-uploads volume)
- **Reverse Proxy:** Caddy
- **Upload URL:** https://postiz.pawalink.com/uploads/