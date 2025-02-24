# Automated SMS Scheduler System - MVP

## 1. Frontend Pages - Detailed Explanation
The system will have a merchant-facing web application where restaurant owners can log in and manage their SMS campaigns.

### Merchant Dashboard
This is the landing page for merchants after they log in.
Displays key metrics such as:
- Number of SMS campaigns sent.
- Upcoming scheduled campaigns.
- Remaining SMS credit balance.
- Performance reports (click rates, delivery status).

### Campaign Creation Page
A form that allows merchants to set up an SMS campaign:
- **Campaign Name** - To identify the campaign internally.
- **Message Content** - The SMS text to be sent.
- **Recipient List Upload** - Allows merchants to:
  - Manually enter phone numbers.
  - Upload a CSV file containing customer details.
- **Scheduled Send Time** - Choose whether to send immediately or schedule for later.
- **SMS Cost Estimation** - Based on message length and number of recipients.

### Campaign Management Page
- A table-style list of all SMS campaigns.
- Each row will have:
  - Campaign name, status, scheduled time.
  - Actions to edit or cancel scheduled campaigns before execution.
- Detailed campaign reports showing:
  - SMS delivery success/failure.

### Billing & Payment Page
- Displays current SMS credits and payment history.
- Payment gateway integration for purchasing SMS credits.


---
## 2. API Requirements - Detailed Explanation
The backend will have a RESTful API handling authentication, campaign management, and SMS scheduling.

### Merchant Management APIs
- `POST /merchants/register` - Registers a new restaurant merchant.
- `POST /merchants/login` - Authenticates the merchant using email & password.
- `GET /merchants/profile` - Returns the logged-in merchant's details.

### Campaign Management APIs
- `POST /campaigns` - Allows a merchant to create a campaign.
- `GET /campaigns` - Fetches all campaigns of the logged-in merchant.
- `GET /campaigns/{id}` - Fetches details of a specific campaign.
- `PATCH /campaigns/{id}` - Allows merchants to edit campaign details before execution.
- `DELETE /campaigns/{id}` - Allows merchants to cancel an upcoming campaign.

### SMS Scheduling APIs
- `POST /sms/send` - Sends an SMS immediately.
- `POST /sms/schedule` - Schedules an SMS for future sending.
- `GET /sms/status/{campaignId}` - Fetches SMS delivery status.

### Billing & Payment APIs
- `GET /billing` - Returns the merchant's current SMS balance and payment history.
- `POST /billing/recharge` - Processes payments for purchasing SMS credits.

### External Tool Integrations
- **Queue System** (Redis + BullMQ) for scheduling SMS messages.

---
## 3. Database Schema - Detailed Explanation
A relational database like PostgreSQL or MySQL is used to store merchants, campaigns, recipients, and transactions.

```json
{
  "merchants": {
    "id": "UUID",
    "name": "VARCHAR(255)",
    "email": "VARCHAR(255) UNIQUE",
    "phone": "VARCHAR(20)",
    "password_hash": "TEXT",
    "sms_credits": "INT DEFAULT 0",
    "created_at": "TIMESTAMP DEFAULT CURRENT_TIMESTAMP"
  },
  "campaigns": {
    "id": "UUID",
    "merchant_id": "UUID REFERENCES merchants(id) ON DELETE CASCADE",
    "name": "VARCHAR(255)",
    "message": "TEXT",
    "status": "VARCHAR(50) CHECK (status IN ('Scheduled', 'Sent', 'Failed'))",
    "scheduled_at": "TIMESTAMP",
    "created_at": "TIMESTAMP DEFAULT CURRENT_TIMESTAMP"
  },
  "recipients": {
    "id": "UUID",
    "campaign_id": "UUID REFERENCES campaigns(id) ON DELETE CASCADE",
    "phone": "VARCHAR(20)",
    "status": "VARCHAR(50) CHECK (status IN ('Pending', 'Sent', 'Failed'))",
    "sent_at": "TIMESTAMP"
  },
  "transactions": {
    "id": "UUID",
    "merchant_id": "UUID REFERENCES merchants(id) ON DELETE CASCADE",
    "amount": "DECIMAL(10,2)",
    "type": "VARCHAR(50) CHECK (type IN ('Credit Purchase', 'Refund'))",
    "created_at": "TIMESTAMP DEFAULT CURRENT_TIMESTAMP"
  }
}
```

---
## 4. SMS Scheduling & Processing - Detailed

### Scheduling Works
1. Merchant creates a campaign via `POST /campaigns`.
2. The system validates the request and stores it in the database.
3. If scheduled, a Redis Job (BullMQ) or AWS SQS is created.
4. When the scheduled time arrives:
   - The job retrieves the recipient list.
   - Sends SMS.
   - Updates recipient delivery status in the database.

### Payment Works
1. Merchant selects an SMS credit package.
2. `POST /billing/recharge` is triggered.
3. Payment gateway handles payment authentication.
4. On successful payment:
   - The merchant's SMS credit is updated.
   - A transaction record is created.

---
