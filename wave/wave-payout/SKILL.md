---
name: wave-payout
description: "Wave Payout API — envoyer de l'argent (unitaire/batch), reverser des paiements, vérifier les destinataires. Utilise ce skill pour tout envoi d'argent via Wave, paiement de salaires, transfert vers clients, disbursement, virement Wave, vérification de numéro Wave."
---

# Wave Payout API

L'API Payout permet d'envoyer de l'argent depuis votre wallet business vers des comptes Wave. Supporte les envois unitaires, par lot (batch), les reversals et la vérification des destinataires.

Pour l'authentification et les types de données, consulter [references/authentication.md](../references/authentication.md).

## Endpoints

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| POST | `/v1/payout` | Envoyer un paiement unitaire |
| GET | `/v1/payout/:id` | Récupérer un payout |
| GET | `/v1/payouts/search?client_reference=X` | Rechercher par référence |
| POST | `/v1/payout-batch` | Envoyer un lot de paiements |
| GET | `/v1/payout-batch/:id` | Statut du batch |
| POST | `/v1/payout/:id/reverse` | Reverser un payout (limite 3 jours) |
| POST | `/v1/verify_recipient/` | Vérifier un destinataire |

## Idempotence (obligatoire)

Toutes les requêtes POST qui modifient des données doivent inclure un header `Idempotency-Key` pour éviter les doubles envois d'argent en cas de retry :

```
Idempotency-Key: uuid-v4-unique-par-operation
```

Utiliser un UUID v4 ou un identifiant unique de votre système. La même clé avec le même corps retourne le même résultat sans re-exécuter l'opération.

Si la même clé est utilisée avec un corps différent → erreur `idempotency-mismatch` (409).

## Envoyer un paiement unitaire

**POST** `/v1/payout`

### Paramètres

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `currency` | string | Oui | Code ISO 4217 (ex: `"XOF"`) |
| `mobile` | string | Oui | Numéro E.164 du destinataire |
| `receive_amount` | string | Oui | Montant entier, sans décimales |
| `name` | string | Non | Nom du destinataire (max 255) |
| `national_id` | string | Non | Pièce d'identité (max 255) |
| `payment_reason` | string | Non | Motif du paiement (max 40) |
| `client_reference` | string | Non | Référence de corrélation (max 255) |
| `aggregated_merchant_id` | string | Si agrégateur | ID du sous-marchand |

### Exemple — cURL

```bash
curl -X POST https://api.wave.com/v1/payout \
  -H "Authorization: Bearer $WAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "currency": "XOF",
    "mobile": "+221761110000",
    "receive_amount": "5000",
    "name": "Fatou Ndiaye",
    "payment_reason": "Salaire janvier",
    "client_reference": "SAL-2024-001"
  }'
```

### Exemple — TypeScript/Node.js

