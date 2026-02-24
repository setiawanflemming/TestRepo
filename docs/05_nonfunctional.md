# 05 Nonfunctional Requirements (MVP)

## Privacy Basics
- Collect only data needed for accountability workflows.
- Default visibility should be group-only for personal activity.
- Minimize sensitive content in notifications (especially lock-screen previews).
- Support user data export and account deletion flows at MVP level.
- Keep audit trails for key social actions (comments, nudges, meetup proposals).

## Safety & Moderation (High-Level)
- Clear group conduct guidelines: supportive, respectful, non-harassing behavior.
- Basic reporting path for harmful content or user behavior.
- Ability to mute/block interactions within a group context.
- Rate limits on comments and nudges to reduce spam.
- Flag repeated negative interactions for admin review queue.

## Notifications & Nudges Principles
- Notification strategy should reinforce habits, not generate stress.
- Respect timezone and user quiet hours.
- Prefer batched reminders when possible (daily digest over excessive pings).
- Nudges should be opt-in and configurable by recipient.
- Nudge copy should be encouragement-oriented and short.
- Streak-at-risk reminders should fire at most once per user per local day.
- Enforce limits, for example:
  - Max nudges per sender->recipient per day.
  - Cooldown window after ignored nudges.

## Reliability & Performance Targets (MVP)
- Core actions (`check-in`, `comment`, `reaction`) should feel near real-time.
- Weekly summary jobs should complete before next planning window.
- Graceful degradation: if summary generation is delayed, preserve check-in flow.
- Mobile-first payload sizing and pagination on feed endpoints.

## Observability & Metrics

### Product Metrics
- DAU/WAU.
- Daily check-in completion rate.
- Weekly commitment completion rate.
- Week-1 and week-4 retention.
- Percentage of groups with active engagement (comments/reactions/nudges).
- Meetup proposal and completion rates (optional flow).

### Group Health Signals
- Group participation distribution (avoid one-person dominance).
- Check-in streak continuity at user and group level.
- Response latency to posts (time to first support interaction).
- Silent group detection (no feed interactions in defined window).

### Operational Metrics
- API latency and error rate by endpoint.
- Notification delivery success/failure.
- Summary pipeline job success rate and runtime.
- Queue backlogs for events/notifications.

## MVP Non-Goals
- Advanced automated moderation decisions.
- Clinical/medical claims or outcomes tracking.
- Highly personalized recommendation model tuning.
