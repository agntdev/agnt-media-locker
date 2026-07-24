# Private Media Vault — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

A private, invite-only Telegram bot that stores and serves large multimedia files (photos, videos) by sending them directly in chat. Access is controlled by admin approval. Users start with 10 tokens to download files, and can replenish tokens by watching ads, purchasing token packages, or completing referral actions.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- private community members
- admins

## Success criteria

- Admins can approve/reject users and manage content
- Users can browse and download media using tokens
- Token balance updates automatically when users watch ads, purchase tokens, or complete referrals

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **Request Access** (button, actor: user, callback: request_access) — User requests to join the private community
- **Browse Media** (button, actor: user, callback: browse_media) — View available media items
- **Renew Tokens** (button, actor: user, callback: renew_tokens) — View options to replenish token balance
- **Admin Panel** (button, actor: admin, callback: admin_panel) — Access admin controls for user management and content upload

## Flows

### User Onboarding
_Trigger:_ User sends /start or taps 'Request Access'

1. User requests access
2. Admin receives notification
3. Admin approves/rejects request
4. Approved user receives 10 starter tokens

_Data touched:_ User

### Media Download
_Trigger:_ User taps media item in browse list

1. User selects media item
2. Bot sends file in Telegram chat
3. Decrement user token balance by 1

_Data touched:_ User, Media item, Token transaction

### Token Renewal - Watch Ad
_Trigger:_ User selects 'Watch ad' option

1. User watches ad
2. Verify ad completion
3. Grant 1 token to user

_Data touched:_ User, Token transaction, Renewal action

### Token Renewal - Purchase
_Trigger:_ User selects 'Buy tokens' option

1. Show token package options
2. Process payment
3. Grant tokens to user

_Data touched:_ User, Token transaction, Renewal action

### Token Renewal - Referral
_Trigger:_ User selects 'Complete referral' option

1. Show referral instructions
2. Verify referral completion
3. Grant tokens to user

_Data touched:_ User, Token transaction, Renewal action

### Admin Management
_Trigger:_ Admin accesses panel

1. View user list
2. Approve/reject users
3. Upload new media
4. Adjust user tokens
5. View analytics

_Data touched:_ User, Media item, Token transaction

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram user with access status and token balance
  - fields: telegram_id, display_name, approved, token_balance
- **Media item** _(retention: persistent)_ — Stored multimedia file with metadata
  - fields: file_id, title, description, tags, file_size
- **Token transaction** _(retention: persistent)_ — Record of token grants and spends
  - fields: user_id, amount, method, timestamp
- **Renewal action** _(retention: persistent)_ — Record of token renewal attempts
  - fields: user_id, action_type, status, timestamp

## Integrations

- **Telegram** (required) — Bot API messaging
- **Ad/Video Provider** (required) — Credit tokens upon ad completion
- **Payment Provider** (required) — Process token purchases
- **Partner Website** (optional) — Credit tokens upon referral completion
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Approve/reject users
- Upload media content
- Adjust user token balances
- View analytics (downloads, token spend)

## Notifications

- New user request notifications to admin chat
- Token balance updates to users
- Download confirmation messages
- Token grant notifications

## Permissions & privacy

- Private access requires admin approval
- User data stored securely
- Token transactions are private to users
- Media files stored in durable object storage

## Edge cases

- User tries to download without tokens
- Admin tries to access without proper permissions
- Ad completion verification fails
- Payment processing errors
- Media file not found in storage

## Required tests

- Verify user onboarding flow from request to approval
- Test media download with token decrement
- Validate token renewal through all methods
- Confirm admin controls for user management and content upload

## Assumptions

- Admin approval model uses single admin id
- Token grant timing is instant upon verified completion
- Ad integration uses standard provider with client-side callback
- Payment model uses token packages
