# Pond Mobile Payment Gateway

Secure payment integration for POND mobile using Authorize.net Hosted Payment Page.

## Architecture

```
┌─────────────┐      ┌─────────────┐      ┌─────────────────┐
│ HubSpot     │ ───> │ Flask (VPS) │ ───> │ Authorize.net   │
│ Static HTML │      │ GoDaddy US  │      │ Hosted Page     │
└─────────────┘      └─────────────┘      └─────────────────┘
```

## Security & Compliance

- **PCI DSS Compliant**: Payment card data never touches our servers
- **Hosted Payment Page**: All sensitive data handled by Authorize.net
- **No Data Storage**: Stateless backend, no database, no payment logs
- **US-based**: VPS hosted on GoDaddy US, data stays within jurisdiction

## Tech Stack

- **Backend**: Flask (Python 3.12+)
- **Server**: Gunicorn + Nginx (Ubuntu 24.04 LTS)
- **Payment**: Authorize.net API
- **Frontend**: Vanilla HTML/JS (hosted on HubSpot)

## Environment Variables

```bash
AUTHORIZE_API_LOGIN_ID=your_login_id
AUTHORIZE_TRANSACTION_KEY=your_transaction_key
APP_BASE_URL=https://payments.pondmobile.com
```

## Deployment

See deployment documentation (TBD).
