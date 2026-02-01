# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Muzik.AI** - AI music generation SaaS platform. Users generate songs with lyrics using AI. Core stack: Next.js 16, Supabase, Eachlabs Mureka V8, Stripe.

## Development Commands

```bash
npm run dev      # Start development server on http://localhost:3000
npm run build    # Build for production
npm run start    # Start production server
npm run lint     # Run ESLint
```

## Project Structure

```
src/
├── app/              # Next.js App Router pages
├── components/       # React components
│   └── ui/           # Shadcn UI components
├── lib/              # Utilities and integrations
│   ├── supabase/     # Supabase client
│   ├── eachlabs/     # Eachlabs API integration
│   ├── stripe/       # Stripe integration
│   └── openai/       # ChatGPT for lyrics generation
└── types/            # TypeScript types
```

## Code Conventions

- **Path aliases**: `@/components`, `@/lib/utils`, `@/components/ui`, `@/lib`, `@/hooks`
- **CSS variables**: Colors in `src/app/globals.css` (OKLCH, light/dark mode)
- **Component styling**: Tailwind + `cn()` for class merging
- **Shadcn style**: "new-york" style components
- **Dark mode**: Toggle with `.dark` class on HTML element

## External Services

### Eachlabs API (Mureka V8)
- **Endpoint**: `https://api.eachlabs.ai/v1/prediction/`
- **Auth**: `X-API-Key` header
- **Model**: `mureka-generate-song` with `mureka-8`
- **Input**: `lyrics` (required), `prompt` (optional), `model`, `n` (1-3)
- **Polling**: Get result with predictionID, ~65s avg response
- **Output**: URL (MP3/WAV/FLAC, valid 30 days)

### Supabase
- **Auth**: Email/password + Google OAuth
- **Database**: PostgreSQL with RLS policies
- **Tables**: `users`, `generations`, `payments`, `credits_history`

### Stripe
- **Products**: Pro Monthly ($9.99), Pro Yearly ($79.99)
- **Credits**: Free (2), Pro (50/month)
- **Webhooks**: Payment success/fail handling

## Credits System

| Plan | Credits | Price |
|------|---------|-------|
| Free | 2 | $0 |
| Pro Monthly | 50 | $9.99/mo |
| Pro Yearly | 600 | $79.99/yr |

- Eachlabs cost: ~$0.09 per 2.5min song
- Credit deducted on generation start

## Database Schema (Supabase)

```sql
users: id, email, full_name, avatar_url, plan, credits_remaining, stripe_customer_id, stripe_subscription_id, created_at

generations: id, user_id, prediction_id, lyrics, prompt, output_url, format, duration_seconds, model_version, song_title, genre_tags, status, created_at

payments: id, user_id, stripe_payment_id, amount, currency, status, plan_type, credits_granted, created_at

credits_history: id, user_id, amount, reason, generation_id, created_at
```

## User Flow

1. Sign Up (Email/Google) → 2 free credits
2. Dashboard → Credits balance, recent songs
3. Create Song → Genre/Mood/Theme → Lyrics (AI or manual) → Generate
4. Processing (65s) → Result (Player, Download, Save to Library)
5. Library → View/Delete all generated songs

## API Routes

```
/api/auth/*         → Supabase Auth
/api/generate       → POST Create Eachlabs prediction
/api/generate/poll  → GET Poll prediction result
/api/generate/webhook → Eachlabs webhook (optional)
/api/lyrics         → POST Generate lyrics with ChatGPT
/api/library        → GET/DELETE User generations
/api/user/credits   → GET User credit balance
/api/subscription/* → Stripe checkout/webhook/portal
```

## Key Dependencies

- **@radix-ui/react-*** - Accessible UI primitives
- **@supabase/supabase-js** - Supabase client
- **class-variance-authority** - Component variants
- **lucide-react** - Icons
- **stripe** - Payment processing
- **openai** - ChatGPT for lyrics
- **zod + react-hook-form** - Form validation
