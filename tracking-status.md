# YunExpress Tracking Statuses with Track123

Carrier tracking events vary in wording, detail, timing, and language. Track123 normalizes YunExpress shipment events into one delivery-status model so an application can use consistent order timelines, notification rules, reporting, and support workflows across carriers.

These are Track123 status values. They are **not** a reproduction of YunExpress's own event taxonomy, and a normalized status should not be presented as a guarantee of delivery.

## The nine Track123 delivery statuses

Track123 documents nine key delivery statuses for package tracking:

| Status | Meaning | Suggested application behavior |
| --- | --- | --- |
| `INIT` | A new shipment has been added and is pending tracking. | Store the record; wait for tracking information or the next update. |
| `NO_RECORD` | No tracking information is available yet. | Avoid treating this as a delivery failure; retry according to your own workflow. |
| `INFO_RECEIVED` | The carrier has received the shipper's request and is preparing to pick up the package. | Show that shipment information has been received. |
| `IN_TRANSIT` | The shipment is moving through the carrier network. | Update the customer timeline and continue monitoring. |
| `WAITING_DELIVERY` | The shipment is out for delivery or available at a collection point. | Trigger an arrival or pickup-ready notification where appropriate. |
| `DELIVERY_FAILED` | A delivery attempt failed, for example because of address or recipient availability. | Create a support or customer-action workflow. |
| `ABNORMAL` | The shipment is damaged, returned, held by customs, or in another abnormal condition. | Escalate for review rather than automatically marking the order complete. |
| `DELIVERED` | The parcel was delivered successfully. | Complete the delivery timeline and stop routine tracking updates. |
| `EXPIRED` | There has been no tracking information for 30 days after the shipment was added. | Flag the record for reconciliation or customer-support review. |

Use the current [Track123 delivery status reference](https://docs.track123.com/reference/package-status) when implementing status-dependent behavior. The documentation is the authoritative source if enum values or definitions change.

## Use sub-status for more detail

The top-level status gives a stable, cross-carrier view of the shipment lifecycle. Use Track123 [delivery sub-statuses](https://docs.track123.com/reference/package-status-copy) when your application needs more precise context for a customer-facing timeline, an exception queue, or a shipping analytics report.

Do not hard-code raw carrier event text as the only state in your application. Raw text can change without notice, may be localized, and may not be comparable between carriers. A durable implementation keeps the normalized Track123 status for workflow logic while retaining the original event information for display or troubleshooting where appropriate.

## Suggested workflow mapping

### Customer-facing order page

Use the normalized status as the main progress label, then show carrier event detail beneath it when available. For example, `INFO_RECEIVED` can communicate that shipping information exists, while `IN_TRANSIT` indicates active movement and `WAITING_DELIVERY` can prompt the customer to expect delivery or pickup.

### Notifications

Use `WAITING_DELIVERY`, `DELIVERED`, `DELIVERY_FAILED`, and `ABNORMAL` as potential notification triggers. Make every notification idempotent: if Track123 delivers the same status more than once, a customer should not receive duplicate messages.

### Operations and support

Route `DELIVERY_FAILED`, `ABNORMAL`, and `EXPIRED` records to an exception workflow. `NO_RECORD` is normally a tracking-information state rather than proof that a shipment was lost; wait or query again before escalating it.

## Status update sources

You can receive status changes through a [Track123 webhook](webhook.md) or retrieve the current record with [Get package trackings](https://docs.track123.com/reference/get-package-trackings). Webhooks support timely event-driven updates; an explicit query is useful for on-demand refreshes and reconciliation.

For known YunExpress shipments, register the tracking number with `courierCode` set to `yunexpress` before querying or awaiting updates.

## Resources

- [Track123 delivery status reference](https://docs.track123.com/reference/package-status)
- [Track123 delivery sub-status reference](https://docs.track123.com/reference/package-status-copy)
- [YunExpress tracking webhook](webhook.md)
- [Track YunExpress packages on Track123](https://www.track123.com/carriers/yunexpress)
- [Back to YunExpress tracking API](README.md)
