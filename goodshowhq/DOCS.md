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
4. Install the GoodShowHQ mobile app to review suggestions — the
   add-on takes care of the rest.

## Options

| Option | Description |
| --- | --- |
| `graphql_endpoint` | GoodShowHQ API endpoint. Leave the default unless support asks you to change it. |

## FAQ

**Does my kid's password go anywhere?** No. You type it into
Google's own sign-in page inside a real browser running on your
machine. GoodShowHQ never sees the password; the add-on keeps the
resulting YouTube session locally.

**What does the add-on send to GoodShowHQ?** Feed and watch-history
metadata for scoring, and confirmations when an approved action
(block/follow) is applied. No credentials.

**The account shows "Needs sign-in".** YouTube rotated the session.
Click **Sign in to YouTube** again — with the saved browser profile
this usually completes without re-entering the password.
