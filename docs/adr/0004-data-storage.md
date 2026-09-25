# ADR-0004: Embedded Gateway Database Selection (SQLite WAL Mode)

## Context
The Edge Gateway requires persistent, structured, low-latency local storage for event logs, device health metrics, and active alerts on Raspberry Pi hardware.

## Problem
Which database system should be selected for local edge storage?

## Considered Options
1. **Full RDBMS (PostgreSQL / MySQL):** Heavyweight server database.
2. **Embedded Time-Series (SQLite in WAL Mode):** Zero-configuration single-file embedded relational store with Write-Ahead Logging.

## Decision
We select **Option 2: SQLite in WAL Mode**.

## Why
* **Resource Footprint:** Requires $<5\text{MB}$ RAM compared to $>200\text{MB}$ for PostgreSQL.
* **Maintenance:** Zero database administration or service setup required; robust ACID transactions on flash storage.

## Status
`[DECISION]` Accepted & Approved.
