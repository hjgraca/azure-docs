---
title: Azure Event Grid concepts
description: Describes Azure Event Grid and its concepts. Defines several key components of Event Grid.
services: event-grid
author: banisadr
manager: timlt

ms.service: event-grid
ms.topic: article
ms.date: 04/24/2018
ms.author: babanisa
---

# Concepts in Azure Event Grid

This article describes the main concepts in Azure Event Grid.

## Events

An event is the smallest amount of information that fully describes something that happened in the system. Every event has common information like: source of the event, time the event took place, and unique identifier. Every event also has specific information that is only relevant to the specific type of event. For example, an event about a new file being created in Azure Storage has details about the file, such as the `lastTimeModified` value. Or, an Event Hubs event has the URL of the Capture file. Each event is limited to 64 KB of data.

## Event sources/publishers

An event source is where the event happens. For example, Azure Storage is the event source for blob created events. Azure VM Fabric is the event source for virtual machine events. Event sources are responsible for publishing events to Event Grid.

For information about implementing any of the supported Event Grid sources, see [Event sources in Azure Event Grid](event-sources.md).

## Topics

Publishers categorize events into topics. The event grid topic includes an endpoint where the publisher sends events. To respond to certain types of events, subscribers decide which topics to subscribe to. Topics also provide an event schema so that subscribers can learn how to consume the events appropriately.

System topics are built-in topics provided by Azure services. Custom topics are application and third-party topics.

When designing your application, you have flexibility when deciding how many topics to create. For large solutions, create a custom topic for each category of related events. For example, consider an application that sends events related to modifying user accounts and processing orders. It's unlikely any event handler wants both categories of events. Create two custom topics and let event handlers subscribe to the one that interests them. For small solutions, you might prefer to send all events to a single topic. Event subscribers can filter for the event types they want.

## Event subscriptions

A subscription instructs Event Grid on which events on a topic a subscriber is interested in receiving. A subscription also holds information on how events should be delivered to the subscriber.

## Event handlers

From an Event Grid perspective, an event handler is the place where the event is sent. The handler takes some further action to process the event. Event Grid supports multiple subscriber types. Depending on the type of subscriber, Event Grid follows different mechanisms to guarantee the delivery of the event. For HTTP webhook event handlers, the event is retried until the handler returns a status code of `200 – OK`. For Azure Storage Queue, the events are retried until the Queue service is able to successfully process the message push into the queue.

For information about implementing any of the supported Event Grid handlers, see [Event handlers in Azure Event Grid](event-handlers.md).

## Filters

When subscribing to an event grid topic, you can filter the events that are sent to the endpoint. You can filter by event type, or subject pattern. For more information, see [Event Grid subscription schema](subscription-creation-schema.md).

## Security

Event Grid provides security for subscribing to topics, and publishing topics. When subscribing, you must have adequate permissions on the resource or event grid topic. When publishing, you must have a SAS token or key authentication for the topic. For more information, see [Event Grid security and authentication](security-authentication.md).

## Failed delivery

If Event Grid can't confirm that an event has been received by the subscriber's endpoint, it redelivers the event. For more information, see [Event Grid message delivery and retry](delivery-and-retry.md).

## Next steps

* For an introduction to Event Grid, see [About Event Grid](overview.md).
* To quickly get started using Event Grid, see [Create and route custom events with Azure Event Grid](custom-event-quickstart.md).
