# YunExpress Tracking API Errors and Rate Limits

Build a reliable YunExpress tracking integration by handling Track123 API errors, account limits, and retry scenarios safely.

> Error codes and rate limits apply to the Track123 API. They are not YunExpress API error codes.

## Handle errors safely

Check both HTTP responses and Track123 business error codes. Correct invalid requests before retrying them, and preserve enough request context in logs to diagnose failures without exposing credentials or tracking data.

## Respect rate limits

Use bounded retries and backoff for transient failures or throttling. Review the current account and endpoint limits before increasing polling volume.

## Resources

- [Track123 error codes](https://docs.track123.com/reference/status-code)
- [Track123 rate limits](https://docs.track123.com/reference/rate-limit)
- [Track123 account quota](https://docs.track123.com/reference/get-account-quota)

- [YunExpress API integration page](https://www.track123.com/carriers/yunexpress/api)

- [Back to YunExpress tracking API](README.md)
