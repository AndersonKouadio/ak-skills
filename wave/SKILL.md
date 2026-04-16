---
name: wave
description: "Boîte à outils Wave Mobile Money API — checkout (paiements), payout (envois d'argent), balance (solde/transactions), webhooks, aggregated merchants. Déclenché pour tout travail avec l'API Wave, intégration Wave, paiement mobile Wave, mobile money Afrique de l'Ouest. Utilise ce skill dès que l'utilisateur mentionne Wave, paiement mobile, checkout Wave, payout Wave, solde Wave, webhook Wave, ou veut intégrer Wave dans une application."
---

# Wave Mobile Money API Toolkit

Guides complets pour l'intégration de l'API Wave — la plateforme de mobile money leader en Afrique de l'Ouest (Sénégal, Côte d'Ivoire, Mali, Burkina Faso, Gambie, Ouganda).

## API disponibles

| API | Fichier | Utilisation |
|-----|---------|-------------|
| Checkout | [wave-checkout/SKILL.md](wave-checkout/SKILL.md) | Créer des sessions de paiement, rediriger les clients, gérer les remboursements |
| Payout | [wave-payout/SKILL.md](wave-payout/SKILL.md) | Envoyer de l'argent (unitaire/batch), reverser, vérifier les destinataires |
| Balance | [wave-balance/SKILL.md](wave-balance/SKILL.md) | Consulter le solde, lister les transactions, réconciliation |
| Webhooks | [wave-webhooks/SKILL.md](wave-webhooks/SKILL.md) | Recevoir les notifications d'événements, vérifier les signatures |
| Aggregated Merchants | [wave-aggregated-merchants/SKILL.md](wave-aggregated-merchants/SKILL.md) | Gérer les sous-marchands (agrégateurs uniquement) |

## Référence commune

| Sujet | Fichier |
|-------|---------|
| Authentification, signature, types de données, pagination | [references/authentication.md](references/authentication.md) |

## Informations essentielles

- **URL de base** : `https://api.wave.com`
- **Authentification** : Bearer token (`Authorization: Bearer wave_sn_prod_...`)
- **Signature optionnelle** : HMAC-SHA256 via header `Wave-Signature`
- **Format** : JSON (UTF-8), HTTPS uniquement
- **Devises** : XOF (Franc CFA BCEAO, 0 décimale), XAF, etc.
- **Téléphones** : Format E.164 (`+221...`, `+225...`)

## Workflow typique d'intégration

1. **Créer une clé API** dans le portail Wave Business (section développeur)
2. **Choisir les APIs** nécessaires selon le cas d'usage :
   - Accepter des paiements → Checkout API
   - Envoyer de l'argent → Payout API
   - Suivre les transactions → Balance API
   - Recevoir des notifications → Webhooks
3. **Configurer la sécurité** : signature de requête + IP whitelisting en production
4. **Implémenter les webhooks** pour les notifications temps réel
5. **Tester** avec des transactions réelles (pas d'environnement sandbox)

## Choix de l'API selon le cas d'usage

| Cas d'usage | API principale | APIs complémentaires |
|-------------|---------------|---------------------|
| E-commerce (accepter des paiements en ligne) | Checkout | Webhooks, Balance |
| Paiement de salaires / transferts | Payout | Balance |
| Marketplace / agrégateur | Checkout + Aggregated Merchants | Webhooks, Payout, Balance |
| Réconciliation comptable | Balance | — |
| Remboursement client | Checkout (refund) ou Balance (refund) | — |
