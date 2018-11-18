---
title: "Discovering Rails' main_app"
date: 2016-06-17T16:59:16-08:00
showDate: true
draft: false
tags: ["rails"]
---

My third rails engine added to OppSites brought up some interesting
challenges. The first two I installed, from [Instacart's Open Source](https://www.instacart.com/opensource)
repository, were completely self contained in their views. This third one,
[user_impersonate2](https://github.com/rcook/user_impersonate2), was my
first experience with an engine that embedded itself within my own layouts.

While that behavior was necessary and useful, it was surprising when
I started getting errors saying previously working path helpers were
undefined.

It turns out, there is some sort of `main_app` that can be appended to a
path helper to tell it to use the helper in the context of the "main app"
instead of the engine. I'm still unclear as to where `main_app` is defined
and how it works exactly, but it does work.

> If you were to leave off the main_app "routing proxy" method call,
> it could potentially go to the engine's or application's root, depending on where it was called from.

Ah, so it seems that `main_app` forces the use application paths rather
than the paths from the current context.

Cool.
