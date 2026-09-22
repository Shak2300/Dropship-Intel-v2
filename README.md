# Dropship-Intel-v2

A production automation platform that runs a live eBay dropshipping
business end to end: product research, listing management, and
automated order fulfillment.

> Source code is private because the platform operates a live
> commercial business. This repo documents the architecture and
> engineering decisions. Happy to walk through the system in detail.

## Problem

Running a dropshipping operation by hand means constant manual work:
researching products, creating and updating listings, matching
incoming orders to the right supplier SKU and variation, placing
supplier orders, and catching supplier price changes before they
turn a sale into a loss. Each step is repetitive, error-prone, and
scales badly.

## What It Does

Dropship Intel automates the operational loop so the business runs
on exceptions rather than on manual effort.

- **Product research and sourcing intelligence** — evaluates candidate
  products against margin, competition, and fee structure so only
  viable items move to listing.
- **Listing management** — creates and maintains eBay listings through
  the eBay API, keeping pricing and availability in sync with
  suppliers.
- **Automated order fulfillment** — ingests incoming orders, maps each
  order line to the correct supplier SKU and variation, and drives
  the supplier order flow through a Fulfill UI that groups related
  work.
- **Margin protection** — a price-drift sanity cap flags or blocks
  fulfillment when a supplier's price has moved beyond a safe
  threshold, preventing silent losses.

## Architecture

- **Application layer** — Python / Flask backend with a browser-based
  operations UI (`gui.html`) for research, listing, and fulfillment
  workflows.
- **Integration layer** — eBay API client (`ebay.py`) for listing
  creation, updates, and order retrieval; supplier-side sourcing
  logic for product data and pricing.
- **Fulfillment engine** (`fulfillment.py`) — variation-to-SKU
  mapping, multi-variation order handling, and the price-drift
  sanity cap.
- **Data layer** — relational database with versioned schema
  migrations; a migration was required when multi-variation order
  support was added.
- **Remote access** — Tailscale provides secure mobile access to the
  locally hosted app without exposing it to the public internet.

## Engineering Notes

- **Multi-variation orders** were the hardest correctness problem:
  an order containing several variations of one listing has to map
  each line to a distinct supplier SKU. Solving it required a schema
  migration and a dedicated variation-to-SKU mapping feature.
- **Fee-aware margins.** eBay's mandatory promotion fee plus standard
  fees total roughly 24%, so product viability is evaluated against
  net margin after fees, not list price.
- **Fail loudly on price drift.** The sanity cap exists because the
  most expensive failure in dropshipping is a fulfillment that
  quietly executes at a loss.
- **AI-assisted development.** Built and iterated using an agentic
  development workflow (Claude Code) with session recovery and
  primer documentation so interrupted sessions resume cleanly.

## Tech

Python, Flask, eBay API, relational database with schema migrations,
HTML/JavaScript operations UI, Tailscale.

## Status

In production and actively used to run the business. Active product
lines include consumer electronics and contractor/DIY tool
categories, with new categories validated in small test batches
before scaling.

