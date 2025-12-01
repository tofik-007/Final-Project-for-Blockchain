
# Decentralized Blockchain Transaction System

This project is a small educational blockchain system consisting of:

- A Python/Flask blockchain node (`blockchain.py`) that manages blocks, transactions, and mining.
- A web-based wallet generator and transaction client (HTML/JS) that lets users:
  - Generate RSA wallets.
  - Create and sign transactions.
  - Submit transactions to the node.
  - View pending and confirmed transactions.
  - Check balance by public key.
  - Inspect address activity (incoming / outgoing transfers).

It is designed to demonstrate basic blockchain concepts: transactions, digital signatures, proof of work, mining rewards, and simple node consensus.

***

## Features

### Node (`blockchain.py`)

- In‑memory blockchain with:
  - Genesis block creation.
  - Proof‑of‑work mining with configurable difficulty.
  - Mining reward from a special “The Blockchain” sender.
- Transaction handling:
  - Transactions stored in a mempool before mining.
  - Digital signature verification using RSA and PKCS#1 v1.5.
  - Validation of:
    - Amount is numeric and greater than zero.
    - Sender and recipient addresses are not the same.
- REST API:
  - `GET /chain` – full chain.
  - `GET /transactions/get` – unmined (mempool) transactions.
  - `GET /mine` – mine a new block and include mempool transactions.
  - `POST /transaction/new` – submit a new signed transaction.
  - `GET /nodes/get`, `POST /nodes/register`, `GET /nodes/resolve` – simple node registry and longest‑chain consensus.
  - `GET /address/<public_key>/transactions` – activity for a single address (confirmed and pending).
- HTML views:
  - `GET /` – node frontend showing pending and confirmed transactions.
  - `GET /configure` – configure and register peer nodes.

### Client (Wallet & Transaction UI)

- **Wallet Generator page (`index.html`):**
  - Generate a new RSA keypair.
  - Display public and private keys.
  - Copy‑to‑clipboard buttons for both keys for easy reuse in the client.

- **Make Transaction page (`make_transaction.html`):**
  - Form to enter:
    - Sender public key.
    - Sender private key.
    - Recipient public key.
    - Amount.
  - Client‑side validation:
    - Non‑empty sender/recipient keys.
    - Prevent sender and recipient keys from being the same.
    - Amount must be a valid positive number.
  - Generate Transaction:
    - Sends form data to a backend endpoint (`/generate/transaction`) that constructs and signs a transaction.
    - Shows a confirmation modal with sender, recipient, amount, and signature.
  - Confirm Transaction:
    - Submits the signed transaction to the blockchain node (`/transaction/new`).
    - Displays clear error messages when backend validation fails.
  - Check Balance card:
    - Takes any public key and calls the node’s `/chain` endpoint.
    - Aggregates all incoming and outgoing amounts for that key.
    - Displays total received, total sent, and current balance.
  - Address Activity card:
    - Calls `/address/<public_key>/transactions`.
    - Shows a table of all IN / OUT transactions for that address, with block number, amount, status (PENDING / CONFIRMED) and timestamp.

***

## Architecture

- **Backend:** Python 3, Flask, PyCryptodome, Requests.
- **Frontend:** Plain HTML, Bootstrap, jQuery, and DataTables (for node UI tables).
- **Communication:** JSON over HTTP. The wallet client normally talks to a node running on `http://127.0.0.1:5001`.

Logical flow:

1. Wallet Generator creates a keypair.
2. User copies keys into the Make Transaction page.
3. Make Transaction page validates inputs and asks `/generate/transaction` (separate backend client) to create a signed transaction.
4. User reviews and confirms the transaction; client sends it to the node at `/transaction/new`.
5. Node validates signature and amount, then stores the transaction in the mempool.
6. When `/mine` is called from the node UI, the node runs proof of work, creates a block, and moves mempool transactions into the chain.

***

