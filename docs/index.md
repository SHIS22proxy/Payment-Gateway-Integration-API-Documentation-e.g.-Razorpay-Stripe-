# Payment Gateway Integration API Documentation (e.g., Razorpay, Stripe)

## 1. Introduction

### Overview  
The **Payment Gateway Integration API** allows your application to connect with external payment providers like **Stripe** or **Razorpay** to securely handle transactions, process customer payments, and manage payment-related events such as refunds, verifications, and webhook notifications.

### Why It Matters  
Secure and seamless payment processing is critical for any digital commerce system. A robust integration ensures smooth customer experience, reduces payment failures, and complies with security and regulatory standards (e.g., PCI DSS).

### Who This Guide is For  
- Backend developers integrating payment flows  
- Frontend developers implementing checkout interfaces  
- Security engineers validating PCI compliance  
- Product teams enabling subscription or one-time payments  
- Finance/operations staff managing refunds and reconciliation

---

## 2. Key Terminology

- **Payment Intent**: (Stripe) An object representing the lifecycle of a payment  
- **Order ID**: A unique reference to the order being paid for  
- **Checkout Session**: A hosted payment page session (Stripe/Razorpay Checkout)  
- **Webhook**: An HTTP callback sent by the gateway to notify of payment events  
- **Capture**: The process of finalizing a previously authorized payment  
- **Refund**: Returning the amount to the customer post-purchase  
- **Tokenization**: Encrypting payment info for secure, one-time or repeat use  
- **PCI DSS**: Payment Card Industry Data Security Standard for compliance  

---

## 3. Technical Overview

### Architecture Flow

Your backend acts as a secure intermediary between the frontend and the payment gateway. It creates the transaction, verifies the status via webhooks, and updates your order system accordingly.

![flow 2](<assets/flow 2.png>){ width="400px" style="border: 1px solid #ccc; border-radius: 8px;"}

## Supported Gateways

- **Stripe**: Global gateway with support for cards, wallets, and bank debits  
- **Razorpay**: Popular in India; supports UPI, cards, wallets, and EMI  

## Backend Tools

- [Stripe SDKs](https://stripe.com/docs)  
- [Razorpay SDKs](https://razorpay.com/docs/)  
- Node.js / Django / Spring Boot for backend logic  
- HTTPS & SSL for all webhook endpoints  

---

## 4. Step-by-Step Guide or Workflow

### 4.1 Create a Checkout Session (Backend)

**Stripe Example**

**Endpoint**:  
`POST /api/payments/stripe/checkout`

**Node.js Code Example**:

```
js
const stripe = require('stripe')('sk_test_...');
const session = await stripe.checkout.sessions.create({
  payment_method_types: ['card'],
  line_items: [{
    price_data: {
      currency: 'usd',
      product_data: {
        name: 'Wireless Mouse',
      },
      unit_amount: 2500,
    },
    quantity: 1,
  }],
  mode: 'payment',
  success_url: 'https://yourdomain.com/success',
  cancel_url: 'https://yourdomain.com/cancel',
});
```
### Response:
```
json
{
  "checkout_url": "https://checkout.stripe.com/pay/cs_test_..."
}
```
## 4.2 Handle Webhooks
### Endpoint:
```
POST /api/webhooks/stripe
```

#### Example Payload:
```
json
{
  "type": "checkout.session.completed",
  "data": {
    "object": {
      "payment_intent": "pi_123",
      "customer_email": "user@example.com"
    }
  }
}
```
#### Action:
###### Update order status in your database to paid.

### 4.3 Verify Payment (Optional Manual Check)
#### Endpoint:
```
GET /api/payments/stripe/status/:payment_intent_id
```

### 4.4 Issue a Refund (Backend)
```
js
const refund = await stripe.refunds.create({
  payment_intent: 'pi_123',
});
```

### 4.5 Razorpay Example (Payment Capture)
```
js
const razorpay = new Razorpay({ key_id: 'rzp_test...', key_secret: '...' });

const order = await razorpay.orders.create({
  amount: 50000, // in paise
  currency: "INR",
  receipt: "order_rcptid_11",
  payment_capture: 1
});
```

#### Frontend Integration:
```
html
<script src="https://checkout.razorpay.com/v1/checkout.js"></script>
```

## 5. Best Practices
Use HTTPS Everywhere — especially for webhooks and callbacks

Never Handle Card Details Directly — use the gateway’s secure UI or SDK

Store Transaction IDs for auditing and reconciliation

Validate Webhooks with Signature Secret to prevent spoofing

Use Idempotent Requests to avoid duplicate charges on retries

Separate Sandbox and Production Keys

Notify Users Post-Payment — send confirmation emails or receipts

### 6. Common Issues & Troubleshooting

|Issue	| Description |	Resolution |
|-------|-------------|------------|
|400 |Bad Request	| Invalid parameters	Double-check the request structure|
|402 | Payment Required	Card declined	|Inform user and retry|
|403 | Forbidden	| Using test keys in prod	Replace with live credentials|
|409 | Conflict	Payment already captured	|Use idempotency keys|
|Signature | Mismatch	| Webhook spoofing attempt	Verify webhook with signature secret|

## 7. References
Stripe API Docs

Razorpay Integration Docs

PCI Compliance Guide

OAuth 2.0 for Secure Auth

Handling Webhooks Securely

## 8. Appendix
### 8.1 Sample Webhook Verification (Stripe, Node.js)
```
js
const endpointSecret = "whsec_...";
const sig = req.headers['stripe-signature'];

let event;
try {
  event = stripe.webhooks.constructEvent(req.body, sig, endpointSecret);
} catch (err) {
  return res.status(400).send(`Webhook Error: ${err.message}`);
}
```

### 8.2 Mermaid Flow: Razorpay Payment Process
![flow 1](<assets/flow 1.png>){ width="400px" style="border: 1px solid #ccc; border-radius: 8px;"}

### 8.3 Curl Example: Stripe Checkout Session
```
bash
curl -X POST https://api.yoursite.com/api/payments/stripe/checkout \
  -H "Authorization: Bearer <your_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": "ORD123",
    "amount": 5000,
    "currency": "usd"
  }'
```