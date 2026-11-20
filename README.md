# Admin backend

Node.js + Express + SQLite: admin dashboard (served at `/`), user API (`/v1`), M-Pesa callbacks (`/mpesa`).

## Run
```bash
npm install
JWT_SECRET=long-random ADMIN_EMAIL=you@example.com ADMIN_PASSWORD=your-password npm start
```
Open http://localhost:3000. See `.env.example` for every variable. On Railway, attach a volume and set `DB_PATH=/data/data.db`.

## Roles (admin panel)
| Role | Can do |
|---|---|
| superadmin | Everything, including admins, platform wallets, platform control, audit log |
| admin | Read and edit all data, approve transactions |
| support | Read-only |

Every successful admin change is written to the audit log (passwords are never logged).

## M-Pesa setup
1. Create a Daraja app, copy the consumer key/secret, and fill in the `MPESA_*` variables. Set `PUBLIC_URL` to your deployed https URL and `MPESA_CALLBACK_SECRET` to a random string.
2. Callback URLs are built for you: `{PUBLIC_URL}/mpesa/stk/{secret}`, `/mpesa/b2c/result/{secret}`, `/mpesa/b2c/timeout/{secret}`. Register the B2C ones in the Daraja portal.
3. Test in the sandbox first (`MPESA_ENV=sandbox`). B2C on production needs Safaricom approval and an initiator.

Flow: **Deposit** = STK push, balance credited when the callback confirms the payment. **Withdrawal** = funds are held immediately, then paid via B2C when an admin approves (or at once if *auto approve withdrawals* is on). Failed or rejected withdrawals are refunded. If a B2C call times out the transaction stays `processing`: check the Daraja portal before refunding.

## User API (`/v1`, Bearer token from login)
| Route | Body | Purpose |
|---|---|---|
| `POST /auth/register` | `name, phone, password, email?` | Create account, returns token |
| `POST /auth/login` | `phone, password` | Returns token |
| `GET /me` | | Profile and balance |
| `GET /transactions` | | Your last 100 transactions |
| `POST /deposits` | `amount` | Sends an STK push |
| `POST /withdrawals` | `amount` | Requests a payout |
| `GET /products`, `POST /orders` | `product_id, qty, coupon?` | Buy with wallet balance |

## Admin API (`/api`)
Unchanged from before, plus `GET/PUT /settings`, `GET /audit-log`, `POST /users/:id/reset-password`, and `role` on `/admins`.

## Product images
In the admin panel, open **Products** and click **Image** on a row (JPEG, PNG or WebP, up to 5 MB). Files are saved in `UPLOAD_DIR` (default: an `uploads` folder next to `DB_PATH`, so on Render they live on the persistent disk). The product's `image` field holds a path like `/uploads/3-a1b2c3.jpg`; `GET /v1/products` returns it, so apps show `https://your-domain` + that path.
API: `POST /api/products/:id/image` with the raw file as the body and `Content-Type: image/jpeg|png|webp`.

## Customer storefront
Served at `/app/` (for example `https://your-domain/app/`). Customers register with their phone number, top up their wallet with M-Pesa, and buy packages. Packages are the **Products** you create in the admin panel (name, description, price, stock, image). Backend additions: `GET /v1/config`, `GET /v1/orders`. To host the page elsewhere, set `window.API_BASE = "https://your-backend"` before the main script.
# frontedconnect
