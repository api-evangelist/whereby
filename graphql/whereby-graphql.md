# Whereby GraphQL Schema

## Overview

This is a conceptual GraphQL schema for the Whereby embeddable video meetings API. Whereby provides a REST API at `https://api.whereby.dev/v1` for creating and managing meeting rooms, recordings, transcriptions, summaries, and insights. This GraphQL schema represents the same domain model in graph form, enabling richer querying and type-safe access to the full Whereby resource graph.

## Source

- **Provider:** Whereby
- **REST API Reference:** https://docs.whereby.com/reference/whereby-rest-api-reference
- **Auth:** Bearer token (`Authorization: Bearer <API_KEY>`)
- **Base URL:** https://api.whereby.dev/v1

## Schema File

See `whereby-schema.graphql` for the full type definitions.

## Key Types

### Meetings and Rooms

- `Meeting` — top-level meeting resource with room URL, host URL, and configuration
- `MeetingDetails` — extended metadata including participant list, status, and timing
- `MeetingType` — enum distinguishing normal, breakout, and broadcast meeting modes
- `MeetingURL` — strongly-typed scalar for the guest-facing join URL
- `RoomURL` — the persistent room URL assigned to a meeting
- `HostURL` — the privileged host join URL including access token
- `Room` — persistent room entity with branding and permission configuration
- `RoomConfig` — room-level settings for locking, waiting room, and recording
- `RoomPrivacy` — enum for public/private/knocking access modes
- `StartDate` — ISO 8601 meeting start time wrapper
- `EndDate` — ISO 8601 meeting end time wrapper

### Tokens and Security

- `MeetingToken` — short-lived bearer token granting meeting access
- `MeetingTokens` — collection of tokens for a meeting (host + participants)
- `APIKey` — organization-scoped API key for authenticating REST calls
- `Token` — generic token container with expiry and scope
- `PermissionsConfig` — participant capability flags (camera, mic, screenshare, chat)

### Participants

- `Participant` — individual in a meeting session
- `ParticipantDetails` — extended info including device, join time, and media state
- `ParticipantRole` — enum: HOST, VIEWER, PARTICIPANT
- `ParticipantMedia` — snapshot of a participant's current media tracks
- `VideoTrack` — video stream state and quality metadata
- `AudioTrack` — audio stream state and quality metadata
- `ScreenShare` — screenshare session descriptor

### Recordings

- `RecordingStatus` — enum: NONE, STARTING, RECORDING, STOPPING, FINISHED, FAILED
- `RecordingMode` — enum: LOCAL, CLOUD
- `LocalRecording` — browser-side recording artifact reference
- `CloudRecording` — server-side recording with download URL and storage details

### Transcriptions and AI

- `Transcription` — top-level transcription job for a meeting recording
- `TranscriptionDetail` — metadata including language, status, and word count
- `TranscriptSegment` — time-bounded spoken segment with speaker attribution
- `TranscriptWord` — individual word with start/end timestamp and confidence score
- `Chat` — in-meeting chat session
- `ChatMessage` — individual chat message with sender and timestamp

### Presentation and Layout

- `Presenter` — active screen presenter in a meeting
- `PresentationMode` — enum: NONE, SCREENSHARE, WHITEBOARD
- `Layout` — current visual arrangement of participant tiles
- `LayoutType` — enum: GRID, SPOTLIGHT, SIDEBAR, PRESENTATION

### Backgrounds

- `Background` — union of standard and custom background options
- `BackgroundImage` — built-in background image reference
- `CustomBackground` — user-uploaded background asset
- `VirtualBackground` — active virtual background configuration

### Access Control

- `Knock` — entry request from a participant waiting outside a locked room
- `KnockStatus` — enum: PENDING, APPROVED, REJECTED
- `LockStatus` — enum: LOCKED, UNLOCKED
- `WaitingRoom` — queue of pending knock requests

### Branding

- `Branding` — organization-level visual identity settings
- `BrandingDetails` — full branding configuration object
- `CustomLogo` — uploaded logo asset URL and dimensions
- `CustomColor` — hex color value for UI theming

### Streaming and Webhooks

- `LiveStream` — active RTMP/HLS stream configuration
- `StreamConfig` — stream destination URL and key
- `Webhook` — registered webhook endpoint
- `WebhookEvent` — enum of subscribable event types

### Organization

- `Organization` — the Whereby account/tenant owning rooms and meetings

## Queries

```graphql
type Query {
  meeting(meetingId: ID!): Meeting
  meetings(roomId: ID, after: String, first: Int): MeetingConnection
  room(roomId: ID!): Room
  rooms(after: String, first: Int): RoomConnection
  recording(recordingId: ID!): CloudRecording
  recordings(meetingId: ID, after: String, first: Int): RecordingConnection
  transcription(transcriptionId: ID!): Transcription
  transcriptions(meetingId: ID, after: String, first: Int): TranscriptionConnection
  organization: Organization
  webhooks: [Webhook!]!
}
```

## Mutations

```graphql
type Mutation {
  createMeeting(input: CreateMeetingInput!): Meeting
  deleteMeeting(meetingId: ID!): DeleteResult
  createRoom(input: CreateRoomInput!): Room
  deleteRoom(roomId: ID!): DeleteResult
  createMeetingToken(input: CreateMeetingTokenInput!): MeetingToken
  startCloudRecording(meetingId: ID!): CloudRecording
  stopCloudRecording(meetingId: ID!): CloudRecording
  deleteRecording(recordingId: ID!): DeleteResult
  startTranscription(meetingId: ID!): Transcription
  stopTranscription(transcriptionId: ID!): Transcription
  deleteTranscription(transcriptionId: ID!): DeleteResult
  approveKnock(knockId: ID!): Knock
  rejectKnock(knockId: ID!): Knock
  lockRoom(roomId: ID!): Room
  unlockRoom(roomId: ID!): Room
  updateBranding(input: UpdateBrandingInput!): Branding
  createWebhook(input: CreateWebhookInput!): Webhook
  deleteWebhook(webhookId: ID!): DeleteResult
}
```

## Notes

- All IDs are opaque strings.
- Timestamps use ISO 8601 format.
- Pagination follows the Relay connection pattern with `edges`/`node`/`pageInfo`.
- The schema is conceptual and maps to the Whereby REST API surface; a production implementation would wrap REST calls as resolvers.
