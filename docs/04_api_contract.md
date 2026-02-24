# 04 API Contract (MVP Draft)

## Conventions
- Base path: `/v1`
- Auth: bearer token for protected endpoints.
- JSON only.
- Timestamps use ISO 8601 UTC.

## Auth

### `POST /v1/auth/signup`
Request:
```json
{
  "email": "user@example.com",
  "password": "string"
}
```
Response:
```json
{
  "user_id": "usr_123",
  "access_token": "jwt_token"
}
```

### `POST /v1/auth/login`
Request:
```json
{
  "email": "user@example.com",
  "password": "string"
}
```
Response:
```json
{
  "user_id": "usr_123",
  "access_token": "jwt_token"
}
```

## Profile & Preferences

### `GET /v1/me`
Response:
```json
{
  "user": {
    "id": "usr_123",
    "email": "user@example.com"
  },
  "profile": {
    "display_name": "Alvin",
    "timezone": "America/Los_Angeles"
  },
  "preferences": {
    "notification_level": "balanced",
    "privacy_level": "group_only"
  }
}
```

### `PATCH /v1/me/profile`
Request:
```json
{
  "display_name": "Alvin",
  "timezone": "America/Los_Angeles",
  "bio_short": "Building consistency"
}
```
Response:
```json
{
  "profile_id": "pro_123",
  "updated_at": "2026-02-16T10:00:00Z"
}
```

### `PATCH /v1/me/preferences`
Request:
```json
{
  "notification_level": "balanced",
  "quiet_hours_start": "22:00",
  "quiet_hours_end": "07:00"
}
```
Response:
```json
{
  "preferences_id": "pref_123",
  "updated_at": "2026-02-16T10:00:00Z"
}
```

## Goals & Availability

### `POST /v1/goals`
Request:
```json
{
  "category": "fitness",
  "title": "Daily 20 minute walk"
}
```
Response:
```json
{
  "goal_id": "goal_123"
}
```

### `PUT /v1/availability`
Request:
```json
{
  "windows": [
    { "day_of_week": "mon", "start_time": "18:00", "end_time": "20:00" },
    { "day_of_week": "wed", "start_time": "18:00", "end_time": "20:00" }
  ]
}
```
Response:
```json
{
  "saved": true
}
```

## Groups

### `POST /v1/matching/enroll`
Request:
```json
{
  "goal_id": "goal_123"
}
```
Response:
```json
{
  "status": "queued"
}
```

### `GET /v1/groups/current`
Response:
```json
{
  "group": {
    "id": "grp_123",
    "name": "Morning Momentum",
    "member_count": 4
  },
  "members": [
    { "user_id": "usr_123", "display_name": "Alvin" }
  ]
}
```

### `POST /v1/groups/{group_id}/goal`
Request:
```json
{
  "statement": "Build a consistent weekday routine",
  "week_start_date": "2026-02-16"
}
```
Response:
```json
{
  "group_goal_id": "ggoal_123"
}
```

## Commitments & Check-Ins

### `POST /v1/groups/{group_id}/commitments`
Request:
```json
{
  "title": "Walk 20 minutes",
  "target_count": 5,
  "week_start_date": "2026-02-16",
  "week_end_date": "2026-02-22"
}
```
Response:
```json
{
  "commitment_id": "com_123"
}
```

### `GET /v1/groups/{group_id}/commitments`
Response:
```json
{
  "items": [
    {
      "id": "com_123",
      "user_id": "usr_123",
      "title": "Walk 20 minutes",
      "progress_count": 2,
      "target_count": 5
    }
  ]
}
```

### `POST /v1/groups/{group_id}/check-ins`
Request:
```json
{
  "commitment_id": "com_123",
  "date": "2026-02-16",
  "status": "done",
  "note": "Completed after lunch",
  "reflection": {
    "text": "Easier when calendar is blocked"
  },
  "evidence": [
    {
      "evidence_type": "text",
      "uri_or_text": "Step count: 6200"
    }
  ]
}
```
Response:
```json
{
  "check_in_id": "chk_123",
  "feed_item_id": "feed_123",
  "streak": {
    "current_streak_days": 4,
    "longest_streak_days": 9,
    "streak_status": "active"
  }
}
```

