---
name: RIPE Atlas
description: Use the RIPE Atlas REST API v2 to find probes, inspect measurements, create and monitor network measurements, and fetch results. Use when the user asks about Atlas probes, measurements, ping, traceroute, DNS, HTTP, TLS, NTP, or globally distributed reachability checks.
---

# RIPE Atlas

Run active measurements from globally distributed RIPE Atlas probes
Treat the local machine as only the API client, never as a probe vantage point

## Prerequisites

Read the API key from `RIPE_ATLAS_API_KEY`
Use base URL `https://atlas.ripe.net/api/v2/`
Send the key on authenticated requests as `Authorization: Key $RIPE_ATLAS_API_KEY`
Never print, log, commit, or persist the key
Confirm the exact `bill_to` email before charging another account
Consult the official reference at `https://atlas.ripe.net/docs/apis/rest-api-reference`
Consult the OpenAPI schema at `https://atlas.ripe.net/api/v2/openapi-v3/`

## Safety rules

Confirm type, target, address family, one-off or recurring schedule, probe count and selection, visibility, and billing account before creating any measurement
Confirm the measurement ID before stopping it, stopping uses `DELETE` and halts execution
Start with a small one-off measurement before launching a recurring one
Check credits with an authenticated request before any large or long-running campaign

## Responses and pagination

Read list responses through `count`, `next`, `previous`, and `results`
Follow `next` whenever the full result set is needed
Prefer `page_size` with narrow filters over unbounded downloads
Request only needed fields with `fields` on lists and `optional_fields` on measurement detail

## Find existing measurements

```sh
curl -sS -G 'https://atlas.ripe.net/api/v2/measurements/' \
  --data-urlencode 'type=ping' \
  --data-urlencode 'status=Ongoing' \
  --data-urlencode 'description__contains=example' \
  --data-urlencode 'page_size=20'
```

Retrieve one measurement with scheduling and probe state:

```sh
curl -sS \
  -H "Authorization: Key $RIPE_ATLAS_API_KEY" \
  'https://atlas.ripe.net/api/v2/measurements/12345678/?optional_fields=status,probes,probe_sources,current_probes'
```

## Find probes

Treat `1` as connected and `2` as disconnected in the `status` filter
Filter by country, ASN, public visibility, and tags to scope candidate probes

```sh
curl -sS -G 'https://atlas.ripe.net/api/v2/probes/' \
  --data-urlencode 'status=1' \
  --data-urlencode 'country_code=JP' \
  --data-urlencode 'asn_v4=2914' \
  --data-urlencode 'is_public=true' \
  --data-urlencode 'page_size=20'
```

Search probes by description, ID, or ASN:

```sh
curl -sS -G 'https://atlas.ripe.net/api/v2/probes/' \
  --data-urlencode 'search=example' \
  --data-urlencode 'page_size=20'
```

## Create a measurement

POST JSON with required `definitions` and `probes` arrays to `https://atlas.ripe.net/api/v2/measurements/`
Use one of `ping`, `traceroute`, `dns`, `sslcert`, `http`, `ntp` as the definition type
Select probes with `area`, `country`, `countries`, `probes`, `asn`, `prefix`, `msm`, or `region`
Set `is_oneoff` both on the definition and the top-level request for single-shot runs
Send the payload from a temporary file and keep the key out of the file

```json
{
  "definitions": [
    {
      "description": "JP probe connectivity check",
      "type": "ping",
      "af": 4,
      "target": "203.0.113.1",
      "packets": 3,
      "size": 48,
      "is_oneoff": true
    }
  ],
  "probes": [
    {
      "requested": 5,
      "type": "country",
      "value": "JP"
    }
  ],
  "is_oneoff": true
}
```

```sh
curl -sS -X POST 'https://atlas.ripe.net/api/v2/measurements/' \
  -H "Authorization: Key $RIPE_ATLAS_API_KEY" \
  -H 'Content-Type: application/json' \
  --data @/tmp/opencode/ripe-atlas-measurement.json
```

Read created IDs from the `measurements` array in the `201` response:

```json
{
  "measurements": [12345678]
}
```

## Monitor and fetch results

Poll status and scheduling state before assuming results exist:

```sh
curl -sS \
  -H "Authorization: Key $RIPE_ATLAS_API_KEY" \
  'https://atlas.ripe.net/api/v2/measurements/12345678/?optional_fields=status,current_probes,probes'
```

Fetch the latest results:

```sh
curl -sS \
  -H "Authorization: Key $RIPE_ATLAS_API_KEY" \
  'https://atlas.ripe.net/api/v2/measurements/12345678/latest/'
```

Fetch filtered results by probe and time window:

```sh
curl -sS -G \
  -H "Authorization: Key $RIPE_ATLAS_API_KEY" \
  'https://atlas.ripe.net/api/v2/measurements/12345678/results/' \
  --data-urlencode 'format=json' \
  --data-urlencode 'probe_ids=1,2,3' \
  --data-urlencode 'start=2026-09-24T00:00:00' \
  --data-urlencode 'stop=2026-09-24T01:00:00'
```

## Stop a measurement

```sh
curl -sS -X DELETE \
  -H "Authorization: Key $RIPE_ATLAS_API_KEY" \
  'https://atlas.ripe.net/api/v2/measurements/12345678/'
```
