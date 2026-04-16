---
name: wave-aggregated-merchants
description: "Wave Aggregated Merchants API — créer, gérer et supprimer des sous-marchands pour les agrégateurs. Utilise ce skill pour la gestion multi-marchands Wave, marketplace Wave, agrégateur de paiements Wave, sous-marchands Wave."
---

# Wave Aggregated Merchants API

L'API Aggregated Merchants permet aux agrégateurs (plateformes qui gèrent plusieurs marchands) de créer et gérer des sous-marchands sous leur compte Wave Business. Accès réservé aux partenaires sélectionnés.

Pour l'authentification et les types de données, consulter [references/authentication.md](../references/authentication.md).

## Endpoints

| Méthode | Endpoint | Description |
|---------|----------|-------------|
| GET | `/v1/aggregated_merchants` | Lister les marchands |
| POST | `/v1/aggregated_merchants` | Créer un marchand |
| GET | `/v1/aggregated_merchants/:id` | Récupérer un marchand |
| PUT | `/v1/aggregated_merchants/:id` | Mettre à jour un marchand |
| DELETE | `/v1/aggregated_merchants/:id` | Supprimer un marchand |

## Créer un marchand

**POST** `/v1/aggregated_merchants`

### Paramètres

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `name` | string | Oui | Nom unique du marchand (max 255) |
| `business_description` | string | Oui | Description de l'activité |
| `business_type` | string | Oui | `"fintech"` ou `"other"` |
| `business_registration_identifier` | string | Non | Numéro d'enregistrement (max 255) |
| `business_sector` | string | Non | Secteur d'activité (max 255) |
| `website_url` | string | Non | URL du site web (max 2083) |
| `manager_name` | string | Non | Nom du gérant |

### Exemple — cURL

```bash
curl -X POST https://api.wave.com/v1/aggregated_merchants \
  -H "Authorization: Bearer $WAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Boutique Moustafa",
    "business_type": "other",
    "business_description": "Épicerie de quartier à Dakar",
    "website_url": "https://boutique-moustafa.sn",
    "manager_name": "Moustafa Diallo"
  }'
```

### Exemple — TypeScript/Node.js

```typescript
interface AggregatedMerchant {
  id: string;
  name: string;
  business_sector: string | null;
  business_type: 'fintech' | 'other';
  business_registration_identifier: string | null;
  website_url: string | null;
  payout_fee_structure_name: string | null;
  checkout_fee_structure_name: string | null;
  business_description: string;
  manager_name: string | null;
  is_locked: boolean;
  when_created: string;
}

async function createMerchant(params: {
  name: string;
  businessDescription: string;
  businessType: 'fintech' | 'other';
  websiteUrl?: string;
  managerName?: string;
  businessSector?: string;
  registrationId?: string;
}): Promise<AggregatedMerchant> {
  const response = await fetch('https://api.wave.com/v1/aggregated_merchants', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.WAVE_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      name: params.name,
      business_description: params.businessDescription,
      business_type: params.businessType,
      website_url: params.websiteUrl,
      manager_name: params.managerName,
      business_sector: params.businessSector,
      business_registration_identifier: params.registrationId,
    }),
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Wave API error: ${error.code} - ${error.message}`);
  }

  return response.json();
}
```

### Exemple — PHP

```php
function createMerchant(
    string $name,
    string $description,
    string $businessType,
    ?string $websiteUrl = null,
    ?string $managerName = null
): array {
    $body = json_encode(array_filter([
        'name' => $name,
        'business_description' => $description,
        'business_type' => $businessType,
        'website_url' => $websiteUrl,
        'manager_name' => $managerName,
    ]));

    $ch = curl_init('https://api.wave.com/v1/aggregated_merchants');
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
```

### Réponse

```json
{
  "id": "am-7lks22ap113t4",
  "name": "Boutique Moustafa",
  "business_sector": null,
  "business_type": "other",
  "business_registration_identifier": null,
  "website_url": "https://boutique-moustafa.sn",
  "payout_fee_structure_name": null,
  "checkout_fee_structure_name": null,
  "business_description": "Épicerie de quartier à Dakar",
  "manager_name": "Moustafa Diallo",
  "when_created": "2024-01-15T09:56:29Z",
  "is_locked": false
}
```

## Lister les marchands

**GET** `/v1/aggregated_merchants`

Supporte la pagination via `first` et `after`.

```bash
curl -H "Authorization: Bearer $WAVE_API_KEY" \
  "https://api.wave.com/v1/aggregated_merchants?first=10"
```

**Réponse :**
```json
{
  "page_info": {
    "has_next_page": false,
    "end_cursor": "YzE6YW0tMWFrbjhkeWcwMTAwOA"
  },
  "items": [
    { "id": "am-7lks22ap113t4", "name": "Boutique Moustafa", ... }
  ]
}
```

## Récupérer un marchand

```
GET /v1/aggregated_merchants/am-7lks22ap113t4
```

## Mettre à jour un marchand

**PUT** `/v1/aggregated_merchants/:id`

Mêmes champs que la création. Le nom doit rester unique.

**Marchands verrouillés** : après la revue par Wave (attribution des structures de frais), le marchand devient `is_locked: true`. Les mises à jour retournent **403 Forbidden**. La suppression reste possible.

```bash
curl -X PUT https://api.wave.com/v1/aggregated_merchants/am-7lks22ap113t4 \
  -H "Authorization: Bearer $WAVE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Boutique Moustafa - Almadies",
    "business_description": "Épicerie de quartier aux Almadies, Dakar",
    "business_type": "other"
  }'
```

## Supprimer un marchand

**DELETE** `/v1/aggregated_merchants/:id`

Retourne HTTP **204 No Content** (pas de corps de réponse).

```bash
curl -X DELETE https://api.wave.com/v1/aggregated_merchants/am-7lks22ap113t4 \
  -H "Authorization: Bearer $WAVE_API_KEY"
```

## Structures de frais

Attribuées par Wave après revue du marchand :

| Nom | Taux |
|-----|------|
| `one_percent` | 1,00% |
| `one_fifty_bps` | 1,50% |

Les structures sont définies séparément pour le payout (`payout_fee_structure_name`) et le checkout (`checkout_fee_structure_name`).

## Utilisation avec les autres APIs

L'`id` du marchand agrégé s'utilise dans les APIs Checkout et Payout via le champ `aggregated_merchant_id` :

**Checkout :**
```json
{
  "amount": "5000",
  "currency": "XOF",
  "success_url": "https://...",
  "error_url": "https://...",
  "aggregated_merchant_id": "am-7lks22ap113t4"
}
```

**Payout :**
```json
{
  "currency": "XOF",
  "mobile": "+221761110000",
  "receive_amount": "5000",
  "aggregated_merchant_id": "am-7lks22ap113t4"
}
```

## Codes d'erreur spécifiques

| Code | Description |
|------|-------------|
| `duplicate-aggregated-merchant-name` | Le nom existe déjà |
| `record-locked` | Marchand verrouillé, modification impossible |
| `not-found` | Marchand introuvable |
| `no-permission` | Clé API invalide ou permissions insuffisantes |
