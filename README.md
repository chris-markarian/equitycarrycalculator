# Real Estate Underwriting Calculator, created by Chris Markarian at markarian.ai

A comprehensive real estate deal analysis tool with seller financing, equity carry structures, and cloud sync capabilities.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Platform](https://img.shields.io/badge/platform-web-brightgreen.svg)

## 🏠 Overview

This calculator helps real estate investors analyze deals using creative financing strategies, specifically **equity carry** and **capital stacking** techniques. It provides instant calculations for cash flow, cap rates, blended interest rates, and more.

### Key Features

- **Deal Analysis** - Complete underwriting with NOI, cap rate, cash-on-cash return, and spread calculations
- **Seller Financing** - Model seller carry terms including interest rates, amortization, and balloon payments
- **Capital Stacking** - Combine first position loans with seller financing to maximize leverage
- **Cloud Sync** - Save deals to the cloud and access them from any device
- **Team Collaboration** - Collision detection alerts when multiple team members work the same property
- **PDF Export** - Generate professional deal summaries
- **LOI Generator** - Create Letters of Intent with deal terms pre-filled
- **AI Prompt Generator** - Generate prompts for AI-assisted market analysis

## 🚀 Live Demo

Access the calculator at: [https://chris-markarian.github.io/equitycarrycalculator/](https://chris-markarian.github.io/equitycarrycalculator/)

## 📊 Calculations Included

| Category | Metrics |
|----------|---------|
| **Returns** | Cap Rate, Cash-on-Cash Return, Blended Interest Rate, Spread |
| **Cash Flow** | NOI, Annual/Monthly Cash Flow, CF to Purchase Price |
| **Financing** | First Position Loan, Seller Carry Amount, Total Financing %, Down Payment |
| **Expenses** | Property Tax, Insurance, HOA, CapEx, Management, Vacancy |
| **Projections** | Value at Balloon, Equity at Balloon, Remaining Balances |
| **Deal Structure** | Cash to Seller at Closing, Cash to Buyer, Agent Fees, Closing Costs |

## 🔧 Setup

### Basic Usage (No Cloud Sync)

Simply open `index.html` in a web browser. All features work except cloud sync.

### Full Setup with Cloud Sync

1. **Create a Supabase Account**
   - Go to [supabase.com](https://supabase.com) and create a free account
   - Create a new project

2. **Run Database Setup**
   - In Supabase, go to **SQL Editor** → **New Query**
   - Paste and run the following SQL:

```sql
-- Create the deals table
CREATE TABLE IF NOT EXISTS deals (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    street_address TEXT,
    city TEXT,
    state TEXT,
    zip TEXT,
    property_link TEXT,
    preparer_name TEXT,
    preparer_email TEXT,
    preparer_phone TEXT,
    seller_name TEXT,
    seller_email TEXT,
    seller_phone TEXT,
    category TEXT DEFAULT 'DSCR Residential',
    first_loan_ltv NUMERIC DEFAULT 26,
    seller_carry_pct NUMERIC DEFAULT 80,
    target_cash_to_seller NUMERIC,
    interest_only TEXT DEFAULT 'No',
    interest_at_balloon TEXT DEFAULT 'No',
    months_to_stabilize INTEGER DEFAULT 0,
    revenue_during_stab NUMERIC DEFAULT 100,
    years_to_balloon INTEGER DEFAULT 8,
    purchase_price NUMERIC DEFAULT 0,
    asking_price NUMERIC DEFAULT 0,
    rental_revenue NUMERIC DEFAULT 0,
    appreciation_rate NUMERIC DEFAULT 2,
    agent_fee NUMERIC DEFAULT 0,
    rehab_cost NUMERIC DEFAULT 0,
    assignment_fee_pct NUMERIC DEFAULT 5,
    appraisal_cost NUMERIC DEFAULT 0,
    closing_cost_pct NUMERIC DEFAULT 5,
    seller_carry_rate NUMERIC DEFAULT 0,
    seller_carry_amort INTEGER DEFAULT 30,
    property_tax NUMERIC DEFAULT 0,
    insurance NUMERIC DEFAULT 0,
    hoa NUMERIC DEFAULT 0,
    other_expenses NUMERIC DEFAULT 0,
    capex_pct NUMERIC DEFAULT 5,
    management_pct NUMERIC DEFAULT 8,
    vacancy_pct NUMERIC DEFAULT 5,
    full_data JSONB
);

-- Create indexes
CREATE INDEX IF NOT EXISTS idx_deals_address ON deals (LOWER(street_address), LOWER(city), LOWER(state));
CREATE INDEX IF NOT EXISTS idx_deals_user_id ON deals (user_id);

-- Enable Row Level Security
ALTER TABLE deals ENABLE ROW LEVEL SECURITY;

-- Create policies
DROP POLICY IF EXISTS "Users can view own deals" ON deals;
DROP POLICY IF EXISTS "Users can insert own deals" ON deals;
DROP POLICY IF EXISTS "Users can update own deals" ON deals;
DROP POLICY IF EXISTS "Users can delete own deals" ON deals;

CREATE POLICY "Users can view own deals" ON deals FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can insert own deals" ON deals FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update own deals" ON deals FOR UPDATE USING (auth.uid() = user_id);
CREATE POLICY "Users can delete own deals" ON deals FOR DELETE USING (auth.uid() = user_id);

-- Create collision detection function
CREATE OR REPLACE FUNCTION check_address_collision(
    p_street_address TEXT,
    p_city TEXT,
    p_state TEXT,
    p_exclude_user_id UUID DEFAULT NULL
)
RETURNS TABLE (user_email TEXT, created_at TIMESTAMP WITH TIME ZONE)
SECURITY DEFINER AS $$
BEGIN
    RETURN QUERY
    SELECT u.email::TEXT, d.created_at
    FROM deals d
    JOIN auth.users u ON d.user_id = u.id
    WHERE LOWER(TRIM(d.street_address)) = LOWER(TRIM(p_street_address))
      AND LOWER(TRIM(d.city)) = LOWER(TRIM(p_city))
      AND LOWER(TRIM(d.state)) = LOWER(TRIM(p_state))
      AND (p_exclude_user_id IS NULL OR d.user_id != p_exclude_user_id)
    LIMIT 5;
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION check_address_collision TO authenticated;
```

3. **Configure Authentication**
   - Go to **Authentication** → **Providers** → **Email**
   - (Optional) Toggle off "Confirm email" for easier onboarding

4. **Update the App**
   - In `index.html`, update the Supabase credentials (around line 2355):
   ```javascript
   const SUPABASE_URL = 'https://your-project.supabase.co';
   const SUPABASE_ANON_KEY = 'your-anon-key';
   ```

5. **Deploy**
   - Host on GitHub Pages, Netlify, Vercel, or any static hosting

## 📱 Usage

### Analyzing a Deal

1. Enter the property address and details
2. Configure loan terms (LTV, seller carry %)
3. Input purchase price and expected rental revenue
4. Add operating expenses
5. Review calculated metrics in the dashboard

### Saving Deals

1. Sign up / log in with your email
2. Fill in deal details
3. Click **"Save Deal to Library"**
4. Access your deals from any device by logging in

### Generating Documents

- **PDF Summary** - Click "PDF Summary" for a printable deal overview
- **LOI** - Click "Generate LOI" to create a Letter of Intent

## 🏗️ Architecture

```
┌─────────────────┐     ┌─────────────────┐
│   Browser       │────▶│  GitHub Pages   │
│   (Any Device)  │     │  (Static Host)  │
└─────────────────┘     └─────────────────┘
                               │
                               ▼
                        ┌─────────────────┐
                        │    Supabase     │
                        │  ┌───────────┐  │
                        │  │   Auth    │  │
                        │  ├───────────┤  │
                        │  │ Database  │  │
                        │  │ (Postgres)│  │
                        │  └───────────┘  │
                        └─────────────────┘
```

## 🔒 Security

- **Row Level Security (RLS)** - Users can only access their own deals
- **Secure Authentication** - Handled by Supabase Auth
- **No Server Required** - Runs entirely client-side with Supabase backend

## 💰 Cost

- **Supabase Free Tier** includes:
  - 500 MB database storage (~166,000 deals)
  - 50,000 monthly active users
  - Unlimited API requests

For typical usage (20-100 users), you'll stay well within the free tier.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with [Supabase](https://supabase.com) for backend services
- Inspired by creative real estate financing strategies

---

**Questions?** Open an issue or reach out to Chris M at chris@markarian.ai
