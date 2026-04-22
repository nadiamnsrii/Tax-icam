# 📱 Tax'icam — Guide de déploiement complet

## Architecture du projet

```
taxicam/
├── index.html      ← Application complète (frontend)
├── manifest.json   ← Config PWA (icône, nom, couleurs)
├── sw.js           ← Service Worker (offline, notifications)
└── icons/          ← Icônes à créer (voir ci-dessous)
    ├── icon-192.png
    └── icon-512.png
```

---

## 🚀 ÉTAPE 1 — Créer les icônes

Créez une image carrée 512×512px avec le logo Tax'icam (fond orange #E8660A, icône blanche).
Exportez en :
- `icons/icon-192.png` (192×192)
- `icons/icon-512.png` (512×512)

Outils gratuits : **Figma** (figma.com) ou **Canva**

---

## 🗄️ ÉTAPE 2 — Base de données Supabase (gratuit)

1. Créez un compte sur **supabase.com**
2. Nouveau projet → choisissez un nom (ex: taxicam)
3. Copiez votre `Project URL` et `anon key`

### Tables à créer (SQL Editor dans Supabase) :

```sql
-- Table utilisateurs
create table profiles (
  id uuid references auth.users primary key,
  name text not null,
  phone text,
  promo text,
  is_sam boolean default false,
  rating decimal default 5.0,
  trips_count int default 0,
  created_at timestamp default now()
);

-- Table trajets
create table rides (
  id serial primary key,
  type text check (type in ('sam','event','daily','request')),
  from_addr text not null,
  to_addr text not null,
  from_lat decimal, from_lng decimal,
  to_lat decimal, to_lng decimal,
  departure_time timestamp not null,
  places int default 3,
  taken int default 0,
  price decimal default 0,
  zero_alcohol boolean default false,
  note text,
  driver_id uuid references profiles(id),
  created_at timestamp default now()
);

-- Table réservations
create table bookings (
  id serial primary key,
  ride_id int references rides(id),
  passenger_id uuid references profiles(id),
  status text default 'pending' check (status in ('pending','accepted','declined')),
  created_at timestamp default now()
);

-- Table messages
create table messages (
  id serial primary key,
  from_id uuid references profiles(id),
  to_id uuid references profiles(id),
  ride_id int references rides(id),
  content text not null,
  read boolean default false,
  created_at timestamp default now()
);
```

### Authentification email ICAM :
Dans Supabase → Authentication → Providers → Email :
- Activez "Confirm email"
- Dans les paramètres avancés, ajoutez cette validation custom :

```sql
-- Fonction de validation email ICAM
create or replace function validate_icam_email()
returns trigger as $$
begin
  if NEW.email not like '%@icam.fr'
     and NEW.email not like '%@%.icam.fr' then
    raise exception 'Seules les adresses @icam.fr sont autorisées';
  end if;
  return NEW;
end;
$$ language plpgsql;

create trigger check_icam_email
  before insert on auth.users
  for each row execute function validate_icam_email();
```

---

## 🌐 ÉTAPE 3 — Déploiement sur Vercel (gratuit)

### Option A — GitHub + Vercel (recommandé)

1. Créez un repo GitHub (github.com)
2. Uploadez vos fichiers (index.html, manifest.json, sw.js, icons/)
3. Allez sur **vercel.com** → New Project → importez votre repo
4. Déployez en 1 clic → Vercel vous donne une URL https://taxicam.vercel.app

### Option B — Vercel CLI

```bash
# Installer Vercel CLI
npm install -g vercel

# Dans votre dossier taxicam/
vercel

# Suivez les instructions → URL générée automatiquement
```

### Domaine personnalisé (optionnel, ~10€/an)
Dans Vercel → Settings → Domains → ajoutez `taxicam.fr` ou `taxicam-vannes.fr`

---

## 📲 ÉTAPE 4 — Installation sur iPhone et Android

### Android (Chrome) — automatique
1. Ouvrez **Chrome** sur votre téléphone
2. Allez sur votre URL (ex: taxicam.vercel.app)
3. Chrome affiche automatiquement **"Ajouter à l'écran d'accueil"**
4. Confirmez → Tax'icam apparaît comme une vraie app !

### iPhone (Safari) — 3 étapes
1. Ouvrez **Safari** (obligatoire, pas Chrome)
2. Allez sur votre URL
3. Appuyez sur l'icône **Partager** (carré avec flèche) en bas
4. Faites défiler → **"Sur l'écran d'accueil"**
5. Confirmez → Tax'icam apparaît sur votre home screen !

> ⚠️ Sur iPhone, utiliser Safari est obligatoire pour l'installation PWA.

---

## 🔔 ÉTAPE 5 — Notifications push (optionnel)

Utilisez **Firebase Cloud Messaging** (gratuit) :

1. Créez un projet sur **firebase.google.com**
2. Project Settings → Cloud Messaging → copiez votre `Server Key`
3. Dans index.html, ajoutez avant `</body>` :

```html
<script type="module">
import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js';
import { getMessaging, getToken } from 'https://www.gstatic.com/firebasejs/10.0.0/firebase-messaging.js';

const app = initializeApp({
  apiKey: "VOTRE_API_KEY",
  projectId: "VOTRE_PROJECT_ID",
  messagingSenderId: "VOTRE_SENDER_ID",
  appId: "VOTRE_APP_ID"
});

const messaging = getMessaging(app);
// Demande la permission et obtient le token
Notification.requestPermission().then(permission => {
  if (permission === 'granted') {
    getToken(messaging, { vapidKey: 'VOTRE_VAPID_KEY' }).then(token => {
      // Envoyez ce token à votre backend pour envoyer des notifs
      console.log('Token:', token);
    });
  }
});
</script>
```

---

## 💰 Coûts

| Service | Coût |
|---------|------|
| Supabase (BDD + Auth) | **Gratuit** jusqu'à 500MB |
| Vercel (hébergement) | **Gratuit** illimité |
| Firebase (notifications) | **Gratuit** jusqu'à 1M/mois |
| Domaine .fr personnalisé | ~10€/an (optionnel) |
| **Total MVP** | **0€** |

---

## 📋 Checklist de lancement

- [ ] Icônes créées (192px et 512px)
- [ ] Projet Supabase créé + tables SQL exécutées
- [ ] Validation email @icam.fr activée
- [ ] Fichiers uploadés sur GitHub
- [ ] Déployé sur Vercel (URL HTTPS obtenue)
- [ ] Testé sur Android (Chrome)
- [ ] Testé sur iPhone (Safari)
- [ ] App installée sur votre téléphone ✓
- [ ] Partagé aux membres du BDE

---

## 🛠️ Prochaines étapes techniques

1. **Connecter Supabase** : remplacer les données statiques par de vraies requêtes
2. **Auth réelle** : brancher le login/signup sur Supabase Auth
3. **Temps réel** : Supabase Realtime pour les nouveaux trajets en live
4. **Messagerie** : stocker les messages dans la table `messages`

---

*Tax'icam — Développé pour le BDE ICAM Vannes*
