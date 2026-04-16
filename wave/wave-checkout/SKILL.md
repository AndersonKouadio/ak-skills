---
name: wave-checkout
description: "Wave Checkout API — créer des sessions de paiement, rediriger les clients vers Wave, gérer les remboursements et expirations. Utilise ce skill pour tout paiement entrant via Wave, intégration e-commerce Wave, bouton 'Payer avec Wave', QR code de paiement Wave."
---

# Wave Checkout API

L'API Checkout permet de créer des sessions de paiement pour collecter de l'argent auprès des clients Wave. Le client est redirigé vers Wave pour autoriser le paiement, puis renvoyé vers votre application.

Pour l'authentification et les types de données, consulter [references/authentication.md](../references/authentication.md).

## Endpoints

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| POST | `/v1/checkout/sessions` | Créer une session de paiement |
| GET | `/v1/checkout/sessions/:id` | Récupérer par ID de session |
| GET | `/v1/checkout/sessions?transaction_id=X` | Récupérer par ID de transaction |
| GET | `/v1/checkout/sessions/search?client_reference=X` | Rechercher par référence client |
| POST | `/v1/checkout/sessions/:id/refund` | Rembourser un paiement |
| POST | `/v1/checkout/sessions/:id/expire` | Expirer une session ouverte |

## Créer une session de paiement

**POST** `/v1/checkout/sessions`

### Paramètres requis

| Champ | Type | Description |
|-------|------|-------------|
| `amount` | string | Montant du paiement (0-2 décimales, XOF = 0) |
| `currency` | string | Code ISO 4217 (ex: `"XOF"`) |
| `success_url` | string | URL HTTPS de redirection en cas de succès |
| `error_url` | string | URL HTTPS de redirection en cas d'erreur |

### Paramètres optionnels

| Champ | Type | Description |
|-------|------|-------------|
| `client_reference` | string | Identifiant de corrélation (max 255 chars) |
| `restrict_payer_mobile` | string | Numéro E.164 pour restreindre le payeur |
| `aggregated_merchant_id` | string | ID du sous-marchand (agrégateurs uniquement) |

### Exemple — cURL

```bash
curl -X POST https://api.wave.com/v1/checkout/sessions \
  -H "Authorization: Bearer $WAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": "1000",
    "currency": "XOF",
    "success_url": "https://monsite.com/paiement/succes",
    "error_url": "https://monsite.com/paiement/erreur",
    "client_reference": "CMD-2024-001"
  }'
```

### Exemple — TypeScript/Node.js

```typescript
interface CheckoutSession {
  id: string;
  amount: string;
  currency: string;
  checkout_status: 'open' | 'complete' | 'expired';
  payment_status: 'processing' | 'cancelled' | 'succeeded';
  transaction_id: string | null;
  wave_launch_url: string;
  client_reference: string | null;
  business_name: string;
  when_created: string;
  when_expires: string;
  when_completed: string | null;
  last_payment_error: { code: string; message: string } | null;
}

async function createCheckoutSession(params: {
  amount: string;
  currency: string;
  successUrl: string;
  errorUrl: string;
  clientReference?: string;
}): Promise<CheckoutSession> {
  const response = await fetch('https://api.wave.com/v1/checkout/sessions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.WAVE_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      amount: params.amount,
      currency: params.currency,
      success_url: params.successUrl,
      error_url: params.errorUrl,
      client_reference: params.clientReference,
    }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Wave API error: ${error.code} - ${error.message}`);
  }

  return response.json();
}

// Utilisation
const session = await createCheckoutSession({
  amount: '5000',
  currency: 'XOF',
  successUrl: 'https://monsite.com/succes',
  errorUrl: 'https://monsite.com/erreur',
  clientReference: 'CMD-2024-001',
});

// Rediriger le client vers Wave
// session.wave_launch_url → URL à ouvrir pour le paiement
```

### Exemple — PHP

```php
function createCheckoutSession(
    string $amount,
    string $currency,
    string $successUrl,
    string $errorUrl,
    ?string $clientReference = null
): array {
    $body = json_encode(array_filter([
        'amount' => $amount,
        'currency' => $currency,
        'success_url' => $successUrl,
        'error_url' => $errorUrl,
        'client_reference' => $clientReference,
    ]));

    $ch = curl_init('https://api.wave.com/v1/checkout/sessions');
    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => $body,
        CURLOPT_HTTPHEADER => [
            'Authorization: Bearer ' . $_ENV['WAVE_API_KEY'],
            'Content-Type: application/json',
        ],
    ]);

    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);

    if ($httpCode !== 200) {
        throw new Exception("Wave API error: $response");
    }

    return json_decode($response, true);
}

