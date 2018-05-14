---
title: Sample for Translation | Microsoft Docs
titleSuffix: "Microsoft Cognitive Services"
description: Here is a sample for speech translation.
services: cognitive-services
author: wolfma61
manager: onano

ms.service: cognitive-services
ms.technology: Speech
ms.topic: article
ms.date: 05/07/2018
ms.author: wolfma
---

# Sample for Translation

> [!NOTE]
> For instructions to download this sample and others, see [Samples for Speech SDK](samples.md).

[!include[Get a Subscription Key](includes/get-subscription-key.md)]

> [!NOTE]
> For all samples below, we assume the following top-level declarations are in place:
>
> [!code-csharp [Using Statements](~/samples-cognitive-services-speech-sdk/Windows/csharp_samples/translation_samples.cs#toplevel)]
>
> - - -

## Translation Using Microphone

The code snippet below shows how to translate speech input from English to German, and also get the voice output of the translated text. It uses the microphone.

[!code-csharp[Translation Using Microphone](~/samples-cognitive-services-speech-sdk/Windows/csharp_samples/translation_samples.cs#TranslationWithMicrophoneAsync)]

- - -

## Translation Using File Input

The code snippet below shows how to translate speech input from English to German and French.
It uses file as input.

[!code-csharp[Translation Using File Input](~/samples-cognitive-services-speech-sdk/Windows/csharp_samples/translation_samples.cs#TranslationWithFileAsync)]

- - -
