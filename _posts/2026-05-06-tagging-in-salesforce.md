---
title: "Tagging in Salesforce: A Considered Alternative"
date: 2026-05-06
layout: post
tags: [salesforce, lwc, architecture, data-model]
---
*One of the perennial requests that arise in requirements is the ability to "tag" records in Salesforce. The reason for the request may be familiarity ("our previous CRM let us tag records. Why does Salesforce not support that?") or function ("We need a way to tag records in order to...") and it's important to figure out where the client is coming from, as well as the likely development of their requirements in the future.*

There's a free Salesforce Labs app called [Lightning Universal Tagging](https://appexchange.salesforce.com/appxListingDetail?listingId=a0N3A00000FR4OlUAL). If you need tags now and don't expect them to develop into something more, install it and stop reading. If you expect them to grow up; to be more like tags in something like GoHighLevel, to be reported on, automated against, governed, secured... here's an alternative.

<!-- SCREENSHOT: side-by-side of LUT tags on an Account vs. SmartTags on an Account, showing the visual difference — coloured pills with descriptions vs. plain text -->
<!--![LUT Tags](/assets/images/lut-example.png) ![Smart Tags](/assets/images/smart-tags-example.png) -->
<figure style="display: flex; gap: 1rem; align-items: flex-start;">
  <div style="flex: 1;">
    <img src="/assets/images/lut-example.png" alt="Lightning Universal Tagging applied to an Account">
    <figcaption><em>Lightning Universal Tagging: plain text labels.</em></figcaption>
  </div>
  <div style="flex: 1;">
    <img src="/assets/images/smart-tags-example.png" alt="SmartTags applied to an Account">
    <figcaption><em>SmartTags: coloured pills with descriptions.</em></figcaption>
  </div>
</figure>

## Why I built one anyway

Tagging requests come up a lot. For years I pushed back, because what users mean by "we need tags" is usually one of three things that Salesforce already does well: picklists, campaigns, or topics. But every so often the requirement is genuinely tag-shaped: flexible, user-curated, cross-object, low-ceremony, and none of those quite fit.

Lightning Universal Tagging is a perfectly reasonable solution to that genuine requirement, and it has a real advantage I want to acknowledge first: **zero schema work to extend.** Drop the component on any object's page, done. That's a meaningful benefit if your need is "let users tag anything they encounter."

What I kept running into, though, was the next set of questions. *Can we report on all Accounts tagged VIP? Can we trigger a flow when a Contact gets tagged? Can we restrict who sees which tags? Can we count how many records carry each tag?Can I use tags to build an audience?* Each of these is *possible* on top of LUT's architecture, but each requires work, sometimes quite substantial, because the underlying data model has to fake polymorphism that Salesforce doesn't support natively for custom lookups.

So I built SmartTags around a different trade-off.

## The architectural choice

SmartTags uses one junction object per taggable object: `Account_Tag__c`, `Contact_Tag__c`, `Project_Tag__c` — each with a primary master-detail relationship to its parent, and another to the central `Tag__c` record. The same LWC works on any of them; you tell it which junction object to use via a configuration property on the App Builder page.

<!-- SCREENSHOT: App Builder configuration showing the Junction Object API Name property -->
![Smart Tags Config in App Builder](/assets/images/smart-tag-config.png)

This does carry a cost: each new taggable object needs its own custom joining object and some custom fields. This does require some setup, well within the capabilities of a Salesforce Admin. In return, you get the things that follow naturally from a properly relational model:

- **Native reporting.** "All Accounts tagged VIP" is a standard report on a standard report type. No formula fields, no manual joins, no denormalised name fields to maintain.
- **Real referential integrity.** Delete an Account, its Account_Tags cascade-delete with it. No orphan records pointing at dead IDs.
- **Sharing inheritance.** Master-detail means each tagging inherits visibility from its parent record automatically. A user who can't see the Account can't see its tags. No special handling required.
- **First-class automation surface.** Per-object validation rules, flows, and triggers on each junction. Tag a Contact, fire a flow that's specific to Contact-tagging.
- **Roll-up summary fields.** Want to know how many Accounts carry each tag? It's a roll-up. No Apex required.

<!-- SCREENSHOT: a report showing Accounts grouped by Tag, with counts -->
![Smart Tags List View](/assets/images/tags-list-view.png)

Tags themselves carry a colour and an optional description, edited inline:

<!-- SCREENSHOT: Tag Settings modal with the colour palette and description field -->
![Smart Tags editing](/assets/images/smart-tag-modal.png)

The colour and description aren't the headline; the architecture is. But they're a quality-of-life upgrade that makes tags more scannable in list views and more self-documenting over time.

## When to choose which

Choose LUT when the requirement really is "let users tag anything," when the tags are primarily a user-facing aid rather than data you'll act on programmatically, and when you don't want schema work every time the scope expands.

Choose SmartTags, or something like it (or build your own solution, if you fancy a quick challenge. I'm in no position to judge...) when tags are going to become first-class data: reported on, automated against, governed, secured. When the list of taggable objects is considered rather than open-ended. When the operational cost of orphan records, broken sharing, or report gymnastics outweighs the cost of a small per-object deployment.

Not better. Differently good.

## The repo

[GitHub link goes here]. MIT licensed. Includes the Tag and junction object definitions, the controller and test class, the LWC, and an installation guide. Pull requests welcome.

---

*Next post in this series: how to use a Tag as the launchpad for a Campaign — adding all VIP-tagged Contacts to a Campaign in two clicks.*
