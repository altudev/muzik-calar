# PRD: Muzik.AI - AI Music Generation Platform

**Version:** 1.0
**Status:** Draft
**Date:** February 2025

---

## 1. Executive Summary

Muzik.AI, içerik üreticileri, pazarlama ekipleri ve müzik hobilerine sahip kişilerin yapay zeka ile kendi şarkılarını oluşturabileceği bir SaaS platformudur. Kullanıcılar; genre, mood ve tema girdikten sonra ister AI ile lirik oluşturur isterse kendi liriklerini yazar. Mureka (Eachlabs) modeli ile profesyonel kalitede, vokalli şarkılar üretilir.

**Rakipler:** Suno, Udio, Mubert
**Farkımız:** Basit workflow + Lyrics-first yaklaşım + Credits tabanlı esnek model

---

## 2. Goals & Non-Goals

### Goals
- 65 saniye içinde profesyonel kalitede şarkı üretimi
- Kullanıcı dostu lyrics generation flow
- Supabase + Stripe entegrasyonu ile tam self-servis platform
- Aylık/yıllık subscription modeli

### Non-Goals
- Voice cloning (v1'de değil)
- Stem separation/remix (v1'de değil)
- Collaborative editing (v1'de değil)
- Mobile app (web first, mobile responsive)

---

## 3. User Personas

| Persona | Motivasyon | Pain Points |
|---------|------------|-------------|
| **Content Creator** | YouTube/TikTok için hızlı, telifsiz müzik | Müzik lisanslama karmaşık, pahalı |
| **Marketing Team** | Reklamlar/kampanyalar için özel jingle | Stok müzik yetersiz, benzersizlik gerekli |
| **Hobbyist** | Eğlence amaçlı müzik yapmak | DAW öğrenmek zor, teknik bilgi gerekiyor |

---

## 4. User Stories

```
AS A [Content Creator]
I WANT TO generate a custom song with my own lyrics in 2 minutes
SO THAT I can add unique music to my YouTube videos

AS A [Marketing Manager]
I WANT TO create a branded jingle matching my campaign mood
SO THAT my brand has a distinctive audio identity

AS A [Hobbyist]
I WANT TO experiment with different genres and lyrics
SO THAT I can explore my creativity without expensive tools
```

---

## 5. Functional Requirements

### 5.1 Authentication
- [ ] Email/password registration & login
- [ ] Google OAuth login
- [ ] Session management (Supabase Auth)
- [ ] Password reset flow

### 5.2 Dashboard
- [ ] Credits balance gösterimi (Free: 2, Pro: 50)
- [ ] Son oluşturulan şarkılar (max 5)
- [ ] "Yeni Şarkı Oluştur" CTA
- [ ] Plan bilgisi (Free/Pro badge)

### 5.3 Song Creation Flow
- [ ] Genre/Mood/Theme input field
- [ ] Lyrics toggle: "AI ile oluştur" vs "Kendi liriklerim"
- [ ] Lyrics Editor (textarea, editable)
- [ ] Additional Style Prompt (optional)
- [ ] Generate butonu (credits kontrolü)
- [ ] Loading state (65s countdown, progress)
- [ ] Result Player (HTML5 audio)
- [ ] Download butonu (MP3/WAV/FLAC seçimi)
- [ ] "Library'ye kaydet" butonu

### 5.4 Lyrics AI Generation
- [ ] ChatGPT API entegrasyonu
- [ ] Prompt: "Genre + Mood + Theme + [Verse/Chorus structure]"
- [ ] 3-4 verse + chorus yapısında output
- [ ] Kullanıcı edit yapabilir

### 5.5 Library
- [ ] Tarihe göre sıralı liste
- [ ] Her şarkıda:
  - Şarkı adı (otomatik veya manuel)
  - Genre tag
  - Prompt snippet
  - Lyrics preview
  - Oluşturma tarihi
  - Model version
- [ ] Filtre: Tarih, Genre
- [ ] Search: Prompt/lyrics içinde ara
- [ ] Delete butonu (confirmation)
- [ ] Play, Download

### 5.6 Credits System
- [ ] Free plan: 2 credits (one-time)
- [ ] Pro plan: 50 credits/ay
- [ ] Credit balance real-time güncelleme
- [ ] "Yetersiz kredi" uyarısı
- [ ] Credit deduction on generation start

### 5.7 Subscription (Stripe)
- [ ] Free plan (2 credits, lifetime)
- [ ] Pro Monthly
- [ ] Pro Yearly (discount)
- [ ] Cancel subscription flow
- [ ] Payment success/fail webhook
- [ ] Customer portal (Stripe)

---

## 6. Technical Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Frontend (Next.js)                      │
│  ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌──────────────┐   │
│  │ Auth    │  │ Dashboard│  │ Creator │  │ Library      │   │
│  │ (Google)│  │          │  │ Flow    │  │              │   │
│  └─────────┘  └──────────┘  └─────────┘  └──────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │ API Routes
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    Backend (Next.js API)                     │
│  ┌──────────────┐  ┌─────────────┐  ┌───────────────────┐   │
│  │ Eachlabs     │  │ Stripe      │  │ Supabase          │   │
│  │ Integration  │  │ Webhooks    │  │ (Auth + DB)       │   │
│  └──────────────┘  └─────────────┘  └───────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│ Eachlabs.ai │  │ Stripe.com   │  │ Supabase     │
│ (Mureka V8) │  │ (Payments)   │  │ (PostgreSQL) │
└─────────────┘  └──────────────┘  └──────────────┘
```

---

## 7. Eachlabs API Integration

### 7.1 Endpoint: Create Prediction
```
POST https://api.eachlabs.ai/v1/prediction/

{
  "model": "mureka-generate-song",
  "version": "0.0.1",
  "input": {
    "model": "mureka-8",
    "n": 1,
    "lyrics": "[verse 1]\n...\n[chorus]\n...",
    "prompt": "Genre, mood, tempo, instruments, structure"
  },
  "webhook_url": "" // Optional
}
```

### 7.2 Endpoint: Poll Result
```
GET https://api.eachlabs.ai/v1/prediction/{prediction_id}

Response (success):
{
  "status": "success",
  "output": "https://output-url.com/result.mp3",
  "metrics": { "predict_time": 65.3 }
}
```

### 7.3 Parameters
| Param | Type | Required | Default | Notes |
|-------|------|----------|---------|-------|
| `lyrics` | string | Yes | - | With [verse], [chorus] tags |
| `prompt` | string | No | auto | "Genre + mood + tempo + instruments" |
| `model` | string | No | auto | Use "mureka-8" for best quality |
| `n` | int | No | 2 | Max 3, charges per song |

### 7.4 Rate Limits
- 100 requests/minute
- 10 concurrent predictions

---

## 8. User Flows

### 8.1 Sign Up Flow
```
Landing Page
    ↓
"Sign Up" → Email/Google
    ↓
Email Verification (if email)
    ↓
Welcome Screen → "2 Credits'in hazır!"
    ↓
Dashboard
```

### 8.2 Create Song Flow
```
Dashboard → "Yeni Şarkı"
    ↓
Step 1: Ne istiyorsun?
├─ Genre/Mood/Theme (text input)
└─ Style Notes (optional)
    ↓
Step 2: Şarkı Sözleri
├─ "AI ile oluştur" → ChatGPT call → Lyrics Editor
└─ "Kendi liriklerim yazacağım" → Lyrics Editor
    ↓
Step 3: Preview & Generate
├─ Lyrics preview
├─ Prompt preview
└─ "Şarkı Oluştur (1 Credit)"
    ↓
Step 4: Processing (65s)
├─ Progress bar
├─ "Mureka çalışıyor..."
└─ Cancel option
    ↓
Step 5: Result
├─ Audio Player
├─ Download (MP3/WAV/FLAC)
├─ "Library'ye Ekle"
└─ "Yeni Şarkı Oluştur"
```

### 8.3 Subscription Flow
```
Dashboard → "Upgrade to Pro"
    ↓
Pricing Page (Monthly/Yearly)
    ↓
Stripe Checkout
    ↓
Payment Success → Credits: 50
    ↓
Dashboard (Pro badge)
```

### 8.4 Library Flow
```
Dashboard → "Library"
    ↓
Song List (chronological)
├─ Search by prompt/lyrics
├─ Filter by genre/date
├─ Play → Modal player
├─ Download
└─ Delete → Confirmation
    ↓
Empty State → "İlk şarkını oluştur!"
```

---

## 9. Data Model (Supabase)

### 9.1 Tables

```sql
-- users (extends Supabase auth.users)
CREATE TABLE users (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  email TEXT NOT NULL,
  full_name TEXT,
  avatar_url TEXT,
  plan TEXT DEFAULT 'free', -- 'free' | 'pro_monthly' | 'pro_yearly'
  credits_remaining INTEGER DEFAULT 2,
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- generations
CREATE TABLE generations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  prediction_id TEXT NOT NULL,
  lyrics TEXT NOT NULL,
  prompt TEXT,
  output_url TEXT NOT NULL,
  format TEXT, -- 'mp3' | 'wav' | 'flac'
  duration_seconds INTEGER,
  model_version TEXT DEFAULT 'mureka-8',
  song_title TEXT,
  genre_tags TEXT[],
  status TEXT DEFAULT 'processing', -- 'processing' | 'success' | 'failed'
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- payments (Stripe webhook log)
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  stripe_payment_id TEXT NOT NULL,
  amount INTEGER NOT NULL, -- in cents
  currency TEXT DEFAULT 'usd',
  status TEXT, -- 'pending' | 'succeeded' | 'failed'
  plan_type TEXT, -- 'monthly' | 'yearly'
  credits_granted INTEGER NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- credits_history
CREATE TABLE credits_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  amount INTEGER NOT NULL, -- + for earned, - for spent
  reason TEXT NOT NULL, -- 'signup_bonus' | 'subscription' | 'generation'
  generation_id UUID REFERENCES generations(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 9.2 RLS Policies
- Users can only read/update their own data
- Service role for admin operations

---

## 10. Pricing & Credits

### 10.1 Credit Economics
| Metric | Value |
|--------|-------|
| Eachlabs cost/2.5min | $0.09 |
| Eachlabs cost/6min | ~$0.22 |
| Free plan (2 credits) | ~$0.18 cost |
| Pro plan (50 credits) | ~$2.25/month cost |
| Target margin | ~80% |

### 10.2 Pricing Tiers (Draft)
| Plan | Credits | Price | Margin (~$0.09/song) |
|------|---------|-------|----------------------|
| Free | 2 | $0 | - |
| Pro Monthly | 50 | $9.99/mo | ~45% |
| Pro Yearly | 600 | $79.99/yr (~$6.66/mo) | ~85% |

*Not: Pricing ayarlanabilir, Eachlabs bulk discount olabilir.*

---

## 11. API Routes (Next.js)

```
/api/auth/*         → Supabase Auth
/api/generate       → Eachlabs prediction create
/api/generate/poll  → Eachlabs prediction poll
/api/generate/webhook → Eachlabs webhook handler
/api/lyrics         → ChatGPT lyrics generation
/api/library        → CRUD operations
/api/user/credits   → Get/balance
/api/subscription/* → Stripe checkout/webhook/portal
```

---

## 12. Milestones

### Phase 1: MVP (Sprint 1-2)
- [ ] Next.js + Supabase setup
- [ ] Auth (email + Google)
- [ ] Eachlabs API integration
- [ ] Generate flow (simple)
- [ ] Basic library (no metadata)
- [ ] Free plan (2 credits)

### Phase 2: Polish (Sprint 3)
- [ ] AI lyrics generation (ChatGPT)
- [ ] Loading states & error handling
- [ ] Library full features (metadata, search, filter)
- [ ] Delete functionality
- [ ] UI/UX polish

### Phase 3: Monetization (Sprint 4)
- [ ] Stripe integration
- [ ] Pro plan
- [ ] Customer portal
- [ ] Email receipts

### Phase 4: Launch
- [ ] Landing page
- [ ] SEO & analytics
- [ ] Error monitoring (Sentry)
- [ ] Deploy (Vercel)

---

## 13. Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Eachlabs API down | High | Retry logic, queue system, user notification |
| Credit abuse | Medium | Rate limiting, RLS policies |
| Mureka quality issues | Medium | Multiple generation option, user feedback |
| Stripe integration | Medium | Test mode first, webhooks verification |
| Cold start latency | Low | Pre-warming, optimistic UI |

---

## 14. Out of Scope (v1)

- Social features (share, collaborate)
- Mobile app
- Voice cloning
- Stem separation
- Playlist management
- API access for third parties
- White-label solution

---

## 15. Success Metrics

| Metric | Target (Launch) |
|--------|-----------------|
| Signups (Day 1) | 100 |
| Songs generated (Week 1) | 500 |
| Conversion to Pro (Month 1) | 5% |
| Avg. generation time | < 90s |
| Error rate | < 2% |

---

## Appendix A: Eachlabs API Response Codes

```
200 - Success
400 - Bad Request (invalid input)
401 - Unauthorized (invalid API key)
429 - Rate limit exceeded
500 - Internal Server Error
```

## Appendix B: Lyrics AI Prompt Template

```
System: You are a professional songwriter. Generate lyrics in {genre} style.

User: Write lyrics for a {mood} song about {theme}.
Requirements:
- Include [verse 1], [verse 2], [chorus], [bridge] tags
- 3-4 verses, 2 choruses
- Catchy, memorable lines
- Length: ~200-300 words

Output: Just the lyrics, no explanations.
```

---

**Next Steps:**
1. PRD review & approval
2. Figma wireframes
3. Database schema setup
4. MVP implementation
