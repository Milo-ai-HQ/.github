# ERP Development — Priority · NetSuite · Business Central · Odoo

![Priority](https://img.shields.io/badge/Priority-SQLI%20%C2%B7%20triggers-0B5FA5)
![NetSuite](https://img.shields.io/badge/NetSuite-SuiteScript%202.1-1F6FB5)
![Business Central](https://img.shields.io/badge/Business%20Central-AL-0078D4)
![Odoo](https://img.shields.io/badge/Odoo-Python%20%C2%B7%20ORM-714B67)
![License](https://img.shields.io/badge/license-MIT-green)

**One business scenario, implemented natively on four ERP platforms.** Not four ports of the same
code — each repository uses the extension model the platform actually gives you, the way it would be
built for a customer.

## The scenario

A company sells on credit and takes orders from a web shop. Three requirements:

1. **Open orders by customer** — a report of what is outstanding, and for how much.
2. **A hard credit-limit block** — refuse an order that would push the customer past their limit.
3. **A Shopify order feed** — web-shop orders land in the ERP automatically.

Requirement 2 is the interesting one. Every one of these platforms ships a credit check that only
*warns*, and only in the UI. A block that actually holds has to sit deeper — on the form trigger,
the User Event, the event subscriber, the model constraint — so that the API and the file imports
hit it too.

> A/R balance + other open orders + this order > credit limit → refuse.

## The repositories

| | What it demonstrates |
|---|---|
| [**Priority — Credit Control**](https://github.com/Milo-ai-HQ/priority-credit-control) | `ORDERITEMS` PRE-INSERT trigger + a report procedure. SQLI, the Procedure and Report Generators, custom tables |
| [**Priority — Order Load Interface**](https://github.com/Milo-ai-HQ/priority-order-load-interface) | `GENERALLOAD` + Form Load interface, with a duplicate guard and an error listing for refused rows |
| [**NetSuite — Sales Controls**](https://github.com/Milo-ai-HQ/netsuite-sales-controls) | User Event `beforeSubmit` that holds in every execution context, a Suitelet report with credit headroom, SDF project layout |
| [**Business Central — Sales Controls**](https://github.com/Milo-ai-HQ/business-central-sales-controls) | AL extension: event subscribers, a query and a list page, a permission set — and no base-app changes, so it survives upgrades |
| [**Odoo — Sales Controls**](https://github.com/Milo-ai-HQ/odoo-sales-controls) | Custom module: `@api.constrains`, a SQL-view report model with list and pivot views, `TransactionCase` tests |
| [**Shopify → ERP Order Sync**](https://github.com/Milo-ai-HQ/shopify-erp-order-sync) | One Node.js service, four adapters. HMAC verification, idempotency, retries that distinguish network failures from business rejections |

## The same three requirements, side by side

| | Priority | NetSuite | Business Central | Odoo |
|---|---|---|---|---|
| **Open orders report** | SQLI procedure + Report Generator | Suitelet + SuiteQL | Query + list page | SQL-view model, list + pivot |
| **Credit-limit block** | `ORDERITEMS` PRE-INSERT trigger | User Event `beforeSubmit` | Event subscriber on Sales Line | `@api.constrains` on `sale.order` |
| **Shopify order sync** | REST/OData deep insert | REST upsert by external id | API v2.0 deep insert | JSON-RPC `execute_kw` |

## On testing without sandboxes

Licensed sandboxes for four ERPs are not something you keep lying around. The sync service ships
with `mock-erp-api` — an in-memory stand-in that reproduces each platform's request, response and
error shapes, and enforces the same credit rule with the same message. That makes the rejection
path testable end to end: **20 integration tests, all four adapters, no network.**
