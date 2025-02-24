# Automated_SMS_Scheduler_System

## Overview
This system enables restaurant merchants to create and schedule bulk promotional SMS campaigns for their customers. The MVP focuses on essential functionalities such as campaign management, SMS scheduling, and billing.

---
## 1. Frontend Pages

### **Merchant Dashboard**
- Displays key metrics:
  - Number of SMS campaigns sent.
  - Upcoming scheduled campaigns.
  - Remaining SMS credit balance.
  - Performance reports (click rates, delivery status).

### **Campaign Creation Page**
- Form to create an SMS campaign:
  - **Campaign Name**
  - **Message Content**
  - **Recipient List Upload** (CSV/Manual Entry)
  - **Scheduled Send Time**
  - **SMS Cost Estimation**

### **Campaign Management Page**
- List of all campaigns with:
  - Status (Scheduled, Sent, Failed)
  - Actions to **edit or cancel** scheduled campaigns.
  - **Detailed reports** on SMS delivery.

### **Billing & Payment Page**
- Displays **SMS credits** and payment history.
- Payment gateway integration (Stripe/Razorpay) for purchasing credits.

### **Settings Page**
- Manage **Sender ID**, **SMS provider settings**, and **Notifications**.

---
## 2. API Requirements

### **Merchant Management APIs**
- `POST /merchants/register` - Register a new merchant.
- `POST /merchants/login` - Authenticate merchant.
- `GET /merchants/profile` - Retrieve merchant details.

### **Campaign Management APIs**
- `POST /campaigns` - Create a new SMS campaign.
- `GET /campaigns` - Fetch all campaigns.
- `GET /campaigns/{id}` - Fetch details of a specific campaign.
- `PATCH /campaigns/{id}` - Edit campaign details before execution.
- `DELETE /campaigns/{id}` - Cancel a scheduled campaign.

### **SMS Scheduling APIs**
- `POST /sms/send` - Send an SMS immediately.
- `POST /sms/schedule` - Schedule an SMS for future sending.
- `GET /sms/status/{campaignId}` - Fetch SMS delivery status.

### **Billing & Payment APIs**
- `GET /billing` - Fetch SMS balance and billing details.
- `POST /billing/recharge` - Process payment and update SMS credits.

### **External Tool Integrations**
- **SMS Gateways**: Twilio, Nexmo, MSG91.
- **Payment Gateways**: Stripe, Razorpay.
- **Queue System**: Redis + BullMQ or AWS SQS for scheduling messages.

---
## 3. Database Schema

### **Merchants Table**
```sql
CREATE TABLE merchants (
    id UUID PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20) NOT NULL,
    password_hash TEXT NOT NULL,
    sms_credits INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Campaigns Table**
```sql
CREATE TABLE campaigns (
    id UUID PRIMARY KEY,
    merchant_id UUID REFERENCES merchants(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    status VARCHAR(50) CHECK (status IN ('Scheduled', 'Sent', 'Failed')),
    scheduled_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Recipients Table**
```sql
CREATE TABLE recipients (
    id UUID PRIMARY KEY,
    campaign_id UUID REFERENCES campaigns(id) ON DELETE CASCADE,
    phone VARCHAR(20) NOT NULL,
    status VARCHAR(50) CHECK (status IN ('Pending', 'Sent', 'Failed')),
    sent_at TIMESTAMP
);
```

### **Transactions Table**
```sql
CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    merchant_id UUID REFERENCES merchants(id) ON DELETE CASCADE,
    amount DECIMAL(10,2) NOT NULL,
    type VARCHAR(50) CHECK (type IN ('Credit Purchase', 'Refund')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---
## 4. SMS Scheduling & Processing

### **How Scheduling Works**
1. Merchant **creates** a campaign via `POST /campaigns`.
2. The system **validates** and stores the campaign.
3. If **scheduled**, a **Redis Job (BullMQ) or AWS SQS** is created.
4. When the time arrives:
   - The job **retrieves recipients**.
   - Sends SMS via **Twilio/Nexmo API**.
   - Updates recipient **delivery status**.

### **How Payment Works**
1. Merchant selects an SMS **credit package**.
2. `POST /billing/recharge` is triggered.
3. Payment gateway (Stripe/Razorpay) handles authentication.
4. On success:
   - **SMS credit balance** is updated.
   - **Transaction record** is stored.

---
## 5. Future Enhancements
- **Multi-SMS Provider Support**: Switch between different SMS providers.
- **AI-based Customer Segmentation**: Suggest target customers.
- **Automated Reports**: Detailed analytics.
- **Personalized SMS**: Use templates with placeholders.

---
## 6. Conclusion
This MVP provides **restaurant merchants** with a robust and scalable system for sending promotional SMS messages. The architecture ensures **flexibility**, **easy provider switching**, and **efficient queue management**.

---
