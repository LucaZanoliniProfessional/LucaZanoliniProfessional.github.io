# Notification Repricing: Faster, Stronger, Better

**November 24, 2025**

In my previous post, I introduced the foundations of my intelligent Amazon repricing system — a fully custom tool designed to make contextual pricing decisions using competitive data, fee modeling, and LLM-driven logic. Since then, the system has evolved significantly, and this update covers one of the most transformative developments so far: **notification-based repricing.**

## The Traditional Approach: Batch Repricing

For most of the project, I used the traditional repricing model that many sellers still rely on today: the **getItemOffersBatch** endpoint. This API returns a comprehensive snapshot of the marketplace for each ASIN in my active inventory, including:

- Buy Box price and owner  
- Lowest FBA, FBM, and featured merchant offers  
- Condition-specific price ladders  
- Prime eligibility  
- Shipping-inclusive pricing  
- Total offer count and seller types  
- Offer metadata (feedback rating, fulfillment type, etc.)

Combined with a fast batch architecture, the system continuously cycled through ~1,500 SKUs and typically identified **only 20–25 items requiring a price update at any given time**.  
It was efficient, stable, and extremely effective — the kind of predictable behavior required for production-grade automation.

## The 2026 SP-API Announcement

In late 2025, Amazon announced upcoming SP-API changes scheduled for 2026, including a new fee structure that meters **GET** requests above a certain threshold.  
At the time, batch pricing appeared vulnerable: getItemOffersBatch *conceptually* performs multiple GET operations inside each payload. My usage charts showed hundreds of thousands of batch calls, and it initially looked like the new pricing model could make batch repricing expensive.

Rather than wait and see, I decided to engineer around the problem.

## Taking the Leap: Notification-Based Repricing

This led me to build a second repricing engine, one that uses Amazon’s **notification system** through AWS.  
I had avoided AWS for a long time — not because it is difficult, but because it’s broad and easy to overcomplicate.  
But this challenge created the perfect opportunity.

Notification repricing works by subscribing to Amazon’s competitive pricing events and wiring them into an AWS pipeline:

- **SNS** receives Amazon’s event streams  
- **SQS** buffers incoming pricing updates  
- A local service or Lambda function processes each event  
- My repricer reacts immediately to competitive changes  
- Updated pricing decisions flow through the same feed-based publishing pipeline

This architecture shifts repricing from a **polling model** to a **reactive, event-driven model**.  
As soon as an offer changes, my system knows — and can respond — instantly.

## The Plot Twist: Batch Repricing Isn’t Metered

After building the notification pipeline, I revisited Amazon’s documentation and confirmed something surprising:

**getItemOffersBatch is a POST endpoint, not a GET endpoint.**

Under the 2026 model, **POST, PUT, and PATCH calls remain unmetered and unlimited**.  
Meaning the batch repricing system I had already built — the one I thought would become expensive — actually incurs **zero metered charges**.

Instead of a problem, I suddenly had an opportunity.

## A Hybrid Architecture: The Best of Both Worlds

Rather than choosing between batch or notification repricing, I adopted both.  
Running them in tandem creates a hybrid system with significant advantages:

### 1. Real-Time Responsiveness (Notifications)
Instant reaction to:
- Buy Box shifts  
- Sudden undercutting  
- Competitive entries and exits  
- Prime status changes  
- Price collapses and spikes  

### 2. Full-Context Validation (Batch Pricing)
Batch repricing still provides the richest dataset available through SP-API and ensures:
- Contextually correct decisions  
- Price stability  
- Lower error rates  
- Strategic consistency  

### 3. Zero Wasted Updates
Before sending any price change to Amazon, the system logs the event and confirms whether the proposed price is still needed.  
This avoids unnecessary feed updates and keeps pricing behavior tight and deliberate.

### 4. Scalable Engineering
A hybrid repricer is inherently more resilient—ideal for future scaling into a multi-tenant SaaS platform.

## Conclusion

What began as a response to a potential API cost issue became a major architectural enhancement. By building a full notification-driven repricing engine and combining it with my existing batch system, I’ve ended up with something far more powerful:

**A faster, stronger, and more intelligent repricing system — built on both real-time events and deep contextual analysis.**

---

Thank you for following along on this journey. Stay tuned for more technical deep-dives and development updates!

**[← Back to Home](index.md)**

---

**Connect with me:** [LinkedIn](https://www.linkedin.com/in/luca-zanolini-a18654392/) | [GitHub](https://github.com/LucaZanoliniProfessional) | lucazanolini.pro@gmail.com