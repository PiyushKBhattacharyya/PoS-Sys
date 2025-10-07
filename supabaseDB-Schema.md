# Supabase Database Schema

This document outlines the detailed Supabase database schema, reflecting the use of Telegram for messaging.

### 1. Customers Table
- **Table name:** `customers`
- **Primary keys:** `customer_id`
- **Column names:**
    - `customer_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `phone_number`: `text` (UNIQUE, NOT NULL)
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Constraints:**
    - `phone_number` must be unique.

### 2. Orders Table
- **Table name:** `orders`
- **Primary keys:** `order_id`
- **Foreign keys:** `customer_id` (references `customers.customer_id`)
- **Column names:**
    - `order_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `customer_id`: `uuid` (Foreign Key, NOT NULL)
    - `order_timestamp`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `outlet_name`: `text` (NOT NULL)
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Relationships:**
    - `orders.customer_id` -> `customers.customer_id` (Many-to-One: many orders can belong to one customer)

### 3. Feedback Table
- **Table name:** `feedback`
- **Primary keys:** `feedback_id`
- **Foreign keys:** `customer_id` (references `customers.customer_id`), `order_id` (references `orders.order_id`)
- **Column names:**
    - `feedback_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `customer_id`: `uuid` (Foreign Key, NOT NULL)
    - `order_id`: `uuid` (Foreign Key, NOT NULL)
    - `rating`: `integer` (NOT NULL, CHECK (`rating` >= 1 AND `rating` <= 5))
    - `feedback_text`: `text`
    - `submitted_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Relationships:**
    - `feedback.customer_id` -> `customers.customer_id` (Many-to-One)
    - `feedback.order_id` -> `orders.order_id` (One-to-One: one feedback per order)

### 4. Telegram_Messages Table
- **Table name:** `telegram_messages`
- **Primary keys:** `message_id`
- **Foreign keys:** `customer_id` (references `customers.customer_id`), `feedback_id` (references `feedback.feedback_id`, NULLABLE)
- **Column names:**
    - `message_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `customer_id`: `uuid` (Foreign Key, NOT NULL)
    - `feedback_id`: `uuid` (Foreign Key, NULLABLE)
    - `message_type`: `text` (e.g., 'initial_feedback', 'feedback_reminder_day2', 'feedback_reminder_day7', 'review_reminder', 'offer_message')
    - `content`: `text` (NOT NULL)
    - `sent_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `status`: `text` (e.g., 'sent', 'delivered', 'read', 'failed')
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Relationships:**
    - `telegram_messages.customer_id` -> `customers.customer_id` (Many-to-One)
    - `telegram_messages.feedback_id` -> `feedback.feedback_id` (Many-to-One: multiple messages can relate to one feedback, e.g., reminders)

### 5. Google_Reviews Table
- **Table name:** `google_reviews`
- **Primary keys:** `review_id`
- **Foreign keys:** `feedback_id` (references `feedback.feedback_id`)
- **Column names:**
    - `review_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `feedback_id`: `uuid` (Foreign Key, UNIQUE, NOT NULL)
    - `review_link_clicked`: `boolean` (NOT NULL, DEFAULT `false`)
    - `review_text`: `text`
    - `auto_reply_text`: `text`
    - `submitted_at`: `timestamp with time zone`
    - `replied_at`: `timestamp with time zone`
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Constraints:**
    - `feedback_id` must be unique to ensure one Google review per feedback.
- **Relationships:**
    - `google_reviews.feedback_id` -> `feedback.feedback_id` (One-to-One)

### 6. Offers Table
- **Table name:** `offers`
- **Primary keys:** `offer_id`
- **Foreign keys:** `customer_id` (references `customers.customer_id`), `feedback_id` (references `feedback.feedback_id`, NULLABLE)
- **Column names:**
    - `offer_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `customer_id`: `uuid` (Foreign Key, NOT NULL)
    - `feedback_id`: `uuid` (Foreign Key, NULLABLE)
    - `offer_details`: `text` (e.g., "10% off next purchase", "Free dessert")
    - `sent_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `redeemed`: `boolean` (NOT NULL, DEFAULT `false`)
    - `redeemed_at`: `timestamp with time zone`
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Relationships:**
    - `offers.customer_id` -> `customers.customer_id` (Many-to-One)
    - `offers.feedback_id` -> `feedback.feedback_id` (Many-to-One: an offer can be sent based on a specific feedback)

### 7. Daily_Summaries_Insights Table
- **Table name:** `daily_summaries_insights`
- **Primary keys:** `summary_id`
- **Column names:**
    - `summary_id`: `uuid` (Primary Key, NOT NULL, DEFAULT `gen_random_uuid()`)
    - `summary_date`: `date` (NOT NULL, UNIQUE)
    - `summary_text`: `text` (NOT NULL)
    - `insights_text`: `text`
    - `generated_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `created_at`: `timestamp with time zone` (NOT NULL, DEFAULT `now()`)
    - `updated_at`: `timestamp with time zone` (DEFAULT `now()`)
- **Constraints:**
    - `summary_date` must be unique.