# YouTube Creator Outreach Pipeline

> A three-script system for finding, qualifying, and reaching YouTube creators at scale without buying contact lists.

**Status:** Complete · In active use  
**Impact:** 10x+ increase in daily email sending capacity; eliminated dependency on third-party prospecting tools

---

## The Problem It Solves

Outreach software sells you lists. Lists are stale, over-targeted, and shared across hundreds of other agencies doing the same thing. The alternative: manually finding creators, checking their upload frequency, estimating their engagement, and pulling contact info doesn't scale past a few channels a day.

This pipeline automates the entire discovery-to-send workflow using the YouTube Data API and Gmail's SMTP/IMAP stack directly. No third-party dependencies. No recurring prospecting fees. Full control over lead quality filters.

---

## How It Works

Three scripts, run in sequence:

### 1. `yt_leads_finder.py` — Lead Discovery & Qualification

Takes a keyword and a set of quality filters as input, then:

- Searches YouTube for channels posting in that niche (paginated, up to 150 results)
- Batch-fetches channel metadata in groups of 50 to stay within API quota limits
- Filters by subscriber range, upload frequency (last 30 days), and minimum average views
- Scores each qualifying channel using a weighted formula: upload velocity, subscriber size (log-scaled), and average viewership
- Exports a ranked CSV of qualified leads

**Filters available at runtime:**
- Keyword / niche
- Subscriber floor and ceiling
- Minimum uploads in last 30 days
- Minimum average views (optional)

The scoring model weights upload consistency most heavily, a creator posting 12 videos a month is a stronger prospect than one with more subscribers but irregular output.

**Output:** `yt_leads_{keyword}.csv`

---

### 2. `outreach.py` — Personalised Email Sending

Takes the qualified leads CSV (or any Excel sheet with the right columns) and sends personalised cold emails via Gmail SMTP directly: no Mailchimp, no SendGrid, no third-party tools.

For each lead, before sending, the script:

- Resolves the channel ID from whatever URL format is present (direct `/channel/`, `@handle`, or name-based fallback)
- Pulls the most-viewed video from the last 30 days via the YouTube API (falls back to all-time most viewed)
- Cleans the video title: strips hashtags, decodes HTML entities, normalises whitespace
- Injects the video title and upload count into the email template dynamically

Each email references something specific to that creator's recent output, not a generic opener. Subject lines are randomised across a small pool to avoid pattern detection.

**Sending behaviour:**
- Skips contacts already marked `Sent` or `Replied` in the sheet
- Randomises send delay between 40–120 seconds per email to avoid triggering spam filters
- Updates the Excel sheet with `Sent` status after each successful send
- Hard cap of 30 emails per run (configurable)

---

### 3. `reply_tracker.py` — Inbox Monitoring & Status Sync

Connects to Gmail via IMAP and checks every contact in the sheet:

- Scans the inbox for any inbound message from that email address → marks `Replied`
- Scans Sent Mail to confirm delivery for contacts not yet marked → marks `Sent`
- Updates the Excel sheet in place

Run this before starting a new outreach session to get an accurate picture of who has already responded.

---

## Architecture

```
yt_leads_finder.py        →    leads CSV
      ↓
outreach.py               →    sends emails, updates sheet
      ↓
reply_tracker.py          →    syncs replies back to sheet
```

All state lives in the Excel file. The pipeline is stateless between runs — you can stop, restart, or hand the sheet to someone else without losing progress.

---

## Stack

```
Python · YouTube Data API v3 · Gmail SMTP/IMAP
pandas · requests · smtplib · imaplib · dotenv
```

Credentials and API keys are loaded from a `.env` file and never hardcoded.

---

## Why This Exists

Most outreach tools solve the sending problem. This solves the discovery problem first — then connects it directly to sending — without a third-party in the middle marking up the data.

The result is a fully owned pipeline: control over targeting criteria, personalisation logic, send pacing, and reply tracking. Running costs are zero beyond the YouTube Data API quota (which is generous for this use case).
