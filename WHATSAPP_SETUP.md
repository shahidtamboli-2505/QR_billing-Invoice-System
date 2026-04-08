# WhatsApp Invoice Sender - Setup Guide

## Overview
Automatically send order invoices via WhatsApp when customers mark orders as paid. Invoices include itemized list, total, and order details.

## Architecture

### Workflow
1. Admin marks order as **"Paid"** in Orders page
2. System auto-creates invoice
3. WhatsApp invoice is **sent automatically** to customer (if configured)
4. Admin can also **manually send** invoices from Billing page

### Files Changed
- `server/src/services/whatsappService.js` — WhatsApp API integration
- `server/src/controllers/orderController.js` — Auto-send on invoice creation
- `server/src/controllers/billingController.js` — Add send endpoint
- `server/src/routes/billingRoutes.js` — New POST route for sending
- `client/src/services/billingService.js` — Client API call
- `client/src/pages/admin/BillingPage.jsx` — UI button to send

---

## Setup (Optional - For Production)

### Step 1: Get Twilio Credentials
1. Sign up at https://www.twilio.com
2. Go to **Console** → **Account Info**
3. Copy:
   - `Account SID`
   - `Auth Token`
4. Go to **Explore Products** → **Messaging** → **WhatsApp**
5. Set up WhatsApp Business Account
6. Your WhatsApp sender number will look like: `whatsapp:+1234567890`

### Step 2: Update `.env`
```bash
# server/.env
TWILIO_ACCOUNT_SID=your_account_sid_here
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_WHATSAPP_NUMBER=whatsapp:+1234567890
```

### Step 3: Restart Server
```bash
cd server
npm run dev
```

---

## Features

### Auto-Send
- When order status → **Paid**, invoice auto-sends to customer WhatsApp
- Fire-and-forget: doesn't block order creation
- Server logs success/failure

### Manual Send
- **Billing page** → Each invoice has **"📱 Send WhatsApp"** button
- Click to retry or send to new number
- Toast notifications show status

### Invoice Message Format
```
🧇 *Belgian Bliss Invoice*

📋 Order #ABC123
🪑 Table 2
💳 Payment: Cash

📦 Items:
• Belgian Waffle ×1 = ₹570
• Extra Chocolate Drizzle ×1 = ₹100

💰 *Total: ₹670*

✅ Thank you for your order!
```

---

## Testing (Without Twilio)

If `TWILIO_*` env vars are **not set**:
- Console logs: `"⚠️  WhatsApp service not configured"`
- Return: `{ success: false, message: "WhatsApp not configured" }`
- App continues working normally
- Use to test locally without Twilio

---

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Missing Twilio config | Logs warning, continues processing |
| Invalid phone number | Twilio API returns error, caught gracefully |
| Network failure | Logged, order still created |
| Invoice already sent | No duplicate sends on retry |

---

## Future Enhancements
- [ ] WhatsApp delivery status tracking
- [ ] Resend invoice to different number
- [ ] Schedule invoice sends
- [ ] Multiple WhatsApp API providers (meta-direct)
- [ ] QR code to invoice in message

---

## Troubleshooting

**Q: Invoice not sending?**
- Check Twilio credentials in `.env`
- Verify WhatsApp number format: `+91...` (with country code)
- Check server logs for API errors

**Q: Customer got duplicate messages?**
- Clicking "Send" button multiple times sends multiple messages (by design)
- Users should confirm before re-sending

**Q: How to test without real Twilio?**
- Leave `.env` WITHOUT Twilio vars
- System logs to console instead
- Check simulated payload in server logs
