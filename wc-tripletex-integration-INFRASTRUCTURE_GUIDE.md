# WooCommerce-Tripletex Integration Infrastructure Guide

Complete reference for the WooCommerce ↔ Tripletex synchronization system.

---

## 1. Project Overview

**Project Name**: WooCommerce-Tripletex Integration  
**Repository**: https://github.com/robinpayman/wc-tripletex-integration  
**Production URL**: Integrated into WooCommerce at ata-domain.tempurl.host  
**Type**: WordPress plugin + Vercel API backend  
**Purpose**: Bi-directional synchronization of orders, products, and stock between WooCommerce and Tripletex  

### Key Features
- **Order Sync**: WooCommerce → Tripletex (automatic on order status change)
- **Product Sync**: Tripletex → WooCommerce (manual/scheduled)
- **Stock Sync**: Tripletex → WooCommerce (manual/scheduled)
- **Customer Mapping**: B2C/B2B customer linking in Tripletex
- **Webhook Integration**: Real-time order status updates
- **Multi-sync coordination**: 3 independent sync processes

---

## 2. Code Repository

### GitHub

**Repository**: `robinpayman/wc-tripletex-integration`  
**URL**: https://github.com/robinpayman/wc-tripletex-integration  
**Main Branch**: `main`  
**Access**: SSH via GitHub  

### Local Development

**Directory**: `/Users/rp/wc-tripletex-integration`  
**Type**: WordPress plugin (PHP)  
**Key Files**:
- `.credentials-database.md` - Operational reference guide (gitignored)
- `.env` / `.env.local` - Environment variables (gitignored)
- `PRODUCTION_GO_LIVE_GUIDE.md` - Deployment checklist
- `PHASE_1_WEBHOOK_SETUP.md` - Webhook configuration
- `PRODUCT_SYNC_STRATEGY.md` - Product mapping rules

---

## 3. Production Hosting

### WordPress Hosting

**Server**: ata-domain.tempurl.host (Shared hosting)  
**WordPress Path**: `/home/atawarp/site/public_html/`  
**Plugin Path**: `/home/atawarp/site/public_html/wp-content/plugins/wc-tripletex-integration/`  
**Database**: MySQL on same server  
**Access**: SSH + SFTP  

### Vercel API Backend

**Project**: wc-tripletex-integration (Vercel)  
**Function URLs**:
- `/api/wc-orders` - Order webhook receiver
- `/api/wc-products` - Product sync endpoint
- `/api/wc-stock` - Stock sync endpoint

**API Keys**: Stored in Vercel environment variables

---

## 4. Synchronization Architecture

### Order Sync (Automatic)

**Trigger**: WooCommerce order status changes to "Processing"  
**Flow**:
1. WooCommerce webhook fires when order → "Processing"
2. Webhook posts to Vercel API
3. API validates order data against Tripletex
4. Creates invoice/order in Tripletex
5. Sets order status to "Ready for Picking" in Tripletex
6. Logs result in WooCommerce notes

**Validation**:
- Order amount must match 100% between systems
- Customer matched or created in Tripletex
- All products mapped and in stock
- Contact assigned (defaults to 'ata Webshop')

### Product Sync (Manual)

**Command**: Run manually when products change in Tripletex  
**Flow**:
1. Fetch products from Tripletex API
2. Match by product ID (exact match required)
3. Update WooCommerce product data:
   - Name, price, description
   - Stock levels (availableStock field)
   - SKU/product code
4. Create products if missing (rare)

**Handling**:
- Bundles: Mapped to static product '9993 Bundle'
- Composite products: Same as bundles
- Products ending in '-x'/'-X': Ignored

### Stock Sync (Manual)

**Command**: Run manually when stock updates in Tripletex  
**Source**: Tripletex `availableStock` field  
**Updates**: WooCommerce product stock levels  
**Frequency**: On-demand or scheduled

---

## 5. Deployment Process

### Local to Production

**Step 1**: Make changes locally in `/Users/rp/wc-tripletex-integration`  
**Step 2**: Test with:
```bash
# Run integration tests
php phpunit tests/

# Verify API endpoints
curl -X POST https://wc-tripletex-integration.vercel.app/api/wc-orders
```

