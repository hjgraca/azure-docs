---
title: Azure Event Grid delivery and retry
description: Describes how Azure Event Grid delivers events and how it handles undelivered messages.
services: event-grid
author: tfitzmac
manager: timlt

ms.service: event-grid
ms.topic: article
ms.date: 05/09/2018
ms.author: tomfitz
---

# Event Grid message delivery and retry 

This article describes how Azure Event Grid handles events when delivery is not acknowledged.

Event Grid provides durable delivery. It delivers each message at least once for each subscription. Events are sent to the registered webhook of each subscription immediately. If a webhook does not acknowledge receipt of an event within 60 seconds of the first delivery attempt, Event Grid retries delivery of the event. 

Currently, Event Grid sends each event individually to subscribers. The subscriber receives an array with a single event.

## Message delivery status

Event Grid uses HTTP response codes to acknowledge receipt of events. 

### Success codes

The following HTTP response codes indicate that an event has been delivered successfully to your webhook. Event Grid considers delivery complete.

- 200 OK
- 202 Accepted

### Failure codes

The following HTTP response codes indicate that an event delivery attempt failed. 

- 400 Bad Request
- 401 Unauthorized
- 404 Not Found
- 408 Request timeout
- 414 URI Too Long
- 500 Internal Server Error
- 503 Service Unavailable
- 504 Gateway Timeout

If Event Grid receives an error that indicates the endpoint is unavailable, it tries again to send the event. 

## Retry intervals and duration

Event Grid uses an exponential backoff retry policy for event delivery. If your webhook does not respond or returns a failure code, Event Grid retries delivery on the following schedule:

1. 10 seconds
2. 30 seconds
3. 1 minute
4. 5 minutes
5. 10 minutes
6. 30 minutes
7. 1 hour

Event Grid adds a small randomization to all retry intervals. After one hour, event delivery is retried once an hour.

By default, Event Grid expires all events that are not delivered within 24 hours.

## Next steps

* To view the status of event deliveries, see [Monitor Event Grid message delivery](monitor-event-delivery.md).
* For an introduction to Event Grid, see [About Event Grid](overview.md).
* To quickly get started using Event Grid, see [Create and route custom events with Azure Event Grid](custom-event-quickstart.md).