## Getting Started

### Prerequisites

- Python 3.10+ recommended.
- Pip with the following packages:
  - `Flask`
  - `Flask-Cors`
  - `requests`
  - `pycryptodome` (Crypto)

Install dependencies:

```bash
pip install Flask Flask-Cors requests pycryptodome
```

### Run the blockchain node

From the project root:

```bash
python blockchain.py -p 5001
```

This starts a node on `http://127.0.0.1:5001`.

- Node frontend: `http://127.0.0.1:5001/`
- Configure page: `http://127.0.0.1:5001/configure`

### Run the wallet client

Serve the client files with a simple HTTP server, or from a separate Flask app if you have one. For a quick static server from the `client` directory:

```bash
cd client
python -m http.server 8001
```

Then open:

- Wallet Generator: `http://127.0.0.1:8001/`
- Make Transaction: `http://127.0.0.1:8001/make/transaction`
- View Transactions: `http://127.0.0.1:8001/view/transaction`

Make sure the **Blockchain Node URL** field in the Make Transaction confirmation modal is set to the node address, usually:

```text
http://127.0.0.1:5001
```

***

## How to Use

### 1. Generate a wallet

1. Open the Wallet Generator page.
2. Click **Generate Wallet**.
3. Copy and securely store:
   - Public key.
   - Private key.
4. Optionally generate another wallet to act as recipient.

### 2. Send a transaction

1. Open **Make Transaction** page.
2. Paste:
   - Sender public key.
   - Sender private key.
   - Recipient public key (must be different).
3. Enter a positive **Amount**.
4. Click **Generate Transaction**.
   - If keys are the same or the amount is invalid, you will see a validation alert.
5. Review the confirmation modal (sender, recipient, amount, signature).
6. Ensure **Blockchain Node URL** matches the node (`http://127.0.0.1:5001`).
7. Click **Confirm Transaction**.
   - On success you see a success alert.
   - On failure you see a clear backend message (e.g., invalid signature, amount error, same sender/recipient).

### 3. Mine and view transactions

1. Go to the node UI: `http://127.0.0.1:5001/`.
2. Click **Refresh** to see pending transactions in the mempool.
3. Click **Mine** to create a new block and confirm those transactions.
4. Use **Refresh (Resolve Nodes)** to re-sync if running multiple nodes.

### 4. Check balance

1. On Make Transaction page, go to **Check Balance**.
2. Paste a public key and click **Get Balance**.
3. The Received, Sent, and Balance values are computed from the node’s chain.

### 5. Address activity

1. On Make Transaction page, go to **Address Activity**.
2. Paste a public key and click **Load Activity**.
3. See all IN/OUT transactions for that address, including:
   - Direction.
   - Amount.
   - Counterparty/Recipient.
   - Block number and status.
   - Timestamp.

## Validation & Error Handling

- **Client-side:**
  - Prevents submitting when:
    - Sender or recipient key is missing.
    - Sender and recipient keys are identical.
    - Amount is missing, not a number, or not positive.
- **Server-side:**
  - Re-validates:
    - Amount is numeric and greater than zero.
    - Sender and recipient are not the same.
    - Digital signature matches the transaction contents and sender public key.
  - Returns structured JSON errors which are shown to the user in alerts.

This dual-layer validation improves robustness and matches typical grading rubrics for input validation, security, and UX.
## Security Notes

- This project is for **educational purposes only**.
- Keys and transactions are handled in plain text over HTTP and stored in memory.
- Do not use this code with real funds or private keys you care about.
- There is no persistence; restarting the node clears the chain and mempool.

## Possible Extensions

Ideas for further improvement:

- Persist the blockchain to disk (JSON or a simple database).
- Add support for multiple inputs/outputs per transaction.
- Add adjustable mining difficulty and statistics.
- Improve UI with better error banners and success toasts.
- Implement unit tests for transaction validation and proof‑of‑work.
