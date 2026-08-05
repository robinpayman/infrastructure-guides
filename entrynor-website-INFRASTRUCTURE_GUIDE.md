# Entrynor.no Infrastructure Guide

Complete reference for where everything is stored, how it works, and how to access it.

---

## 1. Website Overview

**Domain**: entrynor.no  
**Live URL**: https://entrynor.no  
**Production Deployment**: Vercel  
**Repository**: GitHub (`robinpayman/entrynor-website`)  
**Framework**: Astro 7.1.3  
**Local Directory**: `/Users/rp/entrynor-website/minor-mercury`

### Key Features
- Static site (Astro)
- Bilingual content (Norwegian/English) with i18n routing
- Contact form with reCAPTCHA v3 and email delivery
- Responsive design with Tailwind CSS

---

## 2. Code Repository

### GitHub

**Repository**: `robinpayman/entrynor-website`  
**URL**: https://github.com/robinpayman/entrynor-website  
**Main Branch**: `main`  
**Access**: SSH via GitHub  

#### Key Branches
- `main` → deployed to Vercel (live site)
- Feature branches for development

#### Deployment Flow
```
Local changes → git commit → git push to main → Vercel auto-deploys
```

**Important**: All commits require co-author attribution:
```
Co-Authored-By: Oz <oz-agent@warp.dev>
```

---

## 3. Production Hosting

### Vercel

**Project Name**: minor-mercury  
**Live URL**: https://minor-mercury.vercel.app (also serves entrynor.no via CNAME)  
**Region**: Vercel edge network  
**Build Command**: `npm run build`  
**Output Directory**: `dist/`  
**Framework**: Astro  

#### Vercel Configuration
- **File**: `vercel.json`
- **Auto-deploys**: On push to `main` branch
- **Serverless Functions**: API routes in `/api/` directory

#### Accessing Vercel Dashboard
- **URL**: https://vercel.com
- **Project**: minor-mercury
- **Access**: GitHub OAuth (same account)

---

## 4. Contact Form Infrastructure

### Frontend
**Component**: `src/components/ContactFormI18n.astro`  
**Location**: Local → `/Users/rp/entrynor-website/minor-mercury/src/components/`

**Features**:
- Bilingual form (Norwegian/English)
- Client-side validation with specific error messages
- reCAPTCHA v3 integration
- Honeypot field for bot protection
- Dynamic error display

### Backend API

**Endpoint**: `/api/contact-form` (Vercel serverless)  
**File**: `/Users/rp/entrynor-website/minor-mercury/api/contact-form.js`  
**Runtime**: Node.js on Vercel  

**API Flow**:
1. Form submits to `/api/contact-form`
2. Validates: name, phone, email, message
3. Checks reCAPTCHA token (graceful fallback if unavailable)
4. Sends email via Resend API
5. Returns success/error response

**Validation Rules** (must match frontend):
- **Name**: Letters + spaces, hyphens, apostrophes; 2–60 chars
- **Phone**: 8–15 digits; optional +; spaces/hyphens allowed
- **Email**: Standard email format
- **Message**: Non-empty

### Email Delivery

