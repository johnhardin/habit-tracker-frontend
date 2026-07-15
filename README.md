# Habit Tracker Frontend

Static browser frontend for the Habit Tracker project. The client is intentionally lightweight: plain `HTML`, `CSS`, and `JavaScript`, with no framework, no package manager, and no build step.

This repository is best read as the UI layer for the AWS backend in [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend).


## What this frontend is responsible for

- user sign-up, sign-in, and sign-out with AWS Cognito
- session restore on page load
- creating and deleting habits through the API
- rendering a weekly completion grid
- sending authenticated API requests with a JWT bearer token

It does not send reminders directly. Reminder scheduling, completion persistence, and weekly email generation all live in the backend.

## Pages

| File | Purpose |
|---|---|
| `index.html` | Authentication flow and main habit dashboard |
| `weekly.html` | Weekly summary page and completion grid |

## Frontend architecture

```text
Browser
  |
  +--> Cognito JS SDK
  |       |
  |       v
  |    ID token (JWT)
  |
  v
Static HTML/JS
  |
  v
API Gateway
  |
  v
Lambda backend
```

The frontend is hosted as static files and depends on three values configured directly in both pages:

```javascript
const USER_POOL_ID = 'ap-southeast-1_AbCdEfGhI';
const CLIENT_ID    = 'your-app-client-id';
const API          = 'https://your-api-gateway-url.execute-api.ap-southeast-1.amazonaws.com';
```

`USER_POOL_ID` and `CLIENT_ID` are public identifiers, not secrets.

## Authentication flow

`index.html` embeds the Cognito browser SDK from a CDN and implements:

- sign-up with email and password
- email verification with a six-digit code
- sign-in using Cognito authentication
- session restoration on reload via `getCurrentUser()` and `getSession()`

Once a session is established, the frontend stores the JWT in memory and sends it as:

```http
Authorization: Bearer <jwt>
```

The backend then uses the JWT `sub` claim as the source of truth for user identity.

## Main user flows

### Dashboard (`index.html`)

The dashboard supports:

- create a habit with `name`, `email`, `schedule`, and optional `reminderTime`
- list all habits for the signed-in user
- delete a habit
- navigate to the weekly summary page

Supported schedule formats:

- `daily`
- `weekdays`
- `weekends`
- `specific:Mon,Wed,Fri`

### Weekly summary (`weekly.html`)

The weekly page:

- requires a valid Cognito session
- fetches habits from `GET /habits`
- fetches completion records for a date range from `GET /completions`
- renders a 7-day grid from Monday through Sunday
- computes per-habit and overall completion percentages
- calls `GET /complete` when a user marks a day as complete

## Local development

No build step is required.

Run a static file server from the project directory:

```bash
python3 -m http.server 8000
```

Open:

- `http://localhost:8000` - dashboard
- `http://localhost:8000/weekly.html` - weekly view

For local authentication to work, `localhost` must be allowed in the Cognito app client configuration.

## Deployment

This frontend is designed for static hosting on:

- Amazon S3
- Amazon CloudFront
- optional custom domain with ACM-managed TLS

Deployment in the original project is automated with GitHub Actions. Manual deployment is also straightforward:

```bash
aws s3 sync . s3://your-bucket-name \
  --exclude ".git/*" --exclude ".github/*" \
  --delete --region ap-southeast-1
```

Then invalidate CloudFront:

```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

## Known limitations

- The frontend is a single-file-per-page implementation, so UI logic, markup, and styling are coupled intentionally for simplicity.
- The weekly page shows "undo" interaction button in the UI, but the backend does not currently provide a matching delete/un-complete API. As a result, that undo button only for visual implement and if you reload the page, it still shows the habit is completed.
- Configuration is hardcoded into the HTML files and must be updated before deployment.

For the platform side of the project, see [Habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend)
