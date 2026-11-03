# YunExpress Tracking Webhook with Track123

Use a Track123 webhook to receive YunExpress tracking updates at your application instead of relying only on scheduled polling. A webhook is especially useful for order-status notifications, exception handling, customer-support workflows, and dashboards that should update when a new carrier event is available.

This is a Track123 integration guide. It describes Track123's unified webhook delivery flow and is **not** official YunExpress webhook documentation.

## How the webhook flow works

1. Register the shipment with Track123.

2. Include `courierCode: "yunexpress"` when YunExpress is already known.

2. Use [Carrier Detection](https://docs.track123.com/reference/detection-courier) if a confirmed carrier code is unavailable.

3. Configure an HTTPS endpoint in Track123 for tracking notifications.
4. Receive an HTTP POST when Track123 delivers a tracking event.
5. Verify the request signature, identify the tracking record, and safely update your own system.
6. Return a 2xx response only after your application has accepted the delivery.

The webhook should complement—not replace—your own data model. Store the tracking number, your order identifier, the latest normalized status, and enough event metadata to diagnose delivery or reconciliation issues.

## Verify every webhook request

Do not treat an incoming request as trustworthy merely because it reaches your endpoint. Track123's webhook guide documents signing with your Track123 API key and a plain-text timestamp. Verify the signature included in the POST data before acting on an event.

The public guide specifies SHA256 for PHP and HmacSHA256 for Java examples. Use the current [Track123 webhook guide](https://docs.track123.com/reference/getting-started-with-your-api) for the exact payload, signature field, and language-specific verification approach.

A safe server-side handling sequence is:

1. Accept the request over HTTPS.
2. Read the raw request data and timestamp needed for signature verification.
3. Recalculate and compare the signature using your server-side Track123 API key.
4. Reject an invalid signature without changing order state.
5. Use a stable event or tracking-record identifier to prevent duplicate processing.
6. Persist the accepted update and then respond with a 2xx status.

Never expose the API key in client-side JavaScript, a mobile application, or a public repository.

## Retry behavior

When Track123 does not receive an HTTP status from 200 through 299, it may retry a webhook delivery up to seven times. The documented schedule is shown below; build idempotent processing because more than one delivery can represent the same update.

| Delivery attempt | Delay before the attempt |
| --- | ---: |
| 1 | Immediate |
| 2 | 10 minutes |
| 3 | 30 minutes |
| 4 | 1 hour |
| 5 | 4 hours |
| 6 | 12 hours |
| 7 | 36 hours |

If your service temporarily cannot process an event, return a non-2xx response only when a retry is appropriate. Avoid deliberately failing requests for permanent validation errors, because it creates repeated deliveries without resolving the underlying issue.

## Design for idempotency

Webhook events can arrive more than once, and carrier event order may not always match the order in which your application processes requests. Use a deduplication key and make state changes safe to repeat. For example, do not send a second “delivered” customer notification just because a delivery event is received again.

Keep the original tracking data and the latest processed timestamp in your system. If a later query result and a webhook appear inconsistent, use the [Get package trackings](https://docs.track123.com/reference/get-package-trackings) endpoint and the Track123 status references to reconcile the record.

## YunExpress webhook checklist

- Confirm the tracking number was registered successfully.

- Use the carrier code `yunexpress` when registering known YunExpress shipments.

- Use a stable, public HTTPS endpoint with server-side signature verification.
- Return a 2xx response only after persistent processing succeeds.
- Protect against duplicated delivery and out-of-order updates.
- Monitor failed delivery, authentication, and processing errors.

## Resources

- [Track123 webhook guide](https://docs.track123.com/reference/getting-started-with-your-api)
- [Register package trackings](https://docs.track123.com/reference/register-package-trackings)
- [Track123 delivery status reference](https://docs.track123.com/reference/package-status)
- [Track YunExpress packages on Track123](https://www.track123.com/carriers/yunexpress)
- [Back to YunExpress tracking API](README.md)