Event/trigger notes:
- Check-in posted -> update commitment progress counters.
- Check-in posted -> update user streak state.
- Day boundary passed without valid check-in (user timezone) -> mark streak at risk.
- User posts after missed day -> reset current streak and keep longest streak.
- Check-in posted -> create feed item for group timeline.
- Reflection/evidence attached -> include in weekly summary inputs.

## Feed Interactions

### `GET /v1/groups/{group_id}/feed`
Response:
```json
{
  "items": [
    {
      "id": "feed_123",
      "item_type": "check_in",
      "actor_user_id": "usr_123",
      "content": {
        "status": "done",
        "note": "Completed after lunch"
      }
    }
  ]
}
```

### `POST /v1/feed/{feed_item_id}/comments`
Request:
```json
{
  "body": "Nice consistency this week."
}
```
Response:
```json
{
  "comment_id": "cmt_123"
}
```

### `POST /v1/feed/{feed_item_id}/reactions`
Request:
```json
{
  "reaction_type": "support"
}
```
Response:
```json
{
  "reaction_id": "react_123"
}
```

### `POST /v1/groups/{group_id}/nudges`
Request:
```json
{
  "to_user_id": "usr_456",
  "context_feed_item_id": "feed_123",
  "message": "Want to do today’s check-in?"
}
```
Response:
```json
{
  "nudge_id": "ndg_123",
  "status": "sent"
}
```

Event/trigger notes:
- Comment/reaction/nudge created -> enqueue recipient notifications (respect quiet hours).
- Nudge frequency should be rate-limited per sender/recipient/day.

## Weekly Summaries & Recommendations

### `GET /v1/groups/{group_id}/weekly-summaries/current`
Response:
```json
{
  "weekly_summary_id": "sum_123",
  "week_start_date": "2026-02-16",
  "week_end_date": "2026-02-22",
  "insights": [
    {
      "id": "ins_123",
      "scope": "user",
      "user_id": "usr_123",
      "content": "Highest completion on weekdays with morning check-ins."
    }
  ],
  "recommendations": [
    {
      "id": "rec_123",
      "user_id": "usr_123",
      "content": "Keep commitment at 5 days and pre-schedule reminders."
    }
  ]
}
```

### `POST /v1/recommendations/{recommendation_id}/accept`
Request:
```json
{
  "create_commitment": true
}
```
Response:
```json
{
  "accepted": true,
  "new_commitment_id": "com_456"
}
```

Event/trigger notes:
- Scheduled weekly job -> compute summary and insights.
- Summary completion -> publish recommendation set.
- Recommendation accepted -> seed next week commitment draft.

## Meetups (Optional)

### `POST /v1/groups/{group_id}/meetup-proposals`
Request:
```json
{
  "title": "Sunday planning call",
  "proposed_start": "2026-02-21T17:00:00Z",
  "duration_minutes": 45,
  "location_type": "virtual",
  "location_value": "video_link"
}
```
Response:
```json
{
  "meetup_proposal_id": "mp_123"
}
```

### `POST /v1/meetup-proposals/{meetup_proposal_id}/respond`
Request:
```json
{
  "response": "accept"
}
```
Response:
```json
{
  "saved": true
}
```

### `POST /v1/meetup-proposals/{meetup_proposal_id}/confirm`
Request:
```json
{
  "start_at": "2026-02-21T17:00:00Z",
  "end_at": "2026-02-21T17:45:00Z"
}
```
Response:
```json
{
  "meetup_event_id": "mevt_123"
}
```

### `GET /v1/groups/{group_id}/meetup-events`
Response:
```json
{
  "items": [
    {
      "id": "mevt_123",
      "start_at": "2026-02-21T17:00:00Z",
      "status": "scheduled"
    }
  ]
}
```
