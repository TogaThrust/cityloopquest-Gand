# Déploiement — CityLoop Quest Gand

Guide généré par **CLQ App Factory** pour **Gand** (`gand`).  
Dossier de sortie : `CLQ-GAND/clq-gand` — URL Netlify cible : **https://clq-gand.netlify.app**

---

## Vue d’ensemble

| Élément | Valeur pour cette ville |
|--------|-------------------------|
| Slug backend / DB | `gand` |
| Package npm | `clq-gand` |
| Site Netlify | `clq-gand` |
| Site ID Netlify | `ef87d0d6-03c2-4f9f-b4b5-2d5e9ea80da7` |
| Dépôt CI (GitHub → Netlify) | `TogaThrust/cityloopquest-Gand` |
| URL publique | https://clq-gand.netlify.app |
| API (Render) | https://cityloopquest-api.onrender.com |
| Région | Flandre-Orientale, Région flamande, Belgique |

**Ordre recommandé :** Factory (contenu) → Supabase + Stripe + Render → Google Maps → Netlify → tests → `npm run deploy`.

---

## 1. Génération depuis la Factory

1. Ouvrir **CLQ App Factory** (`npm start` → http://localhost:3847).
2. Charger ou compléter le JSON ville (**Gand**).
3. Lancer **Générer l’app** → sortie dans `CLQ-GAND/clq-gand`.
4. Vérifier dans la Factory : audios, images POI, quiz, traductions sans erreur bloquante.
5. Tester en local :
   ```bash
   cd "CLQ-GAND/clq-gand"
   npm install
   npm run dev
   ```
   Ouvrir http://localhost:5173 — en dev, le Service Worker est désactivé automatiquement.

> **API en local :** par défaut l’app pointe vers `https://cityloopquest-api.onrender.com`. Pour forcer une API locale :  
> `?api_base=http://localhost:8081` ou `setApiBase('http://localhost:8081')` en console.

---

## 2. GitHub

Deux dépôts possibles (recommandé pour les villes déjà en prod) :

| Rôle | Exemple | Usage |
|------|---------|--------|
| **Dev / Factory** | `clq-gand` (privé) | Travail quotidien, génération Factory |
| **CI Netlify** | `TogaThrust/cityloopquest-Gand` (public) | Push déclenche le build Netlify |

1. Pousser le code vers le dépôt dev (`origin`).
2. Ajouter le remote CI si besoin :
   ```bash
   git remote add cityloopquest https://github.com/TogaThrust/cityloopquest-Gand.git
   ```
3. Après chaque release : `npm run deploy:ci` (push `main` vers `cityloopquest`).
4. Vérifier `.gitignore` : `.env`, `dist/`, `node_modules/`, secrets — **pas** `.netlify/state.json` ni `netlify.site.json`.
5. Secrets GitHub (dépôt CI) pour le workflow `.github/workflows/netlify-deploy.yml` :
   - `NETLIFY_AUTH_TOKEN`, `NETLIFY_SITE_ID` (= `ef87d0d6-03c2-4f9f-b4b5-2d5e9ea80da7`)
   - `GOOGLE_MAPS_API_KEY` (build)

> **Lille / villes déjà liées dans Netlify :** le push sur `TogaThrust/cityloopquest-Gand` suffit souvent ; le workflow GitHub Actions sert de secours si l’app GitHub Netlify n’a pas accès au dépôt.

---

## 3. Netlify

### 3.1 Création du site

- **Build command :** `npm run build`
- **Publish directory :** `dist`
- **Functions :** `netlify/functions` (voir `netlify.toml`)
- **Node :** 18 (déjà dans `netlify.toml`)

### 3.2 Variables d’environnement (build)

À définir dans **Site settings → Environment variables** (pas la liste complète d’une autre ville — uniquement ce dont **Gand** a besoin) :

| Variable | Usage |
|----------|--------|
| `GOOGLE_MAPS_API_KEY` | Injectée dans `api-key.js` au build (`build.mjs`) |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS` | Formulaire contact / POI (`mail.html`, fonctions Netlify) |
| `POI_STAFF_EMAIL` | Destinataire des propositions POI |
| `POI_APPROVE_SECRET` | Secret d’approbation POI communautaires |
| `SITE_URL` | https://clq-gand.netlify.app |
| `NETLIFY_SITE_ID` + `NETLIFY_AUTH_TOKEN` | Blobs POI (ou `NETLIFY_BLOB_READ_WRITE_TOKEN`) |

Ne **pas** y mettre `STRIPE_SECRET_KEY` (reste sur Render).

### 3.3 Déploiement

**CI (recommandé après changements Git) :**
```bash
npm run deploy:ci
```

**Manuel (CLI Netlify, sans prompt) :**
```bash
cd "CLQ-GAND/clq-gand"
npm install
npm run deploy
```

(`npm run deploy` = build + deploy prod ; le Site ID est lu depuis `netlify.site.json` / `.netlify/state.json`.)

**Site ID :** `ef87d0d6-03c2-4f9f-b4b5-2d5e9ea80da7` — admin : https://app.netlify.com/projects/clq-gand

---

## 4. Google Cloud — clé Maps

Dans [Google Cloud Console](https://console.cloud.google.com/) → APIs & Services → Credentials → votre clé **Maps JavaScript API** :

**Référents HTTP autorisés :**

- `https://clq-gand.netlify.app/*`
- `http://localhost:5173/*` (dev Vite)
- `https://localhost:5173/*` (dev Vite HTTPS)

Activer : **Maps JavaScript API**, **Places API**, **Directions API** (ou Routes API selon migration future).

---

## 5. Render — API `cityloopquest-api`

Dashboard Render → service API → **Environment** :

> **Noms de variables :** pour un slug multi-mots (`gand`), utiliser des **underscores** dans le préfixe env (`GAND` = tirets du slug remplacés par `_`, ex. `caravaca-de-la-cruz` → `CARAVACA_DE_LA_CRUZ`). Ne pas utiliser de tirets ni un slug raccourci.

| Variable | Exemple / valeur |
|----------|------------------|
| `CLIENT_BASE_URL_GAND` | `https://clq-gand.netlify.app` |
| `STRIPE_GAND_FULL_7D_PRICE_ID` | `price_xxx` (depuis Stripe) |
| `STRIPE_GAND_LITE_7D_PRICE_ID` | si applicable |
| `STRIPE_GAND_UPGRADE_7D_PRICE_ID` | si applicable |

Mettre à jour **CORS / ALLOWED_ORIGINS** pour inclure `https://clq-gand.netlify.app`.

Dans le code API (si nouvelle ville) :

- Ajouter `gand` aux villes supportées (checkout, claim-session, activate-code).
- Mapper les Price IDs et `CLIENT_BASE_URL_GAND` dans la config (ex. `STRIPE_PRICE_IDS`, `FRONTEND_URLS`).

Redéployer l’API après modification.

---

## 6. Stripe

1. Créer les **produits / prix** 7 jours pour **Gand** (FULL, LITE, UPGRADE si utilisés).
2. Copier les **Price ID** (`price_…`) dans les variables Render ci-dessus.
3. **Webhook Stripe** → pointer vers **Render** (`https://cityloopquest-api.onrender.com/…`), pas vers Netlify.
4. Ne pas configurer d’URL success/cancel figées dans Stripe : elles sont construites côté API avec `CLIENT_BASE_URL_GAND`.

Flux utilisateur après paiement : `post-checkout.html` → `claim-session` sur Render → redirection `index.html?from=checkout`.

---

## 7. Supabase

Projet partagé (recommandé) :

1. Table **`cities`** : entrée `slug = 'gand'`, nom `Gand`, métadonnées région.
2. Table **`stripe_products_map`** (ou équivalent) : lier les Price ID Stripe à `gand`.
3. Table **`licenses`** : codes d’activation `xxx-xxx-xxx` rattachés à `city = 'gand'`.

Tester un code manuel sur `activation-manual.html` et via le lien email post-achat.

---

## 8. Garde licence par ville

Le fichier `js/license-city-guard.js` associe le hostname au slug `gand`.  
Après génération Factory, vérifier que la ligne contient bien :

```js
if (host.includes('gand')) return 'gand';
```

**Piège :** `localStorage` est partagé entre tous les sites `*.netlify.app` sur un même navigateur. Une licence **Murcia** ne doit pas débloquer **Gand** — le guard compare `clq_city` / JWT au slug du site.

---

## 9. Checklist avant mise en prod

- [ ] `npm run build` sans erreur ; `dist/` contient audio, images, traductions JSON **valides**.
- [ ] Parcours petit / moyen / grand : GPS, audio, quiz.
- [ ] Paiement test Stripe → email → activation lien **et** saisie manuelle du code.
- [ ] Bouton 🏠 (reset) → `parcours.html` sans redemander le code si accès valide.
- [ ] `mail.html` mentionne **Gand**.
- [ ] PWA : `manifest.json`, icônes `cityLoopQuest_Gand_*.png`.
- [ ] Popup fin de parcours + invitation selfie.
- [ ] CORS : appels API depuis https://clq-gand.netlify.app OK.

---

## 10. Dépannage fréquent

| Symptôme | Piste |
|----------|--------|
| `Failed to fetch` / API indisponible en local | Vider `localStorage.api_base` ou `useProdApi()` ; ne pas laisser `localhost:8081` si l’API locale n’est pas lancée. |
| Activation manuelle échoue, lien email OK | Même origine API ; vérifier `api-base.js` et CORS Render. |
| Service Worker / SSL en local | Sous Vite (`npm run dev`), le SW est désactivé. Sous `npm start` / preview HTTP, le SW s’enregistre pour tester l’install PWA. |
| Install PWA absente (barre d’adresse) | `index.html` doit charger `js/clq-sw-register.js` ; icônes `cityLoopQuest_Gand_{192,512}.png` présentes ; HTTPS ou localhost. |
| Licence d’une autre ville acceptée | `license-city-guard.js`, vider `localStorage` ou tester en navigation privée. |
| Images absentes / hotlink Wikimedia | La génération doit télécharger `directImageUrl`/`sourceUrl` vers `images/…`. Relancer la génération Factory ; vérifier les logs « Photos par lien » / « Relance téléchargement ». |
| Webhook 400 | Secret webhook Stripe = celui configuré sur Render. |

---

## 11. Mise à jour de contenu

1. Modifier le JSON dans la **Factory**.
2. Régénérer l’app (les images/audio existants sont préservés si déjà sur disque).
3. `npm run build && npm run deploy` dans `CLQ-GAND/clq-gand`.

---

*Document généré pour CityLoop Quest Gand — slug `gand`.*
