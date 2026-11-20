# Customer-card traversal: live capture fix

## Confirmed cause in the original ZIP

`DistributorBlueprint.java:113–121` searched each node for the combined text `Delivery Point (Dist)` or `Delivery Point (Retailer)`. The supplied live capture describes those labels as separate nodes, so no anchor was found. `SudhaAccessibilityService.java:2026–2103` then saw an expanded list but no parsed cards and stopped. The original split-ID extraction also imposed a geometric same-line condition; the specified card layout gives an ordered node sequence instead.

The repository-level workflow is `.github/workflows/main.yml`, which extracts the ZIP before testing and building. `.github/workflows/build-apk.yml` exists **inside** the ZIP, not at repository root.

## Changes

* Parse visible `cv_parent_view` blocks inside the exact `customer_list` node. Read separate delivery/type/ID/name nodes in order and click the card itself; retain a single-card-ancestor fallback for older layouts. A missing or unreadable block is counted and is not silently skipped.
* Use the exact count, filter and Show Retailers view IDs where present. Verify the All Distributors selection; select it only from a verified list popup, otherwise stop for manual correction. After one Show Retailers tap, wait instead of toggling again blindly.
* Classify by type and normalized name, not ID prefix. Retain observed shadow-card identity while it crosses the viewport boundary. Require overlapping card sequences before advancing. Defer the bottom retailer until its successor or the end has been observed, so a distributor immediately below it is not missed.
* Disable blind ScrollMemory jumps during customer traversal, and use bounded drag scrolling instead of an unbounded accessibility page jump. Saved history is retained. Do not navigate Back to Home to restart a list sweep. Resume still uses the existing arbitrary-ID StartPoint and RetailerSequence paths.
* Add synthetic trees for the reported live sibling layout, arbitrary IDs, exact card targets, split-label fallback, adjacency in both directions, overlapping boundaries, ambiguous/non-overlapping pages, count and filter IDs.

## Safety and verification limits

A page transition without a verifiable overlap, an unobserved predecessor at a newly entered mid-list position, a missing name, or a retailer ID seen on differently named shops stops for review rather than guessing. The ledger and last-done memory are still keyed by ID, so duplicate IDs cannot be generally disambiguated across separate visits; similarly, the typed whitespace alias is resolved from the current page, not proven globally unique. Start from the top of the expanded list when the preceding card is unknown.

The self-tests use synthetic accessibility nodes. They do not prove the installed Android app's actual tree, scroll physics, order/payment outcomes, alarm behavior, or real-device end-to-end operation. Verify with a freshly built APK and a live capture before an unattended run.