**Step 3**: Commit and push to GitHub:
```bash
git add .
git commit -m "Your message

Co-Authored-By: Oz <oz-agent@warp.dev>"
git push origin main
```

**Step 4**: Deploy to WordPress:
```bash
# SSH into server
ssh -i ~/.ssh/id_ed25519 atawarp@ata-domain.tempurl.host

# Backup database
wp db export /home/atawarp/backups/$(date +%Y%m%d)/backup.sql

# Deploy plugin
cd /home/atawarp/site/public_html/wp-content/plugins/wc-tripletex-integration
git pull origin main

# Activate plugin (if new)
wp plugin activate wc-tripletex-integration

# Clear any caches
wp cache flush
```

**Step 5**: Verify deployment:
```bash
# Check plugin is active
wp plugin is-active wc-tripletex-integration

# Test webhook endpoint
curl -X POST https://wc-tripletex-integration.vercel.app/api/wc-orders \
  -H "Content-Type: application/json" \
  -d '{"test": true}'

# Check error logs
tail -f /home/atawarp/site/public_html/wp-content/debug.log
```

---

## 6. Environment Variables & Secrets

### Local Development (`.env.local`)

```bash
# Tripletex API (PRODUCTION)
TRIPLETEX_CONSUMER_TOKEN=your-token
TRIPLETEX_EMPLOYEE_TOKEN=your-employee-token
TRIPLETEX_API_URL=https://api.tripletex.io

# WooCommerce API
WOOCOMMERCE_API_URL=https://ata-domain.tempurl.host/wp-json
WOOCOMMERCE_API_USER=your-api-user
WOOCOMMERCE_API_PASSWORD=your-api-password

# Vercel/Deployment
VERCEL_TOKEN=your-vercel-token
```

### Production (WordPress Server)

**File**: `/home/atawarp/site/public_html/.env`  
**Access**: SSH only  
**Contains**: Same as above + additional secrets

### Vercel Environment

**Dashboard**: https://vercel.com → wc-tripletex-integration  
**Variables**:
- `TRIPLETEX_CONSUMER_TOKEN`
- `TRIPLETEX_EMPLOYEE_TOKEN`
- `TRIPLETEX_API_URL`
- `WOOCOMMERCE_API_URL`
- `WOOCOMMERCE_API_USER`
- `WOOCOMMERCE_API_PASSWORD`

---

## 7. Password & Credential Management

### Azure Key Vault (Preferred)

**Secrets to Store**:
- Tripletex API tokens (PRODUCTION and TEST)
- WooCommerce API credentials
- SFTP credentials for backups
- GitHub personal access tokens

### Local Backup

**File**: `.credentials-database.md` (gitignored)  
**Location**: `/Users/rp/wc-tripletex-integration/.credentials-database.md`  
**Sync**: Backup to OneDrive within 24 hours of changes  
**Format**: Operational references only (no actual secrets)

### Tripletex API Tokens

**Types**:
- **Consumer Token**: Identifies the application
- **Employee Token**: Authenticates as user, expires periodically

