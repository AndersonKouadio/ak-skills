---
name: wave-webhooks
description: "Wave Webhooks — recevoir les notifications d'événements en temps réel, vérifier les signatures HMAC-SHA256, traiter les événements checkout/B2B/merchant. Utilise ce skill pour configurer les webhooks Wave, vérifier la signature Wave, traiter les événements de paiement Wave, intégrer les notifications Wave."
---

# Wave Webhooks

Les webhooks Wave permettent de recevoir des notifications en temps réel quand des événements se produisent (paiement reçu, checkout complété, etc.). Votre serveur reçoit un POST HTTP au lieu de devoir interroger l'API régulièrement.

Pour l'authentification commune, consulter [references/authentication.md](../references/authentication.md).

## Prérequis de l'endpoint

Votre endpoint webhook doit :
- Accepter les requêtes HTTP POST
- Utiliser HTTPS avec un certificat SSL valide
- Répondre en moins de **5 secondes**
- Retourner un code HTTP **2xx** pour confirmer la réception
- Gérer les événements dupliqués de manière idempotente

## Sécurité

### Option 1 : Shared Secret (basique)

Wave inclut le secret dans le header Authorization :
```
Authorization: Bearer votre_webhook_secret
```

Risque : si le header est logué, un attaquant peut forger des requêtes.

### Option 2 : Signing Secret (recommandé en production)

Wave signe le payload avec HMAC-SHA256 et inclut la signature dans le header `Wave-Signature` :

```
Wave-Signature: t=1639081943,v1=942119aedf9fa377844cf010785fe14ef8478c72af0b73d62ea3941335b526a8
```

### Vérification de la signature

**TypeScript/Node.js :**

```typescript
import crypto from 'crypto';

function verifyWaveWebhook(
  rawBody: string,
  waveSignatureHeader: string,
  signingSecret: string,
  toleranceSeconds = 300
): boolean {
  // 1. Extraire timestamp et signatures
  const parts = waveSignatureHeader.split(',');
  const timestamp = parts
    .find(p => p.startsWith('t='))
    ?.slice(2);
  const signatures = parts
    .filter(p => p.startsWith('v1='))
    .map(p => p.slice(3));

  if (!timestamp || signatures.length === 0) {
    return false;
  }

  // 2. Vérifier le timestamp (protection anti-replay)
  const now = Math.floor(Date.now() / 1000);
  if (Math.abs(now - parseInt(timestamp)) > toleranceSeconds) {
    return false;
  }

  // 3. Calculer la signature attendue
  const payload = `${timestamp}${rawBody}`;
  const expectedSignature = crypto
    .createHmac('sha256', signingSecret)
    .update(payload)
    .digest('hex');

  // 4. Comparer (timing-safe)
  return signatures.some(sig =>
    crypto.timingSafeEqual(
      Buffer.from(sig, 'hex'),
      Buffer.from(expectedSignature, 'hex')
    )
  );
}
```

**Express.js (middleware complet) :**

```typescript
import express from 'express';

const app = express();

// Le corps brut est indispensable pour la vérification de signature.
// JSON.parse modifie le formatage et invalide le hash.
app.post('/webhook/wave',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const rawBody = req.body.toString('utf-8');
    const signature = req.headers['wave-signature'] as string;

    if (!verifyWaveWebhook(rawBody, signature, process.env.WAVE_WEBHOOK_SECRET!)) {
      return res.status(401).json({ error: 'Invalid signature' });
    }

    const event = JSON.parse(rawBody);

    // Répondre immédiatement avec 200, traiter en arrière-plan
    res.status(200).json({ received: true });

    // Traitement asynchrone
    processWebhookEvent(event).catch(console.error);
  }
);
```

**PHP :**

```php
function verifyWaveWebhook(
    string $rawBody,
    string $signatureHeader,
    string $signingSecret,
    int $toleranceSeconds = 300
): bool {
    // Extraire timestamp et signatures
    $parts = explode(',', $signatureHeader);
    $timestamp = null;
    $signatures = [];

    foreach ($parts as $part) {
        if (str_starts_with($part, 't=')) {
            $timestamp = substr($part, 2);
        } elseif (str_starts_with($part, 'v1=')) {
            $signatures[] = substr($part, 3);
        }
    }

    if (!$timestamp || empty($signatures)) {
        return false;
    }

    // Vérifier le timestamp
    if (abs(time() - intval($timestamp)) > $toleranceSeconds) {
        return false;
    }

    // Calculer et comparer
    $expected = hash_hmac('sha256', $timestamp . $rawBody, $signingSecret);
    foreach ($signatures as $sig) {
        if (hash_equals($expected, $sig)) {
            return true;
        }
    }

    return false;
}

// Utilisation dans un controller
$rawBody = file_get_contents('php://input');
$signatureHeader = $_SERVER['HTTP_WAVE_SIGNATURE'] ?? '';

if (!verifyWaveWebhook($rawBody, $signatureHeader, $_ENV['WAVE_WEBHOOK_SECRET'])) {
    http_response_code(401);
    exit('Invalid signature');
}

$event = json_decode($rawBody, true);
http_response_code(200);
echo json_encode(['received' => true]);

// Traiter l'événement
processEvent($event);
```

