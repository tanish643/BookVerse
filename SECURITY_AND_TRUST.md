# 🛡️ Security & Trust Architecture

This document details the specific technical measures implemented to ensure platform security and build user trust. Use this for your "Security" and "Future Scope" slides.

## 🔒 1. Security Implementations (Technical)

We adopted a **"Defense in Depth"** strategy, securing the application at the Database, API, and Application layers.

### A. Database Security: Row Level Security (RLS)
Instead of relying on backend code to check permissions, we enforce security **directly in the database engine (PostgreSQL)**.
*   **What we did:** We enabled RLS on all tables (`profiles`, `listings`, `messages`, etc.).
*   **How it works:** Every query is automatically filtered.
    *   *Example:* `CREATE POLICY "Private Messages" ON messages USING (auth.uid() = sender_id OR auth.uid() = receiver_id);`
    *   *Result:* Even if a hacker tries to query `SELECT * FROM messages`, the database only returns their own messages. They physically cannot see others' data.

### B. Secure Handover Protocol (Anti-Fraud)
To secure physical P2P transactions, we built a cryptographic verification system.
*   **The Problem:** In local meetups, a seller might mark an item "Sold" without giving it, or a buyer might claim they never received it.
*   **The Solution:**
    1.  **Code Generation:** When a transaction starts, the server generates a random 6-digit code.
    2.  **Secrecy:** This code is stored in the database but is **only selectable by the Buyer** (via RLS policies). The Seller cannot fetch it.
    3.  **Atomic Verification:** We use a secure RPC function (`verify_handover`). It checks the code and updates the transaction status in a single atomic operation. If the code is wrong, the transaction fails.

### C. Authentication & Session Management
*   **Supabase Auth:** We use industry-standard JWT (JSON Web Tokens) for session management.
*   **Password Security:** Passwords are never stored in plain text. They are hashed using **Bcrypt** by Supabase's GoTrue server.
*   **Protection:** All API requests require a valid, unexpired Bearer Token.

### D. Secure RPC Functions
*   For sensitive operations (like calculating Trust Scores or verifying codes), we use **PostgreSQL Functions** (`SECURITY DEFINER`).
*   This prevents client-side manipulation. A user cannot simply send a command like `UPDATE trust_score SET score = 100`. They must call the function, which runs the strict calculation logic on the server.

---

## 🤝 2. Trust & Credibility Features (User-Facing)

Security protects data; Trust encourages interaction. Here is how we designed for Trust:

### A. The "Trust Score" Algorithm
We don't just use simple 5-star ratings. We calculate a weighted **Reputation Score**.
*   **Formula:** `(Avg Rating * Weight) + (Verified Transactions * Weight) + (Account Age)`
*   **Why:** This prevents a new user with one 5-star review from looking as trustworthy as a veteran user with fifty 4.8-star reviews.
*   **Implementation:** A database trigger automatically recalculates this score whenever a transaction completes or a review is posted.

### B. Verified Purchase Badges
*   **The Problem:** Fake reviews plague e-commerce.
*   **The Solution:** We link reviews directly to `transaction` records in the database.
*   **Result:** If a review is linked to a completed transaction ID, the UI displays a **"Verified Purchase"** badge. This is un-fakeable proof that the user actually bought the book.

### C. Community Moderation
*   **Reporting System:** We implemented a `reports` table and UI dialogs allowing users to flag inappropriate listings, abusive chat messages, or suspicious profiles.
*   **Transparency:** User profiles display their join date and activity history, giving potential buyers context before they engage.

### D. Escrow-Ready Architecture (Future)
*   While currently cash-on-delivery, the `transactions` table is designed with states (`pending`, `confirmed`, `completed`) that perfectly map to an Escrow system (like Stripe Connect) for future payment security.
