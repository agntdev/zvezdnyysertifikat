# Star Naming Service — Bot specification

**Archetype:** commerce

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that sells named stars to users who purchase for themselves via manual bank card transfer. The bot handles product selection, order creation, payment instructions, payment confirmation flow, certificate generation, and delivery of the purchased artifact inside Telegram.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- Individual customers who want to buy and name a star for themselves
- Non-technical users comfortable with Telegram and bank transfers

## Success criteria

- User receives digital certificate and star details after successful payment confirmation
- Admin receives notifications for new orders and payment proofs
- Order status updates are pushed to the buyer in Telegram

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open the main menu
- **Browse packages** (button, actor: user, callback: browse:start) — Show available star packages
  - inputs: selected package
  - outputs: package details
- **My purchases** (button, actor: user, callback: purchases:view) — View purchase history
  - inputs: user profile
  - outputs: order history
- **Help** (button, actor: user, callback: help:show) — Show help and support information
  - inputs: user query
  - outputs: help response

## Flows

### Star Purchase Flow
_Trigger:_ /start or browse:start

1. User selects a star package
2. User enters star name and optional message
3. Bot validates name and checks uniqueness
4. Bot creates order and shows payment instructions
5. User uploads payment proof
6. Admin verifies payment and confirms order
7. Bot generates and delivers certificate
8. User receives certificate and order summary

_Data touched:_ User profile, Product, Order, Star record

### Admin Verification Flow
_Trigger:_ New order with payment proof

1. Admin receives notification
2. Admin reviews payment proof
3. Admin confirms or rejects order
4. Bot updates order status and notifies user

_Data touched:_ Order, Star record

### Order Cancellation Flow
_Trigger:_ User cancels order or payment deadline passes

1. Order status is updated to cancelled
2. User is notified of cancellation
3. Star name is released for reuse

_Data touched:_ Order, Star record

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User profile** _(retention: persistent)_ — Telegram ID, display name, email (optional), language, purchase history
  - fields: Telegram ID, Display name, Email, Language, Purchase history
- **Product** _(retention: persistent)_ — Star package (name, description, price, included data such as coordinate precision, certificate style, additional metadata)
  - fields: Name, Description, Price, Coordinate precision, Certificate style, Metadata
- **Order** _(retention: persistent)_ — Order id, buyer (user), selected product, star name (validated), optional personal message, price, status (pending, awaiting payment proof, confirmed, delivered, cancelled), payment instructions, payment proof (image/file), timestamps
  - fields: Order ID, Buyer, Selected product, Star name, Personal message, Price, Status, Payment instructions, Payment proof, Timestamps
- **Star record** _(retention: persistent)_ — Unique star id, assigned name, coordinates (RA/Dec or catalog-like string), registration date, linked order id, certificate file URL
  - fields: Unique star ID, Assigned name, Coordinates, Registration date, Linked order ID, Certificate file URL

## Integrations

- **Telegram** (required) — Bot API messaging
- **File storage** (required) — Store generated certificate PDFs/images and optionally payment-proof attachments
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure admin notification target (Telegram user/group)
- Set package prices and descriptions
- Manage profanity list for name validation
- Set order lifetime for payment (default 7 days)
- Set certificate template styles
- Set retention period for data (default 1 year)

## Notifications

- Order status updates pushed to buyer in Telegram
- New orders and payment proofs sent to admin(s) for manual verification
- Follow-up message after certificate delivery

## Permissions & privacy

- Minimal personal data stored (Telegram ID, display name, optional email)
- Payment-proof files stored securely
- Data retention for 1 year to support order history and dispute handling

## Edge cases

- User submits invalid payment proof
- Admin rejects order due to mismatched amount
- Multiple users attempt to name the same star simultaneously
- User cancels order before payment deadline
- Payment deadline passes without confirmation

## Required tests

- End-to-end star purchase flow from selection to certificate delivery
- Admin verification flow with confirmation and rejection scenarios
- Order cancellation flow when user cancels or payment deadline passes
- Name validation and uniqueness check for star names
- Data retention and cleanup after retention period

## Assumptions

- Payment method is manual bank card transfer only
- Admin verification is manual via Telegram buttons
- Certificate format is PDF with decorative preview
- Star coordinates are randomly assigned within defined celestial coordinates
- Name validation allows letters, digits, spaces, and common punctuation up to 64 characters with profanity filtering
- Admin notifications are sent to a single configurable Telegram admin user/group
- Order lifetime for payment is 7 days
- Data retention period is 1 year
- Default language is Russian
