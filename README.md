# Signal DCA MM200 → Telegram (n8n, Coinbase)

Un workflow **n8n gratuit** qui calcule chaque semaine un **signal DCA Bitcoin basé sur la moyenne mobile 200 jours (MM200)** et vous l'envoie sur Telegram.

- **Aucune clé API Coinbase** : les prix viennent de l'API publique de Coinbase Exchange.
- **Aucun ordre passé** : le workflow calcule un montant suggéré, vous décidez.
- **5 minutes d'installation** : un fichier JSON à importer, un bot Telegram.

> Outil logiciel à visée pédagogique. Ce n'est pas un conseil en investissement. Les crypto-actifs peuvent perdre tout ou partie de leur valeur.

## Ce que vous recevez chaque lundi

```
📊 Signal DCA BTC-EUR — 2026-09-23
Prix : 75 203,45 € | MM200 : 61 155,29 € (x1.23)
Régime : HAUSSIER (prix > MM) → coefficient x1.5

👉 Montant suggéré cette semaine : 30,00 €.
```

## La règle

Le DCA (investissement programmé) consiste à acheter un montant fixe à intervalle régulier. Cette variante module le montant selon la tendance de fond :

| Situation | Coefficient par défaut | Avec 20 € de base |
|---|---|---|
| Prix **au-dessus** de la MM200 (régime haussier) | × 1,5 | 30 € |
| Prix **en dessous** de la MM200 (régime baissier) | × 0,5 (0 = pause) | 10 € |

L'objectif n'est pas de « battre le marché » mais de **réduire l'exposition quand la tendance longue est cassée**. Contrepartie assumée : on achète davantage quand les prix sont déjà montés. C'est une règle plus prudente en marché baissier prolongé, pas une règle meilleure dans tous les scénarios.

## Installation

1. **n8n** : une instance cloud ou auto-hébergée (version 1.x).
2. **Bot Telegram** : créez-le avec [@BotFather](https://t.me/BotFather), gardez le jeton, puis récupérez votre identifiant de chat (par exemple avec [@userinfobot](https://t.me/userinfobot)).
3. **Import** : dans n8n, *Workflows → Import from File* → `workflows/signal-dca-mm200-telegram.json`.
4. **Credential** : ouvrez le nœud *Telegram*, créez un credential « Telegram API » avec le jeton du bot.
5. **Configuration** : dans le nœud *Configuration*, renseignez `TELEGRAM_CHAT_ID` et ajustez les réglages.
6. **Test** : cliquez sur *Execute workflow*. Le message doit arriver sur Telegram.
7. **Activation** : activez le workflow. Il tournera chaque lundi à 9 h (heure de l'instance).

## Réglages (nœud « Configuration »)

| Paramètre | Défaut | Rôle |
|---|---|---|
| `PRODUCT_ID` | `BTC-EUR` | Paire Coinbase (`ETH-EUR`, `BTC-USD`…) |
| `DCA_AMOUNT_EUR` | `20` | Montant de base par semaine |
| `MA_DAYS` | `200` | Longueur de la moyenne mobile (max. 300) |
| `MULTIPLIER_ABOVE_MA` | `1.5` | Coefficient en régime haussier |
| `MULTIPLIER_BELOW_MA` | `0.5` | Coefficient en régime baissier (`0` = pause) |
| `TELEGRAM_CHAT_ID` | — | Votre identifiant de chat Telegram |

## Fonctionnement

```
Planification (lundi 9 h)
  → Configuration
  → GET api.exchange.coinbase.com/products/{PRODUCT_ID}/candles?granularity=86400  (300 bougies, public)
  → Code : tri chronologique, MM sur les MA_DAYS dernières clôtures, coefficient, montant
  → Code : message
  → Telegram
```

## Aller plus loin

Ce workflow est la brique « signal » d'un protocole plus complet décrit, avec les chiffres réels de 17 ordres (frais de 1,2 % en taker, DCA arrêté faute de solde), dans l'article **[DCA automatisé sur Coinbase : mon protocole et mes vrais chiffres](https://utikcoin.com/article/dca-automatise-coinbase-protocole)**.

Le **[Kit Crypto n8n](https://utikcoin.com/kit)** (payant) ajoute l'exécution de l'ordre sur Coinbase Advanced Trade (clé CDP sans droit de retrait, mode simulation par défaut), les alertes de prix, le rapport de portefeuille quotidien et l'export fiscal mensuel au format Koinly.

## Questions fréquentes

**Le workflow a-t-il accès à mon compte Coinbase ?**
Non. Il lit uniquement les prix publics. Aucune clé, aucun ordre.

**Pourquoi 300 bougies pour une MM200 ?**
C'est le maximum renvoyé par l'API publique en une requête ; il laisse de la marge en cas de jours manquants.

**Puis-je l'utiliser pour l'Ether ?**
Oui : `PRODUCT_ID` = `ETH-EUR`. Dupliquez le workflow pour suivre plusieurs actifs.

## Licence

MIT. Voir [LICENSE](LICENSE). Maintenu par [UTIKCOIN](https://utikcoin.com), média crypto indépendant.