$session = createCheckoutSession('5000', 'XOF',
    'https://monsite.com/succes',
    'https://monsite.com/erreur',
    'CMD-2024-001'
);

// Rediriger : header('Location: ' . $session['wave_launch_url']);
```

### Réponse

```json
{
  "id": "cos-18qq25rgr100a",
  "amount": "5000",
  "currency": "XOF",
  "checkout_status": "open",
  "payment_status": "processing",
  "transaction_id": null,
  "wave_launch_url": "https://pay.wave.com/c/cos-18qq25rgr100a",
  "client_reference": "CMD-2024-001",
  "business_name": "Ma Boutique",
  "when_created": "2024-01-15T10:00:00Z",
  "when_expires": "2024-01-15T10:30:00Z",
  "when_completed": null,
  "last_payment_error": null
}
```

## Récupérer une session

**Par ID de session :**
```
GET /v1/checkout/sessions/cos-18qq25rgr100a
```

**Par ID de transaction :**
```
GET /v1/checkout/sessions?transaction_id=FAH.4827.1734
```

**Par référence client (retourne un tableau) :**
```
GET /v1/checkout/sessions/search?client_reference=CMD-2024-001
```

## Rembourser un paiement

**POST** `/v1/checkout/sessions/:id/refund`

- Idempotent : appeler deux fois ne crée qu'un seul remboursement
- Pas de corps de requête requis
- Retourne HTTP 200 sans corps en cas de succès
- Rembourse le montant total + frais

```bash
curl -X POST https://api.wave.com/v1/checkout/sessions/cos-18qq25rgr100a/refund \
  -H "Authorization: Bearer $WAVE_API_KEY"
```

## Expirer une session

**POST** `/v1/checkout/sessions/:id/expire`

- Retourne HTTP 200 sans corps en cas de succès
- Retourne 409 si la session est déjà complétée ou expirée
- Retourne 404 si la session n'existe pas

```bash
curl -X POST https://api.wave.com/v1/checkout/sessions/cos-18qq25rgr100a/expire \
  -H "Authorization: Bearer $WAVE_API_KEY"
```

## Statuts

### checkout_status

| Valeur | Description |
|--------|-------------|
| `open` | Session créée, en attente du paiement |
| `complete` | Paiement terminé (succès ou échec) |
| `expired` | Session expirée sans paiement |

### payment_status

| Valeur | Description |
|--------|-------------|
| `processing` | Paiement en cours de traitement |
| `cancelled` | Paiement annulé |
| `succeeded` | Paiement réussi |

## Erreurs spécifiques au Checkout

| Code | Description |
|------|-------------|
| `checkout-refund-failed` | Remboursement échoué (paiement pas en état succès) |
| `checkout-session-not-found` | Session introuvable |
| `request-validation-error` | Requête invalide (champ manquant, type incorrect) |
| `unauthorized-wallet` | Compte non autorisé pour cette API |

## Erreurs de paiement (via webhook)

| Code | Description |
|------|-------------|
| `blocked-account` | Compte bloqué |
| `cross-border-payment-not-allowed` | Paiement transfrontalier interdit |
| `customer-age-restricted` | Client mineur |
| `insufficient-funds` | Solde insuffisant |
| `kyb-limits-exceeded` | Limites du compte business dépassées |
| `payer-mobile-mismatch` | Numéro ne correspond pas au payeur restreint |
| `payment-failure` | Erreur technique |

## Flux d'intégration typique

```
1. Votre serveur → POST /v1/checkout/sessions → Wave API
2. Wave API → retourne wave_launch_url → Votre serveur
3. Votre serveur → redirige le client vers wave_launch_url
4. Client → paie sur Wave → Wave redirige vers success_url ou error_url
5. Wave → envoie webhook checkout.session.completed → Votre serveur
6. Votre serveur → vérifie le webhook → met à jour la commande
```

Le webhook est la source de vérité pour confirmer le paiement — ne pas se fier uniquement à la redirection `success_url` car le client peut la manipuler.
