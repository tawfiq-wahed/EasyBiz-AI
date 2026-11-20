# EasyBiz - AI · SME Finance Copilot

A runnable full-stack SME financial operations MVP for the 2026 Shenzhen International FinTech Competition. React/Vite/Tailwind frontend, Express API, MongoDB Atlas/Mongoose, optional Qwen. **Not a production accounting system or lender.**

## Start

Requires Node.js 22 LTS (recommended) and MongoDB Atlas replica set (or local MongoDB replica set for transactions). MongoDB Atlas free tier supports transactions. A standalone local MongoDB server does NOT support purchase/sale transactions.

```bash
cd server
cp .env.example .env
# edit MONGODB_URI and JWT_SECRET (at least 32 random characters)
npm install
npm run dev
```

In another terminal:

```bash
cd client
npm install
npm run dev
```

Open http://localhost:5173. Backend listens on **5001**, frontend proxy forwards `/api` to 5001. Register → create business → create products → buy stock at actual costs → sell items → view dashboard. Do not use port 5000.

Generate a JWT secret with `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`.

## Financial definitions

- Section A: **Sales** = sum of pre-discount sales totals, **Stock purchases** = full stock purchases made during selected period. These are independent flows; their difference is NOT profit.
- Section B: **Cost of goods sold** (COGS) = FIFO cost of inventory units sold, **Discount** = selected 0–20% of `max(0, pre-discount gross profit)` per sale. **Gross profit** = sales before discount − COGS − discount. **Net profit** = gross profit − recorded operating expenses. Inventory purchases do not directly reduce profit until the stock is sold.
- Daily/weekly/monthly/yearly are calendar periods in the **server timezone** (configure TZ=Asia/Dhaka if needed); credit transactions are included in sales/purchases when recorded, not when cash settles.
- Currency follows business setting but the current frontend formats values as BDT (৳); choose BDT for a consistent demo.
- Multi-item sales use MongoDB transactions and conditional FIFO batch decrements. Financial entries are append-only in the MVP. Discounts and costs are snapshotted in each sale.

## API

Auth: `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me`, `POST /api/auth/logout`.

Business: `GET/POST /api/businesses`.

Workspace (requires signed-in cookie and `x-business-id` header):

- `GET/POST /api/workspace/products`, `PATCH /api/workspace/products/:id`
- `GET/POST /api/workspace/purchases`
- `GET/POST /api/workspace/sales`
- `GET/POST /api/workspace/expenses`
- `GET /api/workspace/dashboard?period=daily|weekly|monthly|yearly`
- `GET /api/workspace/insights`, `GET /api/workspace/financing`
- `POST /api/workspace/copilot` `{ "question": "..." }`
- `GET /api/workspace/audit` (owner/admin)

## Optional Qwen

Set `QWEN_API_KEY`, `QWEN_BASE_URL`, and `QWEN_MODEL` in `server/.env`. Server-side only; never put API keys in Vite client env. The assistant is restricted to verified monthly totals; if Qwen is unavailable it uses a local explanation. It does not execute financial mutations or fabricate financing approvals.

## Security and limitations

Password hashes with bcrypt, HTTP-only SameSite session cookie, Helmet, rate limiting, Zod input validation, business membership isolation, basic audit events. Add CSRF protection for cross-site deployment, stronger role-based write permissions, encrypted backups, monitoring, security review, real payment reconciliation, transaction reversal/refund workflows, full test suite, jurisdiction-specific compliance and ledger accounting before real production use. HTTPS is mandatory in production. Do not deploy with placeholder JWT secret. Financing APRs are clearly labeled **illustrative**, not real-time lender offers. No real bank connection or accurate cash-flow forecasting is implemented.

## Demo flow

1. Register and create a BDT retail business.
2. Add Rice (default selling price ৳120), Oil (৳200).
3. Purchase 10 Rice at ৳80 each and 10 Oil at ৳150 each.
4. Sell 2 Rice at ৳120 and 1 Oil at ৳200, apply 20% of pre-discount gross profit.
5. Subtotal ৳440, FIFO cost ৳310, pre-discount gross profit ৳130, discount ৳26, customer pays ৳414, actual gross profit ৳104.
6. Add rent expense ৳20 → net profit for the period becomes ৳84.
7. Open Dashboard: Section A sales ৳440 and purchases ৳2300; Section B gross profit ৳104 and net profit ৳84.
8. Open Finance for rankings and illustrative financing choices; Copilot for verified monthly explanation.

## Competition deliverables

This archive contains code and documentation, **not** a verified deployed service, recorded demo video, 10-minute presentation, external sandbox operation logs, or completed independent security assessment. Create those separately and validate locally before submission.
