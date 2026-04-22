# Habit Tracker Frontend

A lightweight dashboard for the [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend) — built with vanilla HTML, CSS, and JavaScript. No frameworks, no dependencies, no build step.

**Live:** https://habit.johnhardin.site

## Pages

| File | Description |
|---|---|
| `index.html` | Login / sign-up screen + main dashboard — add, view, and delete habits |
| `weekly.html` | Weekly summary — completion grid and stats for any week |

## Features

### index.html
- Sign in or create an account via AWS Cognito (email + password)
- Email verification step on sign-up
- Session is restored automatically on page reload — no re-login needed
- Add habits with a name, email, schedule, and reminder time
- Choose from daily, weekdays, weekends, or specific days of the week
- Set a reminder time in Jakarta time (UTC+7)
- Delete habits
- Link to the weekly summary page

### weekly.html
- Protected page — redirects to `index.html` if not logged in
- **7-day completion grid** — one row per habit, one column per day (Mon–Sun)
- Navigate to any past or future week with prev/next controls
- Click a cell to mark a habit done for that day — calls the `/complete` API and persists state in `localStorage`
- Per-habit completion rate and colour-coded overall percentage (green ≥ 70%, orange ≥ 40%, red < 40%)
- Habits not scheduled on a given day are shown as dimmed, future dates are greyed out

## Tech Stack

- Vanilla HTML, CSS, JavaScript
- **AWS Cognito** — user authentication and JWT token issuance
- Hosted on AWS S3 (private bucket)
- Served via AWS CloudFront (HTTPS, global CDN)
- Custom domain with SSL via AWS Certificate Manager (ACM)

## Architecture

```
User's browser
      │
      ▼
CloudFront (https://habit.johnhardin.site)
      │  serves static HTML
      ▼
S3 bucket (private, CloudFront access only)
      │
      ▼
Cognito User Pool (sign in / sign up)
      │  issues JWT token
      ▼
API Gateway (JWT Authorizer) → Lambda → DynamoDB
(see habit-tracker-backend for full backend architecture)
```

## How It Works

`index.html` handles both authentication and the habit dashboard in a single page — the login screen and app screen are shown/hidden based on the Cognito session state. On load, it checks for an existing session and jumps straight to the dashboard if valid.

All API calls include an `Authorization: Bearer <jwt>` header. The backend extracts the user identity from the JWT `sub` claim — no `userId` is passed from the client.

`weekly.html` does the same session check on load and redirects to `index.html` if not authenticated. It fetches the habit list and tracks completion state in `localStorage` (keyed by the JWT `sub` claim), syncing done-marks back to DynamoDB via the `/complete` endpoint.

## Configuration

Both `index.html` and `weekly.html` have a config block at the top of the script section. Update these before deploying:

```javascript
const USER_POOL_ID = 'ap-southeast-1_AbCdEfGhI';  // Cognito User Pool ID
const CLIENT_ID    = 'your-app-client-id';          // Cognito App Client ID
const API          = 'https://your-api-gateway-url.execute-api.ap-southeast-1.amazonaws.com';
```

`USER_POOL_ID` and `CLIENT_ID` are safe to commit — they are public identifiers, not secrets.

## Hosting Setup

### Prerequisites
- AWS account
- Cognito User Pool with an App Client (no client secret)
- S3 bucket (private)
- CloudFront distribution pointing to the S3 bucket
- ACM certificate for your custom domain (if using one)

### Deploy

Deployment is automated via GitHub Actions on every push to `main` — files are synced to S3 and CloudFront cache is invalidated automatically.

To deploy manually:

**1. Upload to S3**
```bash
aws s3 sync . s3://your-bucket-name \
  --exclude ".git/*" --exclude ".github/*" \
  --delete --region ap-southeast-1
```

**2. Invalidate CloudFront cache**
```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

### GitHub Actions secrets required

| Secret | Description |
|---|---|
| `AWS_ACCESS_KEY_ID` | IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key |
| `AWS_REGION` | e.g. `ap-southeast-1` |
| `S3_BUCKET_NAME` | S3 bucket name |
| `CLOUDFRONT_DISTRIBUTION_ID` | CloudFront distribution ID |

## Local Development

No build step needed. Serve both files with Python:

```bash
python3 -m http.server 8000
```

Then open in your browser:
- `http://localhost:8000` — login + dashboard
- `http://localhost:8000/weekly.html` — weekly summary

> Note: Cognito auth works on localhost as long as `localhost` is added to the allowed callback URLs in your Cognito App Client settings.

## Related

- [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend) — Lambda functions, DynamoDB, SES, EventBridge