**Important**: When new Tripletex tokens are created, delete all existing tokens globally (rule #6)

### SFTP & SSH Access

**Key**: `~/.ssh/id_ed25519`  
**Permissions**: `600` (owner read/write only)  
**Backup**: Azure Key Vault + OneDrive  
**Host**: ata-domain.tempurl.host  
**User**: `atawarp`

---

## 8. Database Structure

### WordPress Database

**Location**: MySQL on ata-domain.tempurl.host  
**Backup Path**: `/home/atawarp/backups/YYYYMMDD/backup.sql`  

**Custom Tables**:
- `wp_wc_tt_customer_mapping` - WooCommerce → Tripletex customer links
- `wp_wc_tt_order_log` - Sync history and results
- `wp_wc_tt_product_mapping` - Product ID mappings

### Local Database Cache

**File**: `.wc_tt_customer_mapping.db` (SQLite)  
**Purpose**: Local customer mapping reference  
**Sync**: Updated when running customer matching

---

## 9. Webhook Configuration

### Setup

**File**: `PHASE_1_WEBHOOK_SETUP.md`  
**Endpoint**: https://wc-tripletex-integration.vercel.app/api/wc-orders  
**Event**: Order status change to "Processing"  
**Auth**: HMAC-SHA256 signature (optional but recommended)

### Testing Webhook

```bash
# Trigger from WooCommerce admin:
# WooCommerce → Settings → API → Webhooks → Manage

# Or test manually:
curl -X POST https://wc-tripletex-integration.vercel.app/api/wc-orders \
  -H "X-WC-Webhook-Signature: hmac-sha256=..." \
  -H "Content-Type: application/json" \
  -d '{
    "id": 12345,
    "status": "processing",
    "total": "1500.00",
    "customer_id": 1
  }'
```

---

## 10. API Field Mapping Reference

### Order Mapping

**WooCommerce → Tripletex**:
- Order ID → Invoice/Order number in TT
- Order total → Invoice amount (must match 100%)
- Order date → Invoice date
- Customer info → Customer/Contact in TT
- Order items → Invoice lines
- Shipping cost → Static product '99995 Frakt ata' or '9999 Frakt'

### Product Mapping

**Tripletex → WooCommerce**:
- TT Product ID → WC Product ID (exact match)
- TT Product Name → WC Product Title
- TT Price → WC Product Price
- TT Available Stock → WC Stock Level
- TT Description → WC Description
- TT SKU → WC SKU

**Special Cases**:
- Bundles → '9993 Bundle' (price from WC, components added with quantity)
- Composite products → Same as bundles
- Products ending in '-x'/'-X' → Ignored
- Stock mapping uses `availableStock` (not `Stock of goods`)

### Customer Mapping

**Rules** (Rule #5):
- **B2C**: No contact needed
- **B2B**: 
  - Match by customer ID first
  - Try to match existing contact
  - Create new contact if no match
  - Link to customer if possible
  - Default to 'ata Webshop' contact (ID: 10075654) if no match

---

## 11. Testing the Integration

### Pre-Deployment Testing

```bash
# 1. Test Tripletex API connection
curl -H "Authorization: Basic $(echo -n 'consumer:token' | base64)" \
  https://api.tripletex.io/v2/accounting/order

# 2. Test WooCommerce API
curl -u "$WC_USER:$WC_PASSWORD" \
  https://ata-domain.tempurl.host/wp-json/wc/v3/orders

# 3. Run integration tests
php phpunit tests/integration/

# 4. Verify webhook signature
php -r 'echo hash_hmac("sha256", "test", "secret");'
```

### Production Testing Checklist

- [ ] Create test order in WooCommerce with status "Processing"
- [ ] Verify order appears in Tripletex API
- [ ] Verify order amount matches 100%
- [ ] Verify customer is created/linked in Tripletex
- [ ] Check WooCommerce order notes for sync result
- [ ] Verify stock levels sync from Tripletex
- [ ] Test product sync retrieves latest prices
- [ ] Verify webhook logs in debug.log

---

## 12. Monitoring & Logs

### WordPress Debug Log

**Location**: `/home/atawarp/site/public_html/wp-content/debug.log`  
**Monitor**: Tail in real-time during sync
```bash
ssh atawarp@ata-domain.tempurl.host tail -f /home/atawarp/site/public_html/wp-content/debug.log
```

**What to Look For**:
- Order sync success/failure
- API authentication errors
- Product mapping mismatches
- Stock sync results
- Webhook signature verification

### WooCommerce Admin

**Location**: WooCommerce → Orders  
**What to Monitor**:
- Order notes showing sync status
- Order status progression
- Error messages in notes

### Vercel Logs

**Access**: https://vercel.com → wc-tripletex-integration → Logs  
**Monitor**:
- API endpoint requests/responses
- Failed validations
- Webhook processing
- Error stack traces

### Database Sync Logs

**Table**: `wp_wc_tt_order_log`  
**Fields**:
- `order_id` - WooCommerce order ID
- `tripletex_id` - Created order ID in TT
- `status` - success/failure
- `message` - Error or status message
- `synced_at` - Timestamp

---

## 13. Troubleshooting

### Orders Not Syncing

**Check**:
1. Order status is "Processing" (not other statuses)
2. Webhook is active in WooCommerce settings
3. Vercel API is responding: `curl https://wc-tripletex-integration.vercel.app/api/wc-orders`
4. Environment variables are set in Vercel
5. Check debug.log for errors

**Common Issues**:
- **Amount mismatch**: Calculate WC total vs TT invoice amount (including shipping)
- **Customer not found**: Create customer in TT manually or check mapping rules
- **Product not in stock**: Stock check in TT before sync
- **API token expired**: Renew Tripletex employee token

### Products Not Syncing

**Check**:
1. Product IDs match exactly between systems
2. Product status is "Published" in WooCommerce
3. Run product sync command: `wp wc-tripletex product-sync`
4. Check for mapping errors in debug.log

**Common Issues**:
- **ID mismatch**: Product codes must match 100% (Rule #hSPuCsCETttJPPhsSL1gFD)
- **Bundle handling**: Check '9993 Bundle' is active in WC
- **Composite products**: Ensure all components have valid IDs in TT

### Stock Levels Wrong

**Check**:
1. TT `availableStock` is correct (not "Stock of goods")
2. Stock sync has been run recently
3. No product mapping issues

**Sync Stock Manually**:
```bash
ssh atawarp@ata-domain.tempurl.host
wp wc-tripletex stock-sync
```

### API Authentication Failures

**Check**:
1. Tripletex tokens are current (not expired)
2. Token permissions allow order creation
3. Employee token is not in expired state

**Renew Tokens**:
1. Go to Tripletex dashboard
2. Generate new tokens
3. Update .env on all systems
4. Delete old tokens globally (Rule #6)
5. Restart any services using tokens

---

## 14. Quick Reference

### Important URLs

| Service | URL |
| --- | --- |
| GitHub Repo | https://github.com/robinpayman/wc-tripletex-integration |
| Production WC | https://ata-domain.tempurl.host |
| Vercel API | https://wc-tripletex-integration.vercel.app |
| Vercel Dashboard | https://vercel.com |
| Tripletex API | https://api.tripletex.io |
| WordPress Admin | https://ata-domain.tempurl.host/wp-admin |

### SSH/SFTP Access

```bash
# SSH into WordPress server
ssh -i ~/.ssh/id_ed25519 atawarp@ata-domain.tempurl.host

# SFTP access
sftp -i ~/.ssh/id_ed25519 atawarp@ata-domain.tempurl.host

# Deploy plugin
git -C /home/atawarp/site/public_html/wp-content/plugins/wc-tripletex-integration pull origin main
```

### Command Cheat Sheet

```bash
# WooCommerce CLI
wp plugin list                           # List all plugins
wp plugin activate wc-tripletex-integration   # Activate plugin
wp db export backup.sql                 # Backup database
wp cache flush                          # Clear cache

# Sync commands
wp wc-tripletex order-sync              # Sync specific orders
wp wc-tripletex product-sync            # Sync all products from TT
wp wc-tripletex stock-sync              # Update stock from TT

# Testing
curl https://wc-tripletex-integration.vercel.app/api/wc-orders  # Test API
tail -f debug.log                       # Watch logs
```

### Environment Variables Summary

| Variable | Sensitivity | Purpose |
| --- | --- | --- |
| `TRIPLETEX_CONSUMER_TOKEN` | **SECRET** | Identifies app to TT |
| `TRIPLETEX_EMPLOYEE_TOKEN` | **SECRET** | Authenticates as user in TT |
| `WOOCOMMERCE_API_USER` | **SECRET** | WC REST API user |
| `WOOCOMMERCE_API_PASSWORD` | **SECRET** | WC REST API password |
| `WOOCOMMERCE_API_URL` | Sensitive | WC API endpoint |
| `TRIPLETEX_API_URL` | Public | TT API endpoint |

---

## 15. Go-Live Checklist

Before going live with order syncing, verify:

- [ ] All Tripletex API tokens are current
- [ ] WooCommerce API credentials are valid
- [ ] Webhook endpoint is active and responding
- [ ] Test order syncs successfully
- [ ] Customer mapping works for B2C and B2B
- [ ] Product sync retrieves correct data
- [ ] Stock levels update correctly
- [ ] Email notifications work
- [ ] Debug logging is enabled
- [ ] Backups are running daily
- [ ] Team trained on monitoring and troubleshooting
- [ ] 24/7 support plan is in place

---

## 16. Future Enhancements

- [ ] Automatic daily product/stock sync
- [ ] Real-time stock notifications
- [ ] Advanced order matching for partial syncs
- [ ] Inventory reserve system (prevent overselling)
- [ ] Custom email notifications for sync failures
- [ ] Dashboard for sync statistics and health

---

## 17. Klubben Order Invoicing Safety Net (Added August 2026)

### Background

Klubben orders are created directly in Tripletex by the separate `tripletex-pdf-integration` project (XML/SFTP based) - no WooCommerce order exists for them, so `fetch_wc_order_by_tt_id()` can never match them.

Previously, Klubben orders reaching Tripletex status `READY_FOR_INVOICING` were auto-invoiced natively by the company's WMS system, not by any script in this repository. As of August 2026, this repository has taken over that responsibility, adding a shipping-cost safety check the WMS did not have.

### How it works

`sync_tt_invoice_to_wc.py`'s `process_order()` branches on `customer.id`: if it matches `KLUBBEN_CUSTOMER_ID` (17831862), it skips all WooCommerce-order matching/validation/status-update steps and instead:

1. Calculates the shipping-cost ratio via `calculate_klubben_shipping_ratio()`: shipping-product line total ÷ non-shipping line total. Shipping products: 83153313 (99995 Frakt ata), 84868197 (9997 Hentes på lager), 24236172 (9999 Frakt).
2. If the ratio exceeds 30% (`KLUBBEN_SHIPPING_RATIO_THRESHOLD`), the order is **not** invoiced. `hold_klubben_order_for_shipping_review()` sets the order's Tripletex status to `PICKED` ("pakket") and emails an alert to robin@atatreningsutstyr.no. This is naturally idempotent - once held, the order leaves the `READY_FOR_INVOICING` scan filter and won't be re-flagged.
3. If the ratio is within threshold, the order is invoiced directly via the existing `create_invoice_from_order()` (same Tripletex `PUT /order/{id}/:invoice` call used for WooCommerce orders).

A dedicated `--scan-klubben` CLI flag scans only Klubben orders in `READY_FOR_INVOICING`. `wc_order_watcher.py`'s continuous poll loop (`run_once()`, every `POLL_INTERVAL` = 300s) calls `sync_klubben_invoices_from_tripletex()`, which shells out to `sync_tt_invoice_to_wc.py --scan-klubben`.

### Feature flag

The Klubben scan is gated behind `KLUBBEN_INVOICE_SCAN_ENABLED`, stored as an Azure Key Vault secret (`klubben-invoice-scan-enabled` in `tt-secrets-vault`) - not in `.env.production` (removed after the Key Vault secret was verified working, per this project's Key Vault-first credential policy).

### Rollout status

- **Live as of August 27, 2026.** Deployed to the `tripletex-sftp` Azure VM (`/opt/tripletex/app/`), running under the `wc_order_watcher.service` systemd service.
- The WMS system's native Klubben auto-invoice feature is being disabled now that this safety net is confirmed running.
- PR: https://github.com/robinpayman/wc-tripletex-integration/pull/9

### Verification performed before go-live

- Both files compile cleanly.
- `--scan-klubben` dry-run against real production data found 12 Klubben orders; all had already been invoiced by the (still-active at the time) WMS before full details could be fetched, confirming the WMS's speed and that the status check behaves safely.
- 6 fully-mocked offline tests confirmed the shipping-ratio math and both branches (safe→invoice, unsafe→hold+alert).
- Live deployment verified via a clean `wc_order_watcher.service` restart and a full poll cycle completing the new "Checking Klubben orders ready for invoicing" step with no errors, both before and after moving the feature flag from `.env.production` to Key Vault.

---

**Last Updated**: August 2026  
**Maintainer**: Robin Payman  
**Status**: Production  
**Backup Location**: Azure Key Vault + OneDrive