**Piège courant** : les frameworks qui parsent automatiquement le JSON (comme Express avec `express.json()`) modifient le formatage du corps. La signature est calculée sur le corps **brut** — utiliser `express.raw()` ou équivalent.

## Événements

### checkout.session.completed

Déclenché quand un client complète un paiement Checkout avec succès.

```json
{
  "id": "EV_QvEZuDSQbLdI",
  "type": "checkout.session.completed",
  "data": {
    "id": "cos-18qq25rgr100a",
    "amount": "1000",
    "currency": "XOF",
    "payment_status": "succeeded",
    "checkout_status": "complete",
    "client_reference": "CMD-2024-001",
    "transaction_id": "TCN4Y4ZC3FM",
    "when_completed": "2024-01-15T10:15:32Z"
  }
}
```

### checkout.session.payment_failed

Déclenché quand une tentative de paiement Checkout échoue.

```json
{
  "id": "EV_8bO0d7TwW6Eq",
  "type": "checkout.session.payment_failed",
  "data": {
    "id": "cos-18qq25rgr100a",
    "payment_status": "failed",
    "last_payment_error": {
      "code": "insufficient-funds",
      "message": "Insufficient balance"
    }
  }
}
```

### merchant.payment_received

Déclenché quand un client paie directement le business (QR code, numéro marchand).

```json
{
  "id": "AE_ijzo7oGgrlM8",
  "type": "merchant.payment_received",
  "data": {
    "id": "T_46HS5COOWE",
    "amount": "990",
    "fee": "10",
    "currency": "XOF",
    "sender_mobile": "+221761110001",
    "merchant_name": "Ma Boutique",
    "custom_fields": {
      "account_number": "abc-123"
    },
    "when_created": "2024-01-15T10:13:04Z"
  }
}
```

### b2b.payment_received

Déclenché quand un business reçoit un paiement d'un autre business Wave.

```json
{
  "id": "AE_ijzo7oGgrlM8",
  "type": "b2b.payment_received",
  "data": {
    "id": "b2b-1ndjb8dj81008",
    "amount": "39800",
    "currency": "XOF",
    "sender_id": "M_qn0zhfcKV1Tl",
    "client_reference": "FACT-456",
    "when_created": "2024-01-15T14:28:15Z"
  }
}
```

### b2b.payment_failed

Déclenché quand un paiement B2B échoue.

### test.test_event

Événement de test envoyé manuellement depuis le portail Wave Business.

## Structure commune des événements

```typescript
interface WaveWebhookEvent {
  id: string;     // Identifiant unique de l'événement
  type: string;   // Type d'événement
  data: object;   // Données spécifiques à l'événement
}
```

## Livraison et retries

- Si votre serveur ne répond pas avec un HTTP 2xx, Wave **réessaie pendant 3 jours**
- Répondre immédiatement avec 200, puis traiter de manière asynchrone
- Les événements peuvent être manqués, livrés plusieurs fois ou arriver dans le désordre
- Utiliser le champ `id` pour la déduplication (idempotence)

## Traitement recommandé

```typescript
async function processWebhookEvent(event: WaveWebhookEvent) {
  // Vérifier si l'événement a déjà été traité (idempotence)
  const alreadyProcessed = await db.webhookEvents.findOne({ eventId: event.id });
  if (alreadyProcessed) return;

  // Marquer comme en cours de traitement
  await db.webhookEvents.create({ eventId: event.id, status: 'processing' });

  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutCompleted(event.data);
      break;
    case 'checkout.session.payment_failed':
      await handleCheckoutFailed(event.data);
      break;
    case 'merchant.payment_received':
      await handleMerchantPayment(event.data);
      break;
    case 'b2b.payment_received':
      await handleB2BPayment(event.data);
      break;
    default:
      console.log(`Événement non géré: ${event.type}`);
  }

  // Marquer comme traité
  await db.webhookEvents.updateOne(
    { eventId: event.id },
    { status: 'processed' }
  );
}
```

## IPs Wave à whitelister

Whitelister toutes ces IPs pour recevoir les webhooks :

```
104.155.43.220/32
34.140.23.175/32
34.22.138.147/32
34.76.157.22/32
34.78.253.137/32
34.79.119.200/32
35.189.207.30/32
35.195.255.192/32
35.205.122.113/32
35.205.190.121/32
35.233.61.130/32
35.240.61.196/32
35.240.75.65/32
35.241.190.127/32
35.241.219.1/32
```

Toutes les IPs doivent être whitelistées — sinon des rejets aléatoires se produiront.

## Rotation des secrets

1. Dupliquer le webhook existant pour générer un nouveau secret
2. Le système reçoit les notifications en double (ancien + nouveau secret)
3. Mettre à jour votre système pour utiliser le nouveau secret
4. Vérifier que la validation fonctionne
5. Supprimer l'ancien webhook

## Configuration dans le portail Wave Business

1. Accéder à la section Webhooks
2. Cliquer "Add New Webhook"
3. Fournir l'URL de l'endpoint (HTTPS requis)
4. Choisir la stratégie de sécurité (Shared Secret ou Signing Secret)
5. Sélectionner les événements à recevoir
6. **Sauvegarder le secret immédiatement** — il ne peut pas être récupéré après
