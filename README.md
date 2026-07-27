# vps-watch — surveillance externe des SaaS

Sonde les services hébergés sur le VPS mutualisé **depuis l'infrastructure
GitHub**, et alerte sur Slack en cas d'indisponibilité.

## Pourquoi ce dépôt existe

Le VPS héberge déjà [Uptime Kuma](https://status.ukulimap.com), qui surveille
les mêmes services avec plus de finesse. Mais **Uptime Kuma tourne sur le
serveur qu'il surveille** : si la machine s'arrête, si le réseau tombe ou si
Docker meurt, la supervision disparaît avec eux et personne n'est prévenu.

Ce dépôt couvre exactement ce point aveugle, et rien d'autre. Les deux sont
complémentaires :

| | Uptime Kuma (sur le VPS) | Ce dépôt (GitHub) |
|---|---|---|
| Granularité | 11 sondes, historique, pages de statut | 8 URL, aucun historique |
| Fréquence | 60 s | 10 min (planificateur GitHub, parfois retardé) |
| Survit à une panne du VPS | ❌ | ✅ |

## Dépôt public — délibérément

Sur un dépôt **privé**, chaque exécution de workflow consomme au minimum une
minute du quota gratuit (2000/mois). À raison d'une sonde toutes les 10 minutes,
soit ~4320 exécutions mensuelles, le quota serait épuisé — et avec lui les
déploiements des trois SaaS, qui vivent dans des dépôts privés. Sur un dépôt
public, les minutes sont illimitées.

Ce dépôt ne contient donc que des URL déjà publiques. Le jeton Telegram est
stocké dans les *secrets* chiffrés du dépôt, jamais dans le code.

## Configuration requise

Settings → Secrets and variables → Actions :

| Secret | Où l'obtenir |
|---|---|
| `SLACK_WEBHOOK_URL` | [api.slack.com/apps](https://api.slack.com/apps) → *Create New App* (from scratch) → **Incoming Webhooks** → activer → *Add New Webhook to Workspace* → choisir le canal |

Cette URL **est** le secret : quiconque la détient peut publier dans le canal.
Elle n'a pas d'expiration ; en cas de fuite, la révoquer depuis la même page.

Tester sans attendre le planificateur : onglet **Actions** → *Surveillance
externe des SaaS* → **Run workflow**.

## Limites connues

- Le planificateur GitHub n'est pas ponctuel : un `*/10` peut s'exécuter avec
  plusieurs minutes de retard, voire sauter un créneau en période de charge.
- Une panne prolongée déclenche une alerte **à chaque exécution** (toutes les
  10 min). Il n'y a pas de mémoire d'état entre deux exécutions ; ajouter une
  déduplication supposerait de stocker l'état quelque part.
- GitHub désactive les workflows planifiés d'un dépôt resté **60 jours sans
  commit**. Un commit trivial suffit à réarmer le planificateur — et les
  commits poussés par le robot Actions ne comptent pas comme activité.
