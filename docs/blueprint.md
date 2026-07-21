# StreakMate — Bot specification

**Archetype:** workflow

**Voice:** encouraging and warm — write every user-facing message, button label, error, and empty state in this voice.

StreakMate is a private habit-tracking Telegram bot that sends timezone-aware reminders, allows one-tap habit completion, tracks streaks and statistics, and provides encouraging weekly summaries. It focuses on gentle accountability through milestone celebrations and manual adjustments while maintaining strict user data privacy.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individuals seeking to build habits
- Users needing private tracking
- People preferring minimal social features

## Success criteria

- Users can create and manage habits with one-tap interactions
- Reminders are sent at correct local times across timezones
- Streak calculations update accurately with manual history corrections
- Weekly summaries show progress with encouraging feedback

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu with habit overview
- **Create New Habit** (button, actor: user, callback: habit:create) — Start guided habit creation flow
  - inputs: habit name, schedule type, reminder time
  - outputs: habit created confirmation
- **Weekly Summary** (button, actor: user, callback: summary:weekly) — Request compact weekly progress report
  - outputs: summary with streaks and stats
- **Mark done** (button, actor: user, callback: occurrence:mark_done) — Record habit completion for current day
  - outputs: streak update confirmation
- **Pause habit** (button, actor: user, callback: habit:pause) — Temporarily disable a habit
  - outputs: habit status change confirmation

## Flows

### Onboarding
_Trigger:_ /start

1. Welcome message
2. Timezone selection (auto-detect default)
3. Guided habit creation prompts

_Data touched:_ User

### Habit Creation
_Trigger:_ habit:create

1. Request habit name
2. Schedule type selection (daily/weekdays/N times/week)
3. Reminder time selection with quick options
4. Milestone celebration preferences

_Data touched:_ Habit

### Daily Reminder
_Trigger:_ scheduled_reminder

1. Send habit reminder message
2. Display 'Mark done' and other action buttons
3. Disable 'Mark done' button after first use

_Data touched:_ Occurrence, Habit

### Manual Adjustment
_Trigger:_ habit:view

1. Show habit details
2. Edit schedule/time/name
3. Mark specific dates as done/skipped/failed
4. Pause/unpause/delete habit

_Data touched:_ Habit, Occurrence

### Milestone Celebration
_Trigger:_ streak_milestone_reached

1. Detect milestone (7,14,30 days or 50%/75% completion)
2. Send subtle celebration message if enabled

_Data touched:_ Streaks & stats

### Weekly Summary
_Trigger:_ summary:weekly

1. Aggregate habit data for week
2. Format compact summary with stats
3. Add encouraging comment

_Data touched:_ Streaks & stats, Occurrence

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram account metadata and preferences
  - fields: telegram_id, locale, timezone, preferences
- **Habit** _(retention: persistent)_ — User-defined habit with scheduling rules
  - fields: title, description, schedule_type, reminder_time, status, creation_date, milestone_settings, timezone_locked
- **Occurrence** _(retention: persistent)_ — Daily habit completion record
  - fields: date (local), status, timestamp, source, occurrence_id
- **Streaks & stats** _(retention: session)_ — Derived metrics from occurrences
  - fields: current_streak, longest_streak, completion_percentage

## Integrations

- **Telegram** (required) — Bot API messaging and timezone detection
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure milestone celebration thresholds
- Enable/disable automated weekly summaries
- Set default schedule types

## Notifications

- Scheduled habit reminders
- Milestone celebrations (7,14,30 days or 50%/75% completion)
- Weekly summary reports

## Permissions & privacy

- All habit data stored privately per Telegram account
- No cross-user data sharing or public feeds
- Occurrences track timestamps and sources to prevent double-marking

## Edge cases

- Timezone change after habit creation
- Double-marking prevention via occurrence_id
- Paused habits during active streaks
- Failed status auto-applied at end of local day

## Required tests

- End-to-end onboarding flow with timezone handling
- One-tap marking prevents duplicate entries
- Streak calculations after history edits
- Weekly summary formatting with real user data

## Assumptions

- Users want milestone celebrations to be optional
- Three schedule types cover most use cases
- Telegram account ID is sufficient for user identification