```typescript
import { randomUUID } from 'crypto';

interface Payout {
  id: string;
  currency: string;
  receive_amount: string;
  fee: string;
  mobile: string;
  name: string | null;
  status: 'processing' | 'succeeded' | 'failed' | 'reversed';
  timestamp: string;
  client_reference: string | null;
  payout_error: { error_code: string; error_message?: string } | null;
}

async function sendPayout(params: {
  currency: string;
  mobile: string;
  receiveAmount: string;
  name?: string;
  paymentReason?: string;
  clientReference?: string;
}): Promise<Payout> {
  const response = await fetch('https://api.wave.com/v1/payout', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.WAVE_API_KEY}`,
      'Content-Type': 'application/json',
      'Idempotency-Key': randomUUID(),
    },
    body: JSON.stringify({
      currency: params.currency,
      mobile: params.mobile,
      receive_amount: params.receiveAmount,
      name: params.name,
      payment_reason: params.paymentReason,
      client_reference: params.clientReference,
    }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Wave Payout error: ${error.code} - ${error.message}`);
  }

  return response.json();
}

// Utilisation
const payout = await sendPayout({
  currency: 'XOF',
  mobile: '+221761110000',
  receiveAmount: '5000',
  name: 'Fatou Ndiaye',
  paymentReason: 'Salaire janvier',
  clientReference: 'SAL-2024-001',
});
console.log(`Payout ${payout.id}: ${payout.status}`);
```

### Exemple — PHP

```php
function sendPayout(
    string $currency,
    string $mobile,
    string $amount,
    ?string $name = null,
    ?string $reason = null,
    ?string $reference = null
): array {
    $body = json_encode(array_filter([
        'currency' => $currency,
        'mobile' => $mobile,
        'receive_amount' => $amount,
        'name' => $name,
        'payment_reason' => $reason,
        'client_reference' => $reference,
    ]));

    $ch = curl_init('https://api.wave.com/v1/payout');
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => $body,
        CURLOPT_HTTPHEADER => [
            'Authorization: Bearer ' . $_ENV['WAVE_API_KEY'],
            'Content-Type: application/json',
            'Idempotency-Key: ' . bin2hex(random_bytes(16)),
        ],
    ]);

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($httpCode !== 200) {
        throw new Exception("Wave Payout error: $response");
    }

    return json_decode($response, true);
}
```

### Réponse

```json
{
  "id": "pt-185b5e4b8100c",
  "currency": "XOF",
  "receive_amount": "5000",
  "fee": "50",
  "mobile": "+221761110000",
  "name": "Fatou Ndiaye",
  "status": "succeeded",
  "timestamp": "2024-01-15T10:00:00Z",
  "client_reference": "SAL-2024-001",
  "payout_error": null
}
```

## Paiements par lot (batch)

Pour envoyer plusieurs paiements en une seule requête (traitement asynchrone).

**POST** `/v1/payout-batch`

```bash
curl -X POST https://api.wave.com/v1/payout-batch \
  -H "Authorization: Bearer $WAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{
    "payouts": [
      {
        "currency": "XOF",
        "mobile": "+221761110000",
        "receive_amount": "5000",
        "name": "Fatou Ndiaye",
        "client_reference": "SAL-001"
      },
      {
        "currency": "XOF",
        "mobile": "+221761110001",
        "receive_amount": "7500",
        "name": "Amadou Ba",
        "client_reference": "SAL-002"
      }
    ]
  }'
```

**Réponse :**
```json
{
  "id": "pb-185skxq8g1006"
}
```

**Vérifier le statut du batch :**
```
GET /v1/payout-batch/pb-185skxq8g1006
```

```json
{
  "id": "pb-185skxq8g1006",
  "status": "complete",
  "payouts": [
    { "id": "pt-1", "status": "succeeded", "mobile": "+221761110000", ... },
    { "id": "pt-2", "status": "failed", "mobile": "+221761110001", "payout_error": { ... }, ... }
  ]
}
```

- `status: "processing"` → encore en cours, re-vérifier (poll toutes les 2 secondes)
- `status: "complete"` → terminé, vérifier le statut de chaque payout individuellement

Les payouts dans un batch peuvent avoir des statuts mixtes (certains réussis, d'autres échoués).

## Reverser un payout

**POST** `/v1/payout/:id/reverse`

- **Limite** : 3 jours après la création du payout
- **Idempotent** : appeler deux fois ne crée qu'un seul reversal
- Reverse le montant + les frais
- Retourne HTTP 200 sans corps en cas de succès

```bash
curl -X POST https://api.wave.com/v1/payout/pt-185b5e4b8100c/reverse \
  -H "Authorization: Bearer $WAVE_API_KEY"
```

### Erreurs de reversal

| Code | Description |
|------|-------------|
| `insufficient-funds` | Le destinataire n'a plus le solde suffisant |
| `payout-reversal-time-limit-exceeded` | Délai de 3 jours dépassé |
| `payout-reversal-account-terminated` | Compte du destinataire fermé |

## Vérifier un destinataire

**POST** `/v1/verify_recipient/`

Permet de vérifier le nom et la pièce d'identité d'un destinataire avant d'envoyer un paiement.

```bash
curl -X POST https://api.wave.com/v1/verify_recipient/ \
  -H "Authorization: Bearer $WAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mobile": "+221761110000",
    "name": "Fatou Ndiaye",
    "national_id": "1234567890",
    "amount": "5000",
    "currency": "XOF"
  }'
```

**Réponse :**
```json
{
  "within_limits": true,
  "name_match": "MATCH",
  "national_id_match": "MATCH"
}
```

### Valeurs de correspondance

**name_match :**
| Valeur | Signification |
|--------|---------------|
| `MATCH` | Le nom correspond au dossier Wave |
| `NO_MATCH` | Le numéro existe mais le nom diffère |
| `NAME_NOT_KNOWN` | Numéro inconnu ou pas de KYC-2 |

**national_id_match :**
| Valeur | Signification |
|--------|---------------|
| `MATCH` | La pièce d'identité correspond |
| `NO_MATCH` | Ne correspond à aucune pièce approuvée |
| `ID_NOT_KNOWN` | Pas de pièce d'identité enregistrée |
| `null` | Vérification non activée ou champ omis |

L'algorithme utilise des variations de jaro_winkler avec un seuil de 0.65.

### Rate limit

- **30 vérifications** par numéro de téléphone par fenêtre de 5 minutes
- En cas de dépassement → blocage du numéro pendant **60 minutes**

## Statuts des payouts

| Statut | Description |
|--------|-------------|
| `processing` | En cours de traitement |
| `succeeded` | Paiement réussi |
| `failed` | Paiement échoué |
| `reversed` | Paiement reversé |

## Codes d'erreur

| Code | HTTP | Description |
|------|------|-------------|
| `country-mismatch` | 400 | Destinataire dans un autre pays |
| `currency-mismatch` | 400 | Devise incompatible avec les wallets |
| `idempotency-mismatch` | 409 | Même clé, corps de requête différent |
| `insufficient-funds` | 422 | Solde insuffisant (montant + frais) |
| `recipient-minor` | 422 | Destinataire mineur |
| `recipient-account-blocked` | 422 | Compte destinataire bloqué |
| `recipient-account-inactive` | 422 | Compte destinataire inactif |
| `recipient-limit-exceeded` | 422 | Limite mensuelle du destinataire atteinte |
| `aggregated-merchant-required` | 400 | ID marchand requis mais manquant |

## Stratégie de retry

**Erreurs système (retry avec la même clé d'idempotence) :**
- 408, 500, 503, 5xx → marquer comme `pending`, retry avec backoff exponentiel (démarrer à 1s)

**Rate limiting :**
- 429 → marquer comme `pending`, attendre avant de réessayer

**Erreurs finales (ne pas retry) :**
- Toutes les erreurs de validation (4xx sauf 429)
- Erreurs de solde, limites dépassées, données invalides
