# YunExpress Tracking API Errors and Rate Limits

A reliable YunExpress tracking integration needs more than a successful request path. Your application should distinguish invalid requests from temporary service failures, respect Track123 throughput limits, protect credentials, and make retry behavior predictable.

The codes and limits below apply to the Track123 API. They are **not** YunExpress API error codes.

## Track123 rate limit

Track123 documents a maximum of **5 requests per second per endpoint**. This is an endpoint-level limit, so distributing requests across many shipments does not remove the need to control the rate sent to a single Track123 API endpoint.

When the limit is exceeded, Track123 documents the business error code `A0706` (`RATE_LIMIT_EXCEEDED`). Reduce request frequency, wait briefly, and retry with bounded backoff. Do not immediately replay a large batch, because that can prolong throttling.

### Recommended client behavior

1. Queue requests instead of sending all shipment checks at once.
2. Limit concurrent calls to the same endpoint.
3. On `A0706`, pause or slow that endpoint's queue.
4. Retry temporary failures with a capped exponential backoff and jitter.
5. Stop retrying after a defined limit and record the failure for review.
6. Prefer [webhooks](webhook.md) for event-driven updates rather than polling every shipment at a high frequency.

See the current [Track123 rate limit reference](https://docs.track123.com/reference/rate-limit) before changing production polling volume.

## Read HTTP and business error codes together

Check both the HTTP response and the Track123 business code in the response body. The following examples are documented by Track123 and help determine whether a request should be corrected, retried, or escalated.

| Code | Meaning | Recommended action |
| --- | --- | --- |
| `A0301` | Unauthorized access | Check the server-side API key, account access, and request authentication. Do not retry unchanged credentials. |
| `A0400` | Request parameter error | Validate the request format and correct the invalid field before sending again. |
| `A0410` | Required parameter is null | Add the missing required field, such as a tracking number, then retry once corrected. |
| `A0706` | Rate limit exceeded | Reduce request frequency and retry after a short delay with backoff. |
| `B0001` | System execution error | Treat as a possible temporary service problem; retry carefully and retain diagnostics. |
| `B0100` | System execution timeout | Retry a limited number of times; avoid duplicating downstream processing. |
| `B0210` | Flow limiting | Slow the client and review account or endpoint throughput. |

For the full and current list, including query-specific codes, use the [Track123 error code reference](https://docs.track123.com/reference/status-code).

## Avoid unsafe retries

Before retrying, decide whether the failure is transient or permanent. Authentication and parameter errors are usually permanent until corrected. Rate limits and timeouts can be transient, but retries should be bounded. Log the endpoint, request correlation identifier, business code, and retry count; do not log the API key or unnecessary customer data.

If an operation can create or update a tracking record, make the calling workflow idempotent. A timeout does not always mean the server performed no work, so an immediate duplicate request may create confusion in your own system.

## Account quota and production monitoring

Rate limits control request speed; account quota controls the usage available to an account. Monitor both before growing polling traffic or onboarding a new batch of orders. The [Get account quota](https://docs.track123.com/reference/get-account-quota) endpoint and the Track123 account controls are the appropriate references for current account limits.

Set alerts for repeated unauthorized requests, sustained throttling, webhook delivery failures, and abnormal tracking states. These signals are more actionable than only monitoring HTTP success rates.

## YunExpress integration reference

For carrier-level tracking lookup and the Track123 integration entry point, see the [YunExpress API integration page](https://www.track123.com/carriers/yunexpress/api). Use the general Track123 API references for endpoint schemas and error-code definitions.

## Resources

- [Track123 error codes](https://docs.track123.com/reference/status-code)
- [Track123 rate limits](https://docs.track123.com/reference/rate-limit)
- [Track123 account quota](https://docs.track123.com/reference/get-account-quota)
- [YunExpress tracking webhook](webhook.md)
- [Back to YunExpress tracking API](README.md)