**Service**: Resend (https://resend.com)  
**Account**: Associated with `robin.payman@gmail.com`  
**API Key**: Stored in environment variable `RESEND_API_KEY`  

**Sender Email**:
- Primary: `info@entrynor.no` (requires domain verification in Resend)
- Fallback: `onboarding@resend.dev` (always works)

**Recipient Email**:
- Primary: `info@entrynor.no` (set in env)
- Fallback: `robin.payman@gmail.com` (Resend account owner)

**Configuration in Vercel**:
```
Environment Variables → Add:
- RESEND_API_KEY=your-key-here
- RECIPIENT_EMAIL=info@entrynor.no (optional; defaults to info@entrynor.no)
```

#### Resend Setup Checklist
- [ ] Domain (`entrynor.no`) verified in Resend dashboard
- [ ] API key generated and stored in Vercel env vars
- [ ] Test email sent and received

---

## 5. Environment Variables & Secrets

### Local Development

**File**: `.env.local` (gitignored, never commit)  
**Example Template**: `.env.example`

```
# Resend API Configuration
RESEND_API_KEY=your-key-here

# Google reCAPTCHA v3
PUBLIC_RECAPTCHA_SITE_KEY=your-site-key
RECAPTCHA_SECRET_KEY=your-secret-key

# Contact Form API
PUBLIC_CONTACT_API_URL=https://minor-mercury.vercel.app/api/contact-form
```

**Important Secrets**:
- `RESEND_API_KEY` - Controls email sending
- `RECAPTCHA_SECRET_KEY` - Validates reCAPTCHA tokens

### Production (Vercel)

**Where to Set**: Vercel Dashboard → Settings → Environment Variables  
**Variables Needed**:
- `RESEND_API_KEY` - Resend email service API key
- `RECAPTCHA_SECRET_KEY` - Google reCAPTCHA secret
- `RECIPIENT_EMAIL` - (optional) Email to receive form submissions

**Important**: Vercel variables are automatically injected at build/deploy time.

---

## 6. Password & Credential Management

### Azure Key Vault (Preferred)

**Purpose**: Centralized secret management  
**Access**: Azure Portal  
**Current Status**: Preferred method (per rules)

**Secrets to Store**:
- Resend API Key
- reCAPTCHA Secret Key
- GitHub Personal Access Tokens
- SSH private keys (backups)

### Local Credential Backups

**File**: `.credentials-database.md`  
**Location**: `/Users/rp/wc-tripletex-integration/` (referenced in rules)  
**Sync**: Backup to OneDrive within 24 hours of any credential changes  
**Encryption**: Encrypted in cloud backup only  

**Contents**:
- Operational references only (no actual secrets exposed)
- Maps to secrets in Azure Key Vault or Resend
- Example: `Resend API: [stored in Azure Key Vault]`

### Google reCAPTCHA

**Console**: https://www.google.com/recaptcha/admin  
**Site Key**: `6Ler10QnAAAAAHPMsiNc97HAUkoLyOx9Bzm594Db` (public, safe in code)  
**Secret Key**: Stored in Vercel env vars and local `.env.local`  
**Type**: reCAPTCHA v3 (no user interaction)

### Resend

**Dashboard**: https://resend.com  
**Account**: `robin.payman@gmail.com`  
**API Key Location**: Vercel environment variables  
**Domain Setup**: Verify `entrynor.no` in Resend dashboard

---

## 7. DNS & Domain Configuration

**Domain Provider**: (Not yet specified in docs)  
**Domain**: entrynor.no  

### DNS Records Needed
- **A Record** or **CNAME**: Point to Vercel
  - Vercel provides exact records in dashboard
  - Or use Vercel's nameserver delegation

### Vercel Domain Setup
1. Go to Vercel Dashboard → Project → Settings → Domains
2. Add `entrynor.no`
3. Follow Vercel's DNS configuration instructions
4. Verify domain (usually takes 5–30 minutes)

---

## 8. Local Development

### Setup

```bash
# Clone repository
git clone git@github.com:robinpayman/entrynor-website.git
cd entrynor-website/minor-mercury

# Install dependencies
npm install

# Create .env.local with your secrets
cp .env.example .env.local
# Edit .env.local and add your actual API keys
```

### Running Locally

**Development Server** (per AGENTS.md rule):
```bash
astro dev --background
```

Manage with:
- `astro dev stop` - Stop the server
- `astro dev status` - Check status
- `astro dev logs` - View logs

**Preview Build**:
```bash
npm run build
npm run preview
```

### File Structure

```
minor-mercury/
├── src/
│   ├── components/
│   │   └── ContactFormI18n.astro  ← Bilingual contact form
│   ├── pages/
│   │   └── [lang]/                ← Dynamic i18n routes
│   └── utils/
│       └── i18n.ts                ← i18n helper functions
├── api/
│   └── contact-form.js            ← Vercel serverless API
├── messages/
│   ├── no.json                    ← Norwegian translations
│   ├── en.json                    ← English translations
│   └── ...                        ← Other languages
├── .env.local                     ← Local secrets (gitignored)
├── .env.example                   ← Secrets template
├── vercel.json                    ← Vercel build config
└── package.json                   ← Dependencies
```

---

## 9. Deployment Process

### Local to Live

**Step 1**: Make changes locally in `/Users/rp/entrynor-website/minor-mercury`  
**Step 2**: Test with `astro dev` or `npm run build` + `npm run preview`  
**Step 3**: Commit and push to GitHub:
```bash
git add .
git commit -m "Your message

Co-Authored-By: Oz <oz-agent@warp.dev>"
git push origin main
```
**Step 4**: Vercel auto-deploys from `main`  
**Step 5**: Verify live at https://entrynor.no

### Rollback

If something goes wrong:
1. Vercel Dashboard → Deployments
2. Find the previous good deployment
3. Click "Rollback" → Confirm

This reverts the site to the previous state instantly.

---

## 10. Monitoring & Logs

### Vercel Logs

**Access**: Vercel Dashboard → Project → Deployments → [Latest] → Logs  
**View**: Build logs, serverless function logs, deployment status

### Contact Form Logs

**Function Logs**: Vercel Dashboard → Serverless Functions → contact-form  
**What to Look For**:
- Form submission received
- Validation errors
- Email delivery success/failure
- reCAPTCHA verification

### Email Delivery Status

**Resend Dashboard**: https://resend.com  
**View**: Emails tab → See delivery status, bounces, failures

---

## 11. Contact Form: Field Validation

### Frontend Validation

**File**: `src/components/ContactFormI18n.astro` (lines 332–343)

**Messages** (Norwegian/English):
- **Name**: "Skriv inn et gyldig navn (kun bokstaver, 2–60 tegn)." / "Enter a valid name (letters only, 2–60 characters)."
- **Phone**: "Telefonnummeret må inneholde 8–15 sifre..." / "Phone must contain 8–15 digits..."
- **Email**: "Skriv inn en gyldig e-postadresse." / "Enter a valid email address."
- **Message**: "Skriv en melding." / "Enter a message."

**Note**: These messages display **only when user clicks a field with an error**, then are cleared on input.

### Backend Validation

**File**: `api/contact-form.js` (lines 36–57)

**Validation** (must match frontend):
- Name: `/^[\p{L}][\p{L}' .\-]{1,59}$/u` (Unicode-aware)
- Email: `/^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/`
- Phone: 8–15 digits (after stripping separators)
- Message: Non-empty string

---

## 12. Testing the Form

### Test Locally

```bash
# Start dev server
astro dev --background

# Visit http://localhost:3000/no/ or /en/
# Fill form with test data
# Check browser console for network requests
```

### Test Production

1. Go to https://entrynor.no
2. Fill the contact form
3. Submit
4. Check:
   - Success message appears
   - Email arrives at recipient
   - No errors in Vercel logs

### Manual Testing Checklist

- [ ] **Name validation**: Empty → error; invalid chars → error; valid name → OK
- [ ] **Phone validation**: Too short → error; valid phone → OK
- [ ] **Email validation**: Invalid format → error; valid email → OK
- [ ] **Message validation**: Empty → error; filled → OK
- [ ] **All errors shown at once** (not one at a time)
- [ ] **Error messages clear on input**
- [ ] **Success message shows after submit**
- [ ] **Email received** at configured recipient
- [ ] **Bilingual**: Works in both /no/ and /en/

---

## 13. Troubleshooting

### Form Not Submitting

**Check**:
1. Browser console for JS errors
2. Network tab in DevTools → XHR requests
3. Vercel function logs for 500 errors

**Common Issues**:
- `RESEND_API_KEY` missing in Vercel env vars
- reCAPTCHA secret key missing or invalid
- Network blocked by browser CORS policy

### Emails Not Arriving

**Check**:
1. Resend dashboard for delivery status
2. Spam/junk folder (especially Gmail)
3. Verify `entrynor.no` domain in Resend
4. Check recipient email is correct in env vars

### Validation Messages Disappear

**Expected behavior**: Field-level messages show when error occurs, clear when user clicks field or types. Status message shows summary: "Kontroller de markerte feltene og prøv igjen."

**If not working**: Check browser console for JS errors in `ContactFormI18n.astro` script.

---

## 14. Quick Reference

### Important URLs

| Purpose | URL |
| --- | --- |
| Live Site | https://entrynor.no |
| Vercel Alt URL | https://minor-mercury.vercel.app |
| GitHub Repo | https://github.com/robinpayman/entrynor-website |
| Vercel Dashboard | https://vercel.com |
| Resend Dashboard | https://resend.com |
| reCAPTCHA Console | https://www.google.com/recaptcha/admin |
| Azure Key Vault | https://portal.azure.com |

### Environment Variables Summary

| Variable | Storage | Sensitivity | Purpose |
| --- | --- | --- | --- |
| `RESEND_API_KEY` | Vercel Env, Azure KV | **SECRET** | Email delivery |
| `RECAPTCHA_SECRET_KEY` | Vercel Env, `.env.local` | **SECRET** | Bot protection |
| `PUBLIC_RECAPTCHA_SITE_KEY` | Code, Vercel Env | Public | reCAPTCHA frontend |
| `RECIPIENT_EMAIL` | Vercel Env | Sensitive | Contact form recipient |
| `PUBLIC_CONTACT_API_URL` | Code | Public | API endpoint URL |

### Command Cheat Sheet

```bash
# Local Development
astro dev --background       # Start dev server
astro dev stop              # Stop dev server
astro dev status            # Check status
npm run build               # Build for production
npm run preview             # Preview production build

# Git/Deployment
git push origin main        # Deploy to Vercel
git status                  # Check changes
git log --oneline           # View commits
```

---

## 15. Future Enhancements

- [ ] Set up monitoring/alerting for form submission failures
- [ ] Add email templates for auto-reply to user
- [ ] Implement form submission rate limiting
- [ ] Add analytics for form engagement
- [ ] Setup backup email delivery (Sendgrid as fallback to Resend)

---

**Last Updated**: August 2026  
**Document Owner**: Robin Payman  
**Backup Location**: Azure Key Vault + OneDrive
