# Habit Tracker Frontend

A lightweight dashboard for the [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend) — built with vanilla HTML, CSS, and JavaScript. No frameworks, no dependencies, no build step.

**Live:** https://habit.johnhardin.site

## Pages

| File | Description |
|---|---|
| `index.html` | Main dashboard — add, view, and delete habits |
| `weekly.html` | Weekly summary — completion grid and stats for any week |

## Features

### index.html
- Add habits with a name, email, schedule, and reminder time
- Choose from daily, weekdays, weekends, or specific days of the week
- Set a reminder time in Jakarta time (UTC+7)
- Delete habits
- **User selector** — switch between multiple user IDs from a dropdown; known users are saved to `localStorage` and auto-populated when you add a habit for a new user

### weekly.html
- **7-day completion grid** — one row per habit, one column per day (Mon–Sun)
- Navigate to any past or future week with prev/next controls
- Click a cell to mark a habit done for that day — calls the `/complete` API and persists state in `localStorage`
- Per-habit completion rate and colour-coded overall percentage (green ≥ 70%, orange ≥ 40%, red < 40%)
- Habits not scheduled on a given day are shown as dimmed, future dates are greyed out
- Links back to `index.html`

## Tech Stack

- Vanilla HTML, CSS, JavaScript
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
      │  API calls
      ▼
API Gateway → Lambda → DynamoDB
(see habit-tracker-backend for full backend architecture)
```

## How It Works

The dashboard is two HTML files with no build process or dependencies. They make direct HTTP requests to the API Gateway endpoints defined in the backend.

When you add a habit, `index.html` sends a POST request to the backend API which saves it to DynamoDB. `weekly.html` fetches the habit list and tracks completion state in `localStorage`, syncing done-marks back to DynamoDB via the `/complete` endpoint.

## Hosting Setup

### Prerequisites
- AWS account
- S3 bucket (private)
- CloudFront distribution pointing to the S3 bucket
- ACM certificate for your custom domain (if using one)

### Deploy

**1. Upload to S3**
```bash
aws s3 cp index.html s3://your-bucket-name/index.html --region ap-southeast-1
aws s3 cp weekly.html s3://your-bucket-name/weekly.html --region ap-southeast-1
```

**2. Invalidate CloudFront cache after updates**
```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

### Update API endpoint

Update the `API` constant at the top of the script in both files:

```javascript
const API = 'https://your-api-gateway-url.execute-api.ap-southeast-1.amazonaws.com';
```

## Local Development

No build step needed. Serve both files with Python:

```bash
python3 -m http.server 8000
```

Then open in your browser:
- `http://localhost:8000/index.html` — main dashboard
- `http://localhost:8000/weekly.html` — weekly summary

## Related

- [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend) — Lambda functions, DynamoDB, SES, EventBridge
