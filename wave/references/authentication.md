# Authentification Wave API

## URL de base

Toutes les requêtes : `https://api.wave.com`

## Clé API (Bearer Token)

Chaque requête nécessite un header `Authorization` :

```
Authorization: Bearer wave_sn_prod_YhUNb9d...i4bA6
```

- Les clés sont créées dans le portail Wave Business (section développeur)
- Chaque clé est liée à un seul wallet business
- Format : `wave_sn_prod_[alphanumérique]`
- Ne jamais exposer côté client

## Signature de requête (optionnel, recommandé en production)

Quand activée sur une clé API, chaque requête doit inclure un header `Wave-Signature` :

```
Wave-Signature: t={timestamp},v1={signature}
```

### Construction de la signature

1. Obtenir le timestamp Unix courant (secondes entières)
2. Concaténer `timestamp` + corps brut de la requête
3. Calculer HMAC-SHA256 avec le signing secret comme clé
4. Formater : `t={timestamp},v1={signature}`

Pour les requêtes GET (sans corps), utiliser une chaîne vide comme payload.

### Validation du timestamp

- Rejet si > 5 minutes dans le passé
- Rejet si > 30 secondes dans le futur
- Le serveur doit être synchronisé (NTP)

### Exemples

**cURL :**
```bash
TIMESTAMP=$(date +%s)
BODY='{"amount": "1000", "currency": "XOF"}'
SIGNING_SECRET="wave_sn_AKS_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

SIGNATURE=$(printf '%s%s' "$TIMESTAMP" "$BODY" \
  | openssl dgst -sha256 -hmac "$SIGNING_SECRET" \
  | sed 's/^.* //')

curl -X POST \
  -H "Authorization: Bearer $API_KEY" \
  -H "Wave-Signature: t=${TIMESTAMP},v1=${SIGNATURE}" \
  -H "Content-Type: application/json" \
  -d "$BODY" \
  https://api.wave.com/v1/checkout/sessions
```

**TypeScript/Node.js :**
```typescript
import crypto from 'crypto';

function createWaveSignature(body: string, signingSecret: string): string {
  const timestamp = Math.floor(Date.now() / 1000);
  const payload = `${timestamp}${body}`;
  const signature = crypto
    .createHmac('sha256', signingSecret)
    .update(payload)
    .digest('hex');
  return `t=${timestamp},v1=${signature}`;
}

// Utilisation
const body = JSON.stringify({ amount: "1000", currency: "XOF" });
const waveSignature = createWaveSignature(body, SIGNING_SECRET);

const response = await fetch('https://api.wave.com/v1/checkout/sessions', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Wave-Signature': waveSignature,
    'Content-Type': 'application/json',
  },
  body,
});
```

**PHP :**
```php
$timestamp = time();
$body = json_encode(['amount' => '1000', 'currency' => 'XOF']);
$signature = hash_hmac('sha256', $timestamp . $body, $signing_secret);
$wave_signature = "t={$timestamp},v1={$signature}";

$ch = curl_init('https://api.wave.com/v1/checkout/sessions');
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => $body,
    CURLOPT_HTTPHEADER => [
        "Authorization: Bearer {$api_key}",
        "Wave-Signature: {$wave_signature}",
        "Content-Type: application/json",
    ],
]);
$result = curl_exec($ch);
curl_close($ch);
```

## IP Whitelisting

Restreint l'accès API aux IPs pré-approuvées.

**Formats supportés :**
- IP unique : `203.0.113.45` (stockée en /32 ou /128)
- Plage CIDR : `203.0.113.0/24`

**Règles :**
- Préfixe minimum : /8 (IPv4), /48 (IPv6)
- Pas d'adresses privées/réservées/loopback
- Pas de plages qui se chevauchent
- Activation automatique dès le premier ajout

**Erreur si IP non autorisée :**
```json
{
  "error": {
    "code": "ip-not-allowed",
    "message": "Request from IP address not in allowlist",
    "httpcode": 403
  }
}
```

## Codes d'erreur d'authentification

| Status | Code | Description |
|--------|------|-------------|
| 401 | `missing-auth-header` | Header Authorization requis |
| 401 | `invalid-auth` | Header non traitable |
| 401 | `api-key-not-provided` | Clé API manquante |
| 401 | `no-matching-api-key` | Clé inexistante |
| 401 | `api-key-revoked` | Clé révoquée |
| 401 | `missing-signature` | Header Wave-Signature manquant |
| 401 | `invalid-signature-format` | Format invalide (doit être `t=...,v1=...`) |
| 401 | `invalid-signature` | Signature ne correspond pas |
| 401 | `invalid-signature-timestamp` | Timestamp invalide |
| 401 | `expired-signature-timestamp` | Timestamp hors fenêtre |
| 403 | `invalid-wallet` | Wallet incompatible |
| 403 | `disabled-wallet` | Wallet temporairement désactivé |
| 403 | `ip-not-allowed` | IP non dans la whitelist |

## Types de données communs

| Type | Format | Exemple |
|------|--------|---------|
| Montant | String, 0-2 décimales (XOF = 0 décimale) | `"1000"`, `"10.50"` |
| Devise | ISO 4217 majuscules | `"XOF"`, `"XAF"` |
| Timestamp | ISO 8601 UTC | `"2024-01-15T14:30:00Z"` |
| Téléphone | E.164 | `"+221761110000"` |
| ID | String, max 20 caractères | `"cos-18qq25rgr100a"` |

## Codes HTTP communs

| Code | Signification |
|------|---------------|
| 200 | Succès |
| 400 | Requête malformée |
| 401 | Non authentifié |
| 403 | Non autorisé |
| 404 | Ressource introuvable |
| 409 | Conflit |
| 422 | Contenu non traitable |
| 429 | Rate limit dépassé |
| 500 | Erreur serveur |
| 503 | Service indisponible |

## Pagination

Les endpoints qui retournent des listes utilisent un système de curseur :

```json
{
  "page_info": {
    "has_next_page": true,
    "end_cursor": "YzE6YW0tMWFrbjhkeWcwMTAwOA"
  },
  "items": [...]
}
```

Pour paginer, passer `after={end_cursor}` au prochain appel jusqu'à `has_next_page: false`.
