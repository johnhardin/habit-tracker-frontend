# Habit Tracker Frontend

A lightweight dashboard for the [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend) — built with vanilla HTML, CSS, and JavaScript. No frameworks, no dependencies, no build step.

**Live:** https://habit.johnhardin.site

## Features

- Add habits with a name, email, schedule, and reminder time
- Choose from daily, weekdays, weekends, or specific days of the week
- Set a reminder time in Jakarta time (UTC+7)
- Delete habits
- All data persists in DynamoDB via the backend API

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

The dashboard is a single HTML file with no build process or dependencies. It makes direct HTTP requests to the API Gateway endpoints defined in the backend.

When you add a habit, the dashboard sends a POST request to the backend API which saves it to DynamoDB. The backend Lambda functions handle all the scheduling and email logic independently — the frontend only manages habit creation and deletion.

## Hosting Setup

### Prerequisites
- AWS account
- S3 bucket (private)
- CloudFront distribution pointing to the S3 bucket
- ACM certificate for your custom domain (if using one)

### Deploy

**1. Upload to S3**
```bash
aws s3 cp index.html s3://your-bucket-name/habit-tracker.html \
  --region ap-southeast-1
```

**2. Invalidate CloudFront cache after updates**
```bash
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

### Update API endpoint

If you deploy your own backend, update the `API` constant at the top of the script in `index.html`:

```javascript
const API = 'https://your-api-gateway-url.execute-api.ap-southeast-1.amazonaws.com';
```

## Local Development

No build step needed. Just serve the file with Python:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/index.html` in your browser.

## Related

- [habit-tracker-backend](https://github.com/johnhardin/habit-tracker-backend) — Lambda functions, DynamoDB, SES, EventBridge