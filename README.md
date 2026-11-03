# YunExpress Tracking API & Webhooks

Build YunExpress shipment tracking into an order page, operations dashboard, notification workflow, or customer-support tool with the Track123 Tracking API. Track123 provides one integration flow for carrier detection, tracking registration, tracking queries, normalized statuses, and webhook delivery.

This repository is a practical integration guide for YunExpress tracking with Track123. It is **not** official YunExpress API documentation, and it does not provide a YunExpress SDK or test tracking numbers.

## What you can build

With the Track123 API, an application can register YunExpress tracking numbers, retrieve the latest shipment events, and receive updates at an HTTPS webhook endpoint. The same API design can be used across the other carriers supported by Track123, which helps teams avoid building and maintaining a different integration for every carrier.

For YunExpress, use the Track123 carrier code `yunexpress` when the carrier is already known.

When the carrier code is not known, use [Carrier Detection](https://docs.track123.com/reference/detection-courier) before registering the shipment.

- [Track YunExpress packages on Track123](https://www.track123.com/carriers/yunexpress)

- [YunExpress API integration page](https://www.track123.com/carriers/yunexpress/api)

- [Track123 API documentation](https://docs.track123.com)

## Integration flow

1. Get a Track123 API key and read the [API Guide](https://docs.track123.com/reference/request).
2. Register one or more tracking numbers with the Track123 tracking registration endpoint.

3. Set `courierCode` to `yunexpress` when registering a shipment known to be handled by YunExpress.

3. Detect the carrier when a verified carrier code is unavailable.

4. Query tracking results when your application needs the current state, or configure a webhook for event-driven updates.
5. Map Track123's normalized delivery status and sub-status into your own order, notification, and support workflows.

Track123 documents a maximum of **100 tracking numbers per registration request** and **100 tracking numbers per tracking query request**. For implementation details, field definitions, and the current response schema, use the linked API references as the source of truth.

## Register a YunExpress tracking number

The registration endpoint is `POST https://api.track123.com/gateway/open-api/tk/v2.1/track/import`. Replace the placeholder values below with your own API key and your own YunExpress tracking number. Do not commit credentials or customer tracking numbers to a repository.

```bash
curl --request POST 'https://api.track123.com/gateway/open-api/tk/v2.1/track/import' \
  --header 'Content-Type: application/json' \
  --header 'Track123-Api-Secret: YOUR_TRACK123_API_KEY' \
  --data '[
    {
      "trackNo": "YOUR_yunexpress_TRACKING_NUMBER",

      "courierCode": "yunexpress"

      "courierCode": "YOUR_DETECTED_COURIER_CODE"

    }
  ]'
```

The request example is intentionally limited to the carrier code and tracking number. Review [Register package trackings](https://docs.track123.com/reference/register-package-trackings) before adding optional order, customer, destination, or carrier-specific fields.

## Retrieve tracking results

Use `POST https://api.track123.com/gateway/open-api/tk/v2.1/track/query` to retrieve the results for tracking numbers you have registered. A typical query supplies a `trackNoInfos` list. The response should be handled as a tracking record, not as a guarantee of shipment delivery; carrier source data can be delayed, incomplete, or revised.

```bash
curl --request POST 'https://api.track123.com/gateway/open-api/tk/v2.1/track/query' \
  --header 'Content-Type: application/json' \
  --header 'Track123-Api-Secret: YOUR_TRACK123_API_KEY' \
  --data '{
    "trackNoInfos": [
      { "trackNo": "YOUR_yunexpress_TRACKING_NUMBER" }
    ]
  }'
```

See [Get package trackings](https://docs.track123.com/reference/get-package-trackings) for the current request and response reference.

## Choose polling, webhooks, or both

Polling is useful when a dashboard refreshes on demand or when an application needs a current result for a specific order. Webhooks are useful when an application should react automatically to a new tracking event without continuously polling every shipment. Many integrations use both: webhooks for timely updates and an on-demand query for reconciliation or a customer-facing refresh.

Read the related implementation guides:

- [YunExpress tracking webhook](webhook.md)
- [YunExpress tracking statuses](tracking-status.md)
- [YunExpress API errors and rate limits](errors-and-rate-limits.md)

## Before going live

- Keep the `Track123-Api-Secret` on the server side; do not expose it in browser code.
- Send only tracking numbers that your business is authorized to process.
- Validate webhook signatures and make webhook handling idempotent.
- Respect Track123 rate limits and use bounded retries for temporary failures.
- Review the current [error codes](https://docs.track123.com/reference/status-code), [rate limits](https://docs.track123.com/reference/rate-limit), and [account quota](https://docs.track123.com/reference/get-account-quota) references before increasing volume.

## Additional resources

- [All supported carriers](https://www.track123.com/carriers)
- [Track123 developer documentation](https://docs.track123.com)
- [Track YunExpress packages and view carrier information](https://www.track123.com/carriers/yunexpress)

Carrier names and websites are used only to identify compatible carriers. Track123 is not affiliated with or endorsed by YunExpress unless explicitly stated.
