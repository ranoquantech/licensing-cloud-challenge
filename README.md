# Licensing Cloud Challenge

## 1. Objectif

Vous devez concevoir et implémenter un **système de gestion de licences cloud** permettant de limiter l’usage d’une plateforme selon les droits attribués à chaque client.

Chaque **client** dispose d’une **licence numérique** définissant notamment :
- le nombre maximum d’applications pouvant être enregistrées (`maxApps`) ;
- le nombre maximum d’exécutions autorisées sur une période de 24 heures glissantes (`maxExecutionsPer24h`).

Le système doit garantir que ces limites ne puissent pas être dépassées, même en cas d’appels concurrents.

---

## 2. Contexte général

La plateforme exécute des traitements (“jobs”) pour le compte de clients identifiés par leur licence.
Une application peut embarquer plusieurs jobs et un job est défini pour une et une seule application.

Chaque requête effectuée par un client est accompagnée d’un jeton représentant sa licence.  
Ce jeton contient les informations nécessaires à la vérification de ses droits (identité, limites, période de validité, statut, etc.).

Le système doit :
1. Vérifier la **validité** de la licence (authenticité, date de validité, statut actif).  
2. Appliquer les **restrictions** d’usage définies par la licence.  
3. Empêcher toute exécution supplémentaire en cas de dépassement de quota.

---

## 3. Fonctionnalités attendues

Le projet doit exposer au minimum les endpoints suivants :

| Endpoint | Description |
|-----------|--------------|
| `POST /apps/register` | Enregistre une nouvelle application pour le client. Doit refuser l’opération si `maxApps` est atteint. |
| `POST /jobs/start` | Démarre une exécution (vérifie le quota 24h). |
| `POST /jobs/finish` | Termine une exécution (facultatif selon le design). |

**Comportements attendus :**
- Une requête au-delà du quota autorisé doit être refusée.  
- Les exécutions plus anciennes que 24 heures ne doivent plus être comptabilisées.  
- Une licence expirée ou révoquée doit bloquer toute opération.

---

## 4. Sécurité

La gestion des licences doit être sécurisée.  
Le format du jeton est libre, mais le candidat doit :
- garantir la **signature et la vérification** côté serveur (aucune confiance côté client) ;
- inclure les dates de validité (`validFrom`, `validTo`) et l’état de la licence (`ACTIVE`, `SUSPENDED`, etc.) ;
- protéger les communications et les échanges de clés (dans la mesure du possible);
- éviter toute falsification possible du jeton.

La solution doit rester conforme aux bonnes pratiques de sécurité applicative et réseau.

---

## 5. Fenêtre glissante

Le quota `maxExecutionsPer24h` doit être appliqué sur une **fenêtre glissante de 24 heures** :  
le système ne compte que les exécutions réalisées dans les 24 dernières heures, à partir de l’instant courant.  
Les exécutions plus anciennes sont ignorées automatiquement.

Le mécanisme choisi pour cette fenêtre (ex. Redis, base SQL, mémoire interne, etc.) est libre, mais doit être expliqué et justifié.

---

## 6. Liberté et analyse technique

Le candidat est libre de :
- choisir le langage et le framework (Spring Boot (différentiateur si choisi), Node.js, Go, Python, etc.) ;
- déterminer l’architecture (monolithique, modulaire ou distribuée) ;
- choisir les mécanismes de stockage et de calcul des quotas.
- implémenter une interface simple, efficace et fonctionnelle de gestion du licensing (creation, revocation de license, upgrade, etc...) (différentiateur pour les profils fullStack)
- améliorer et proposer d'autres fonctionnalités

Cependant, il est attendu une **analyse technique claire** :
- exposer les **options envisagées** (par exemple plusieurs approches possibles pour la gestion de la fenêtre glissante ou la synchronisation concurrente) ;
- expliquer le **choix retenu** et ses avantages/inconvénients ;
- préciser comment la solution assure la **cohérence**, la **sécurité** et la **scalabilité**.

---

## 7. Ce qui sera évalué

- Pertinence du raisonnement architectural et des choix techniques.  
- Robustesse du contrôle de licence et absence de dépassement possible.  
- Cohérence de la sécurité et de la validation des jetons.  
- Qualité et clarté du code (lisibilité, organisation, documentation).  
- Simplicité de mise en œuvre et reproductibilité.  
- Présence de tests et d’un scénario démontrant le bon fonctionnement.

---

## 8. Environnement et exécution

L’environnement doit être démarré avec **Docker Compose**.  
L’objectif est que le projet puisse être exécuté simplement à l’aide d'une seule commande telle que :

```bash
docker compose up
```

Toute dépendance ou configuration particulière devra être embarquée dans le processus de conteneurisation.

---

## 9. Scénario de test attendu

1. Création d’une licence :
```json
{
  "tenantId": "acme",
  "maxApps": 2,
  "maxExecutionsPer24h": 100,
  "validFrom": "2025-11-01T00:00:00Z",
  "validTo": "2025-12-01T00:00:00Z",
  "status": "ACTIVE"
}
```

2. Enregistrement de deux applications → accepté.  
3. Tentative d’enregistrement d’une troisième → refusée.  
4. Exécution de 100 jobs sur 24 heures → accepté.  
5. 101ᵉ exécution dans la même période → refusée.

---

## 10. Livrable attendu

Le dépôt doit contenir :
- le code source fonctionnel ;
- un court document expliquant les choix techniques effectués et les options écartées ;
- un jeu de commandes ou une collection Postman permettant de valider le comportement.

---

## 11. Structure suggérée du dépôt

```
licensing-challenge/
├── README.md                 # Ce document
├── docker-compose.yml        # (optionnel) environnement local
├── gateway/                  # API principale
│   ├── src/
│   └── README.md
├── licensing-service/        # (optionnel) gestion et validation des licences
│   ├── src/
│   └── README.md
├── metering-service/         # (optionnel) gestion du quota et de la fenêtre glissante
│   ├── src/
│   └── README.md
└── tests/
    └── scenario.postman.json # (ou équivalent)
```

---

## 12. Délais de rendu estimé

Durée estimée : **5 jours** .  
L’objectif est de démontrer votre capacité à raisonner, structurer et concevoir une solution fiable et maintenable, pas d'avoir une solution complète.

