# 03 Data Model

## Modeling Notes
- This is a minimal logical model for MVP behavior.
- All entities include:
  - `id`
  - `created_at`
  - `updated_at`
- Foreign keys are listed as `<entity>_id`.

## Identity & Onboarding

### User
- Key fields: `id`, `email`, `auth_provider`, `status`, timestamps.
- Relationships:
  - One-to-one with `Profile`.
  - One-to-one with `Preferences`.
  - One-to-many with `Goal`.
  - One-to-many with `GroupMember`.

### Profile
- Key fields: `id`, `user_id`, `display_name`, `timezone`, `bio_short`, timestamps.
- Relationships:
  - Belongs to `User`.

### Preferences
- Key fields: `id`, `user_id`, `notification_level`, `quiet_hours_start`, `quiet_hours_end`, `privacy_level`, timestamps.
- Relationships:
  - Belongs to `User`.

### Goal
- Key fields: `id`, `user_id`, `category`, `title`, `active`, timestamps.
- Relationships:
  - Belongs to `User`.
  - One-to-many with `Commitment`.

### AvailabilityWindow
- Key fields: `id`, `user_id`, `day_of_week`, `start_time`, `end_time`, `timezone`, timestamps.
- Relationships:
  - Belongs to `User`.

## Group & Membership

### Group
- Key fields: `id`, `name`, `status`, `formation_version`, timestamps.
- Relationships:
  - One-to-many with `GroupMember`.
  - One-to-one (active) with `GroupGoal`.
  - One-to-many with `GroupFeedItem`.
  - One-to-many with `WeeklySummary`.
  - One-to-many with `MeetupProposal`.
  - One-to-many with `MeetupEvent`.

### GroupMember
- Key fields: `id`, `group_id`, `user_id`, `role`, `joined_at`, `status`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - Belongs to `User`.
  - One-to-many with `Commitment`.
  - One-to-many with `CheckIn`.
  - One-to-one with `StreakState`.

### GroupGoal
- Key fields: `id`, `group_id`, `statement`, `week_start_date`, `week_end_date`, `created_by_user_id`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - References creator `User`.

## Commitment & Progress

### Commitment
- Key fields: `id`, `group_id`, `user_id`, `goal_id`, `title`, `target_count`, `week_start_date`, `week_end_date`, `status`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - Belongs to `User`.
  - Belongs to `Goal`.
  - One-to-many with `CheckIn`.
  - One-to-many with `Recommendation` (historical linkage).

### CheckIn
- Key fields: `id`, `group_id`, `user_id`, `commitment_id`, `date`, `status`, `note`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - Belongs to `User`.
  - Belongs to `Commitment`.
  - Optional one-to-one with `Reflection`.
  - Optional one-to-many with `Evidence`.
  - One-to-one projection to `GroupFeedItem` (type: check-in).

### StreakState
- Key fields: `id`, `group_id`, `user_id`, `current_streak_days`, `longest_streak_days`, `last_check_in_date`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - Belongs to `User`.
  - One-to-one with `GroupMember`.

### Reflection
- Key fields: `id`, `check_in_id`, `user_id`, `text`, timestamps.
- Relationships:
  - Belongs to `CheckIn`.
  - Belongs to `User`.

### Evidence
- Key fields: `id`, `check_in_id`, `user_id`, `evidence_type`, `uri_or_text`, timestamps.
- Relationships:
  - Belongs to `CheckIn`.
  - Belongs to `User`.

## Social Layer

### GroupFeedItem
- Key fields: `id`, `group_id`, `actor_user_id`, `item_type`, `source_entity_id`, `source_entity_type`, `occurred_at`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - References actor `User`.
  - One-to-many with `Comment`.
  - One-to-many with `Reaction`.

### Comment
- Key fields: `id`, `feed_item_id`, `user_id`, `body`, timestamps.
- Relationships:
  - Belongs to `GroupFeedItem`.
  - Belongs to `User`.

### Reaction
- Key fields: `id`, `feed_item_id`, `user_id`, `reaction_type`, timestamps.
- Relationships:
  - Belongs to `GroupFeedItem`.
  - Belongs to `User`.

### Nudge
- Key fields: `id`, `group_id`, `from_user_id`, `to_user_id`, `context_feed_item_id`, `message`, `status`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - References sender `User`.
  - References recipient `User`.
  - Optional reference to `GroupFeedItem`.

## Weekly Intelligence

### WeeklySummary
- Key fields: `id`, `group_id`, `week_start_date`, `week_end_date`, `summary_status`, `generated_at`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - One-to-many with `Insight`.
  - One-to-many with `Recommendation`.

### Insight
- Key fields: `id`, `weekly_summary_id`, `scope` (user/group), `user_id` (nullable), `insight_type`, `content`, timestamps.
- Relationships:
  - Belongs to `WeeklySummary`.
  - Optional reference to `User`.

### Recommendation
- Key fields: `id`, `weekly_summary_id`, `user_id`, `recommendation_type`, `content`, `accepted_at` (nullable), timestamps.
- Relationships:
  - Belongs to `WeeklySummary`.
  - Belongs to `User`.
  - Optional reference to `Commitment` created from recommendation.

## Meetup (Optional)

### MeetupProposal
- Key fields: `id`, `group_id`, `proposed_by_user_id`, `title`, `proposed_start`, `duration_minutes`, `location_type`, `location_value`, `status`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - References proposer `User`.
  - Can create one `MeetupEvent`.

### MeetupEvent
- Key fields: `id`, `group_id`, `meetup_proposal_id`, `start_at`, `end_at`, `location_type`, `location_value`, `status`, timestamps.
- Relationships:
  - Belongs to `Group`.
  - Optional belongs to `MeetupProposal`.
  - Attendance can be represented by extension entity in later scope.
