# GoodShowHQ

Keep your kid's YouTube healthy. This add-on replaces the desktop
app for Home Assistant households: sign in to your kid's YouTube
account once, right here in Home Assistant, and the add-on keeps
watching their feed and history and applies the actions you approve
in the GoodShowHQ mobile app — 24/7, from your own box.

**Local-first:** your kid's YouTube session (the sign-in cookies)
lives only in this add-on's private storage on your Home Assistant
machine. Only content scores and the actions you approve go through
the GoodShowHQ cloud.

## Setup

1. Start the add-on and open its panel from the sidebar.
2. Click **Sign in to YouTube** and log in with your **kid's**
   Google account in the embedded browser. (Passkey-only accounts:
   use password + code, or approve from another device.)
3. When the account shows **Connected**, click **Link to your
   GoodShowHQ account →** to attach the profile to your family
   account.
4. More than one kid? Click **Add another YouTube account** and
   repeat — each account gets its own row with its avatar, @handle,
   and profile ID, exactly like the desktop app.
5. Install the GoodShowHQ mobile app to review suggestions — the
   add-on takes care of the rest.

## Options

| Option | Description |
| --- | --- |
| `graphql_endpoint` | GoodShowHQ API endpoint. Leave the default unless support asks you to change it. |

## Home Assistant entities

The add-on publishes sensors you can use in dashboards and
automations. Per linked account (entity ids use the account's name,
e.g. `noor_k`):

| Entity | Meaning |
| --- | --- |
| `sensor.goodshowhq_<kid>_last_watched` | Title of the most recently watched video. Attributes: `channel`, `video_id`, `url`, `watched_at`, `entity_picture` (thumbnail), and `recent` (the last 5 watches). Sourced from GoodShowHQ's analyzed watch history and refreshed every few minutes. |
| `sensor.goodshowhq_<kid>_videos_today` | Videos watched since midnight (UTC). Attribute `videos_7d` carries the 7-day count. |
| `sensor.goodshowhq_<kid>_flagged_week` | Watches from the last 7 days rated harmful (your own "Not for…" votes included). |
| `sensor.goodshowhq_<kid>_pending_decisions` | Items waiting in the app's Decisions deck. |
| `sensor.goodshowhq_<kid>_history_health` | % of scored watches (7 days) rated beneficial. `unknown` until something is scored. |
| `sensor.goodshowhq_<kid>_feed_health` | Same ratio over the current feed. |
| `sensor.goodshowhq_<kid>_top_channel_week` | Most-watched channel of the last 7 days. Attribute `watch_count_7d`. |
| `binary_sensor.goodshowhq_<kid>_connected` | `on` while the YouTube session is healthy. Attributes: `profile_id`, `email`, `channel_handle`, `last_error`. |

The stats sensors refresh every 30 minutes; `last_watched` every
5 minutes. All of it comes from GoodShowHQ's analyzed data — the
add-on never computes content stats locally.

Add-on wide:

| Entity | Meaning |
| --- | --- |
| `binary_sensor.goodshowhq_worker_running` | Worker heartbeat. Attributes: cycle counters and last error. |
| `binary_sensor.goodshowhq_signin_needed` | `on` when any account needs a re-sign-in (or a sign-in attempt failed). Attribute `accounts_needing_signin` lists them — alert on this one entity instead of each kid's connectivity sensor. |
| `sensor.goodshowhq_pending_tasks` | Tasks currently queued for this box. Attribute `by_type` breaks the count down (`QUERY_HISTORY`, `TRANSCRIBE`, …). |
| `sensor.goodshowhq_tasks_completed_today` | Tasks this box completed since midnight (UTC), with a `by_type` breakdown. |
| `sensor.goodshowhq_transcriptions_today` | Videos this box transcribed today (subset of the above, split out for dashboards). |

Note: watch timestamps follow YouTube's history buckets, so
`watched_at` can be day-precision. The `last_watched` state changing
is the reliable "they watched something new" trigger.

Example automation:

```yaml
automation:
  - alias: "Ping me when Noor watches something new"
    trigger:
      - platform: state
        entity_id: sensor.goodshowhq_noor_k_last_watched
    condition:
      - "{{ trigger.from_state is not none
            and trigger.to_state.state not in ['unknown', 'unavailable']
            and trigger.to_state.state != trigger.from_state.state }}"
    action:
      - service: notify.mobile_app_your_phone
        data:
          message: >-
            Noor watched: {{ trigger.to_state.state }}
            ({{ state_attr('sensor.goodshowhq_noor_k_last_watched', 'channel') }})
```

## FAQ

**Does my kid's password go anywhere?** No. You type it into
Google's own sign-in page inside a real browser running on your
machine. GoodShowHQ never sees the password; the add-on keeps the
resulting YouTube session locally.

**What does the add-on send to GoodShowHQ?** Feed and watch-history
metadata for scoring, and confirmations when an approved action
(block/follow) is applied. No credentials.

**The account shows "Needs sign-in".** YouTube rotated the session.
Sign in with that account again — the add-on recognizes the account
and updates it in place (it won't create a duplicate).

**Removing an account.** Click **Remove** on its row (twice, to
confirm). This only forgets the sign-in on your box — the profile
and its history in your GoodShowHQ account are untouched, and
signing in again reconnects the same profile.
