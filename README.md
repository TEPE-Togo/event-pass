# 🎟️ EventPass — Test Technique Stagiaire Full Stack

> **Poste visé :** Stagiaire Développeur Full Stack  
> **Stack attendue :** Next.js · Tailwind · FastAPI · PostgreSQL · Redis · Docker  
> **Soumission :** Fork ce repo → développe → envoie le lien de ton repo à [careers@karaba.africa]

---

## Contexte

La billetterie événementielle repose sur trois moments critiques : la **vente** (le billet doit être unique et sécurisé), la **réception** (l'acheteur reçoit un justificatif clair), et le **contrôle d'accès** (le jour J, le scan doit être rapide et fiable).

Ton mission : construire **EventPass**, une plateforme complète de billetterie avec génération de QR codes et interface de scan en temps réel.

---

## Ce que tu vas construire

### 🖥️ Dashboard Organisateur (Frontend — Next.js)

#### Gestion des événements
- Créer / modifier / archiver un événement (`titre`, `description`, `lieu`, `date_debut`, `date_fin`, `image_url`)
- Définir plusieurs **types de billets** par événement (`nom`, `prix`, `quota`, `date_fin_vente`)
  - Exemple : "VIP — 15 000 FCFA — 50 places", "Standard — 5 000 FCFA — 200 places"
- Activer / désactiver la vente pour un type de billet

#### Suivi des ventes
- Liste des commandes avec statut (`payée`, `annulée`)
- Détail d'une commande (acheteur, billets, total, date)
- Compteur en temps réel : places vendues / places restantes par type de billet

#### Interface de scan _(page dédiée, optimisée mobile)_
- Activation de la caméra pour lire un QR code
- Résultat affiché immédiatement après scan :
  - ✅ **Valide** — nom du porteur, type de billet, événement
  - ⚠️ **Déjà scanné** — heure du premier scan
  - ❌ **Invalide** — QR code inconnu ou billet annulé
- Historique des 10 derniers scans de la session

---

### 🌐 Page Publique Événement (Frontend — Next.js)

Une page accessible sans connexion (`/events/{slug}`) :

- Présentation de l'événement (titre, date, lieu, image, description)
- Liste des types de billets disponibles avec prix et places restantes
- Formulaire d'achat :
  - Nom complet, email, numéro de téléphone
  - Sélection du type de billet et quantité
  - Récapitulatif avant confirmation
  - Paiement simulé (bouton "Payer" → succès immédiat)
- Page de confirmation avec lien de téléchargement du billet PDF

---

### 🎫 Billet PDF

Chaque billet généré doit contenir :

- Nom de l'événement, date, lieu
- Nom de l'acheteur
- Type de billet
- **QR code unique** (encodant l'ID du billet)
- Numéro de commande
- Mention "Ce billet est valable pour une seule entrée"

> Un billet = un QR code. Une commande de 3 billets = 3 PDFs distincts avec 3 QR codes distincts.

---

### ⚙️ API Backend (FastAPI)

#### Endpoints organisateur (admin)
```
POST   /admin/events                        → créer un événement
GET    /admin/events                        → lister les événements
PUT    /admin/events/{id}                   → modifier un événement
GET    /admin/events/{id}/orders            → commandes d'un événement
GET    /admin/events/{id}/stats             → ventes par type de billet

POST   /admin/events/{id}/ticket-types     → créer un type de billet
PUT    /admin/ticket-types/{id}            → modifier un type de billet
```

#### Endpoints publics
```
GET    /events/{slug}                       → détail public d'un événement
GET    /events/{slug}/ticket-types         → types de billets disponibles
POST   /orders                             → créer une commande (achat)
GET    /orders/{id}                        → détail d'une commande
GET    /orders/{id}/tickets/{ticket_id}/pdf → télécharger un billet PDF
```

#### Endpoint de scan
```
POST   /scan
Body : { "qr_code": "<ticket_id>" }

Réponses possibles :
→ 200 { "status": "valid",    "ticket": { ... }, "holder": "...", "type": "VIP" }
→ 200 { "status": "already_scanned", "scanned_at": "2025-04-18T19:32:00Z" }
→ 200 { "status": "invalid" }
```

**Contrainte de performance :** cet endpoint doit répondre en **moins de 200ms**. Utilise Redis pour mettre en cache le statut des billets scannés — la base de données seule ne suffit pas.

---

### Règles métier obligatoires

- Un billet ne peut être vendu que si le quota du type n'est pas atteint — gérer la concurrence (pas de survente possible même avec des requêtes simultanées)
- Un QR code ne peut être scanné qu'une seule fois — toute tentative de double scan retourne `already_scanned`
- La vente est bloquée si `date_fin_vente` est dépassée
- Un billet annulé ne peut pas être scanné

---

## Fonctionnalités bonus _(non obligatoires, mais appréciées)_

- [ ] Auth JWT pour le dashboard organisateur
- [ ] Webhook sortant à l'achat (simule l'envoi du billet par message — email ou WhatsApp)
- [ ] Export CSV des participants d'un événement
- [ ] Page de validation alternative : saisie manuelle du numéro de commande (si QR illisible)
- [ ] Rate limiting sur `/scan` (protection anti-spam)
- [ ] Tests backend — notamment sur la logique de quota et le double scan (pytest)
- [ ] CI GitHub Actions

---

## Ce qu'on évalue

| Critère | Points |
|---|---|
| Fonctionnalités obligatoires livrées | 30 |
| Architecture et qualité du code (front + back) | 20 |
| Génération PDF + QR code correct et lisible | 15 |
| Performance et fiabilité du scan (Redis + concurrence) | 15 |
| UI/UX dashboard + page scan (optimisée mobile) | 10 |
| Bonus | +10 |

---

## Contraintes techniques

- **Frontend :** Next.js (App Router recommandé) + Tailwind CSS + TypeScript recommandé
- **Backend :** FastAPI + SQLAlchemy + Alembic + PostgreSQL
- **Cache scan :** Redis obligatoire pour l'endpoint `/scan`
- **PDF :** bibliothèque au choix (`reportlab`, `weasyprint`, `fpdf2`…) — mentionne ton choix dans ton README
- **QR code :** bibliothèque au choix (`qrcode`, `segno`…)
- **Docker Compose** obligatoire : `docker compose up` doit tout lancer (frontend + API + DB + Redis)
- Node.js 18+ / Python 3.11+

---

## Structure de repo attendue

```
eventpass/
├── frontend/
│   ├── app/
│   │   ├── admin/
│   │   │   ├── events/
│   │   │   └── scan/
│   │   └── events/
│   │       └── [slug]/
│   ├── components/
│   └── ...
├── backend/
│   ├── app/
│   │   ├── api/
│   │   │   ├── admin.py
│   │   │   ├── public.py
│   │   │   └── scan.py
│   │   ├── models/
│   │   ├── schemas/
│   │   ├── services/
│   │   │   ├── ticket_generator.py   # PDF + QR code
│   │   │   └── scan_service.py       # logique Redis
│   │   └── main.py
│   ├── alembic/
│   └── ...
├── docker-compose.yml
└── README.md
```

---

## Comment soumettre

1. **Fork** ce repository
2. Développe sur ta propre branche ou directement sur `main`
3. Assure-toi que `docker compose up` démarre tout sans erreur
4. Ton README doit inclure :
   - URL du dashboard admin
   - URL d'un événement exemple
   - Identifiants admin de test
   - Comment tester le scan (QR code d'exemple ou instructions)
5. Envoie le lien de ton repo GitHub à **careers@karaba.africa** avec en objet : `[STAGE FULLSTACK] Ton Nom`

> ⚠️ Un repo sans Docker Compose fonctionnel, sans génération de PDF opérationnelle, ou sans endpoint `/scan` sera automatiquement écarté.

---

## Questions ?

Ouvre une **GitHub Issue** sur ce repo. On répond sous 48h.
