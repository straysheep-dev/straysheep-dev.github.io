---
title: "Data Removal"
icon: material/eye-remove
draft: false
#date:
#  created: 2026-02-21
#  updated: 2026-03-19
categories:
  - privacy
  - opt-out
  - osint
  - breach
  - peoplesearch
---


# :material-eye-remove: Data Removal

This is a comparison of my experience doing it myself vs platforms that provide it as a service.

!!! abstract "Summary"

    Ultimately, you'll hear from privacy and OSINT experts the DIY approach is the most effective. This has been my experience as well.

    All of the relevant guides and resrouces are linked below.

    [resouces.md#osint - IntelTechniques Guides](../resources.md#osint)

Both of the major platforms that specialize in data removal have their pros and cons, but they aren't as good as doing things yourself. DeleteMe maintains a somewhat up-to-date list of sites they opt-out from, Cloaked does not. DeleteMe does not have as wide of coverage as Cloaked, but in some cases may be more effective at completing the removal. The lack of a clear CSV or raw data-driven deliverable is what's frustrating on both sides. The status and catalogue of activity in your dashboard for both is somewhat vague. Cloaked wins here, as you can at least see what's happening from the web interface. DeleteMe appears to require pulling the PDF report and reviewing that, which in my opinion is unnecessary as a deliverable and is bloated.

!!! tip "Learn, then Maintain"

    What I did (and suggest) is doing everything yourself first, then signing up for a service unless you really enjoy doing this maintenance once per quarter. This is purely a time trade-off, and is in no way more or less effective. Having a service maintain your work means it's continuing to take things down that will appear again, and acts as an early alert system if things ever pop back up again.

Alternatively if you are proficient in [selenium](https://github.com/SeleniumHQ/selenium) and have found a way to automate this yourself, that's likely the best case scenario for ongoing maintenance. I'm not sure how realistic that is, or how much of a pet that type of project would turn into, but I think it could be worth exploring just for the learning experience alone.


## DIY

An individual can reasonably achieve this in a month or two, doing 1 a day or set aside an hour a week to do 5-10 at a time. This gives you direct insight into what's out there, how these companies work, and how to assess your own footprint. It's also free, but it will cost you in time. Despite that, it is the most effective option.

<div class="grid cards" markdown>

-   :material-web-plus:{ .lg .middle } __Pros__

    ---

    :material-plus-box: Hands on perspective of your online footprint

    :material-plus-box: No monetary cost

    :material-plus-box: Muscle memory for finding information

    :material-plus-box: Understanding of how data brokers operate

    :material-plus-box: Learn how to deal with dark patterns (artificial limitations)


-   :material-web-minus:{ .lg .middle } __Cons__

    ---

    :material-minus-box: It costs time and requires persistence

    :material-minus-box: Some sites are malicious, use a VM or browser profile

</div>

## DeleteMe

Strong support for major sites where you may have to DIY it anyway compared to Cloaked. Loses out in terms of practical application of proactive privacy tools, cost, and breadth of coverage.

<div class="grid cards" markdown>

-   :material-web-plus:{ .lg .middle } __Pros__

    ---

    :material-plus-box: Easy to enroll and navigate the platform

    :material-plus-box: Excels at removals from some majors sites

    :material-plus-box: Transparency in site coverage

-   :material-web-minus:{ .lg .middle } __Cons__

    ---

    :material-minus-box: No dashboard view or raw data

    :material-minus-box: Reports are mostly bloated, unnecessary

    :material-minus-box: Email, phone, and other masked options are limited

    :material-minus-box: More expensive

</div>


## Cloaked

More robust and active, especially in terms of proactive privacy features. Wins in terms of practical usage and cost.

<div class="grid cards" markdown>

-   :material-web-plus:{ .lg .middle } __Pros__

    ---

    :material-plus-box: Most effective removals (aggressive)

    :material-plus-box: Robust masked options (VOIP, email, virtual cards)

    :material-plus-box: Identity management / password manager

    :material-plus-box: Most cost-effective option

    :material-plus-box: Easy to understand dashboard


-   :material-web-minus:{ .lg .middle } __Cons__

    ---

    :material-minus-box: Missing coverage for a few major sites (may change)

    :material-minus-box: No raw data download

    :material-minus-box: Pushes you to use the mobile app on phones

    :material-minus-box: Does not work on iOS with lockdown enabled

    :material-minus-box: No public list of site coverage

</div>