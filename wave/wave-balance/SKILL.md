---
name: wave-balance
description: "Wave Balance & Reconciliation API — consulter le solde, lister les transactions, rembourser. Utilise ce skill pour vérifier le solde Wave, réconciliation comptable, historique des transactions Wave, export de transactions, remboursement de transaction Wave."
---

# Wave Balance & Reconciliation API

L'API Balance permet de consulter le solde de votre wallet business, de lister les transactions et d'effectuer des remboursements. C'est l'API principale pour la réconciliation comptable.

Pour l'authentification et les types de données, consulter [references/authentication.md](../references/authentication.md).

## Endpoints

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| GET | `/v1/balance` | Consulter le solde |
| GET | `/v1/transactions` | Lister les transactions |
| POST | `/v1/transactions/:transaction_id/refund` | Rembourser une transaction |

## Consulter le solde

**GET** `/v1/balance`

### Paramètres optionnels

| Champ | Type | Description |
|-------|------|-------------|
| `include_subaccounts` | boolean | Inclure les soldes des sous-comptes dans le total |

### Exemple — cURL

```bash
curl -H "Authorization: Bearer $WAVE_API_KEY" \
  https://api.wave.com/v1/balance
```

### Exemple — TypeScript/Node.js

```typescript
interface Balance {
  amount: string;
  currency: string;
}

async function getBalance(includeSubaccounts = false): Promise<Balance> {
  const params = includeSubaccounts ? '?include_subaccounts=true' : '';
  const response = await fetch(`https://api.wave.com/v1/balance${params}`, {
    headers: {
      'Authorization': `Bearer ${process.env.WAVE_API_KEY}`,
    },
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Wave API error: ${error.code}`);
  }

  return response.json();
}

const balance = await getBalance();
console.log(`Solde: ${balance.amount} ${balance.currency}`);
```

### Exemple — PHP

```php
function getBalance(bool $includeSubaccounts = false): array {
    $url = 'https://api.wave.com/v1/balance';
    if ($includeSubaccounts) {
        $url .= '?include_subaccounts=true';
    }

    $ch = curl_init($url);
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'Authorization: Bearer ' . $_ENV['WAVE_API_KEY'],
        ],
    ]);

    $response = curl_exec($ch);
    curl_close($ch);

    return json_decode($response, true);
}
```

### Réponse

```json
{
  "amount": "1250000",
  "currency": "XOF"
}
```

## Lister les transactions

**GET** `/v1/transactions`

### Paramètres optionnels

| Champ | Type | Description |
|-------|------|-------------|
| `date` | string | Date spécifique (YYYY-MM-DD), défaut = aujourd'hui |
| `after` | string | Curseur de pagination |
| `include_subaccounts` | boolean | Inclure les transactions des sous-comptes |

### Exemple — cURL

```bash
curl -H "Authorization: Bearer $WAVE_API_KEY" \
  "https://api.wave.com/v1/transactions?date=2024-01-15"
```

### Exemple — TypeScript/Node.js

```typescript
interface Transaction {
  timestamp: string;
  transaction_id: string;
  transaction_type: string;
  amount: string;
  fee: string;
  balance: string;
  currency: string;
  is_reversal: boolean;
  counterparty_name: string | null;
  counterparty_mobile: string | null;
  counterparty_id: string | null;
  business_user_name: string | null;
  business_user_mobile: string | null;
  employee_id: string | null;
  client_reference: string | null;
  payment_reason: string | null;
  checkout_api_session_id: string | null;
  batch_id: string | null;
  aggregated_merchant_id: string | null;
  aggregated_merchant_name: string | null;
  custom_fields: Record<string, string> | null;
  submerchant_id: string | null;
  submerchant_name: string | null;
  government_tax_amount: string | null;
  government_tax_paid_by_wave: string | null;
}

interface TransactionPage {
  page_info: {
    start_cursor: string | null;
    end_cursor: string | null;
    has_next_page: boolean;
  };
  date: string;
  items: Transaction[];
}

async function getTransactions(
  date?: string,
  includeSubaccounts = false
): Promise<Transaction[]> {
  const allTransactions: Transaction[] = [];
  let cursor: string | undefined;

  do {
    const params = new URLSearchParams();
    if (date) params.set('date', date);
    if (cursor) params.set('after', cursor);
    if (includeSubaccounts) params.set('include_subaccounts', 'true');

    const response = await fetch(
      `https://api.wave.com/v1/transactions?${params}`,
      {
        headers: {
          'Authorization': `Bearer ${process.env.WAVE_API_KEY}`,
        },
      }
    );

    if (!response.ok) {
      const error = await response.json();
      throw new Error(`Wave API error: ${error.code}`);
    }

    const page: TransactionPage = await response.json();
    allTransactions.push(...page.items);

    cursor = page.page_info.has_next_page
      ? page.page_info.end_cursor ?? undefined
      : undefined;
  } while (cursor);

  return allTransactions;
}

