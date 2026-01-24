# Trust & Credibility Features - Implementation Guide

## 🎉 Features Implemented

### ✅ User Ratings & Reviews
- **User Profile Ratings**: Users now have an aggregate rating displayed on their profile.
- **Transaction Reviews**: Users can review each other after transactions (schema support added).
- **Reviews Tab**: A new tab on the profile page lists all reviews received by the user.

### ✅ Enhanced Book Reviews
- **Photo Attachments**: Users can now upload up to 3 photos with their book reviews.
- **Helpful Voting**: Community members can vote "Helpful" or "Unhelpful" on reviews.
- **Visual Updates**: Reviews now display attached images in a lightbox-friendly format (opens in new tab).

### ✅ Trust Infrastructure (Database)
- **Transactions Table**: Tracks exchanges between buyers and sellers.
- **User Reviews Table**: Stores ratings and comments for users.
- **Review Votes Table**: Tracks community votes on book reviews.
- **Storage**: Dedicated bucket for review images.

---

## 📋 Setup Instructions

### Step 1: Run Database Migration

**IMPORTANT**: You must run the SQL migration to create the new tables and storage buckets.

1. Open your **Supabase Dashboard**
2. Navigate to **SQL Editor**
3. Open the file: `supabase/migrations/20251123_trust_features.sql`
4. Copy the entire contents
5. Paste into Supabase SQL Editor
6. Click **Run**

This will create:
- `transactions` table
- `user_reviews` table
- `book_review_votes` table
- `review-images` storage bucket
- RLS policies for security

---

## 🚀 How to Use

### For Users

1. **Writing a Review with Photos**:
   - Go to any book detail page
   - Click "Reviews" tab
   - Click the image icon to select photos
   - Submit your review

2. **Voting on Reviews**:
   - Browse reviews on a book page
   - Click the "Thumbs Up" or "Thumbs Down" icon to vote
   - See helpful counts update in real-time

3. **Viewing User Reputation**:
   - Go to your Profile page
   - See your star rating under your username
   - Click "Reviews" tab to see what others said about you

### For Developers

**Fetching User Ratings**:
```typescript
const { data: reviews } = await supabase
  .from('user_reviews')
  .select('rating')
  .eq('reviewed_user_id', userId);

const average = reviews.reduce((acc, r) => acc + r.rating, 0) / reviews.length;
```

**Handling Votes**:
```typescript
// Vote helpful
await supabase
  .from('book_review_votes')
  .upsert({ review_id, user_id, vote_type: 'helpful' });
```

---

## 🔮 Future Enhancements

1. **Transaction Flow**: Implement "Mark as Sold" to populate the `transactions` table.
2. **Verified Badges**: Show "Verified Purchase" on reviews linked to a transaction.
3. **Review Moderation**: Admin tools to moderate reported reviews.