// Récupérer toutes les transactions d'une journée
const transactions = await getTransactions('2024-01-15');
```

### Réponse

```json
{
  "page_info": {
    "start_cursor": null,
    "end_cursor": "TFRfdUZ1MGoyMzVKemtz",
    "has_next_page": true
  },
  "date": "2024-01-15",
  "items": [
    {
      "timestamp": "2024-01-15T14:41:15Z",
      "transaction_id": "T_V3TFOUE7VU",
      "transaction_type": "merchant_payment",
      "amount": "990",
      "fee": "10",
      "balance": "1250990",
      "currency": "XOF",
      "is_reversal": false,
      "counterparty_name": "Mame Diop",
      "counterparty_mobile": "+221761110000",
      "client_reference": null,
      "custom_fields": null
    }
  ]
}
```

## Types de transactions

| Type | Description |
|------|-------------|
| `merchant_payment` | Paiement client → business |
| `merchant_payment_refund` | Remboursement d'un paiement marchand |
| `api_checkout` | Paiement via Checkout API |
| `api_checkout_refund` | Remboursement Checkout API |
| `api_payout` | Envoi via Payout API |
| `api_payout_reversal` | Reversal Payout API |
| `bulk_payment` | Paiement en masse (portail) |
| `bulk_payment_reversal` | Reversal paiement en masse |
| `b2b_payment` | Paiement business-to-business |
| `b2b_payment_reversal` | Reversal B2B |
| `merchant_sweep` | Transaction de balayage marchand |

## Rembourser une transaction

**POST** `/v1/transactions/:transaction_id/refund`

- Opération idempotente
- Pas de corps de requête nécessaire
- Rembourse le paiement incluant les frais

```bash
curl -X POST \
  https://api.wave.com/v1/transactions/T_V3TFOUE7VU/refund \
  -H "Authorization: Bearer $WAVE_API_KEY"
```

**Réponses :**
- 200 : Succès (pas de corps)
- 404 : Transaction introuvable
- 500 : Erreur serveur

## Réconciliation comptable

Pour réconcilier les transactions, itérer sur les dates avec pagination complète :

```typescript
async function reconcile(startDate: string, endDate: string) {
  const start = new Date(startDate);
  const end = new Date(endDate);
  const allTransactions: Transaction[] = [];

  for (let d = new Date(start); d <= end; d.setDate(d.getDate() + 1)) {
    const dateStr = d.toISOString().split('T')[0];
    const dayTransactions = await getTransactions(dateStr);
    allTransactions.push(...dayTransactions);
  }

  // Calculer les totaux par type
  const summary = allTransactions.reduce((acc, tx) => {
    const type = tx.transaction_type;
    if (!acc[type]) acc[type] = { count: 0, total: 0, fees: 0 };
    acc[type].count++;
    acc[type].total += parseInt(tx.amount);
    acc[type].fees += parseInt(tx.fee);
    return acc;
  }, {} as Record<string, { count: number; total: number; fees: number }>);

  return { transactions: allTransactions, summary };
}
```
