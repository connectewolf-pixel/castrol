# Castrol Service Témara — refonte du site

Landing page statique, une seule page, **sans backend**. Aucune dépendance, aucun
build : on ouvre `index.html` et ça marche.

Démo prospect de la campagne d'acquisition Réponse Online.

---

## Lancer en local

```bash
cd ~/Desktop/castrol-redesign
python3 -m http.server 8080
# → http://localhost:8080
```

Ouvrir le fichier en `file://` marche aussi, mais la carte Google et les vidéos
se comportent mieux derrière un serveur.

## Structure

```
castrol-redesign/
├── index.html      ← toute la page : HTML + CSS + JS inline
├── 404.html        ← page d'erreur, autonome
├── robots.txt
├── sitemap.xml
└── images/
    ├── hero-video.mp4 / hero-poster.webp|jpg
    ├── service-video.mp4 / service-poster.webp|jpg
    ├── produit-castrol.webp    ← section « Le seul centre certifié »
    ├── logo-png-castrol.png
    ├── icon-officiel-castrol-service-temara.webp
    └── services/               ← une photo par service, nommée par le service
        vidange · freinage · diagnostic · climatisation
        boite-auto · pneus · embrayage · filtres
```

Les WebP servent de posters vidéo (plus légers), les JPG restent pour l'aperçu
des partages sociaux — WhatsApp et Facebook ignorent le WebP.

Les fichiers sources lourds et les images inutilisées du template d'origine sont
conservés **hors du dossier de déploiement**, dans
`~/Desktop/castrol-redesign-assets-source/` (29 Mo de vidéos 4K + 25 images).
Ne rien remettre ici sans raison : ce dossier doit rester léger.

---

## Les photos des services

Chaque panneau du sélecteur porte sa photo, dans `images/services/`. Le nom du
fichier dit le service : pour en remplacer une, écraser le fichier du même nom,
il n'y a rien à changer dans le HTML.

Les originaux (jusqu'à 4,7 Mo pièce) sont conservés dans
`~/Desktop/castrol-redesign-assets-source/originaux-services/`. Les versions
servies sont réencodées en WebP à 1400 px maximum — 9 photos pour 504 Ko au
total. Pour en réintégrer une nouvelle :

```bash
ffmpeg -i source.jpg -vf "scale='min(1400,iw)':-2:flags=lanczos" \
       -c:v libwebp -quality 76 images/services/<service>.webp
```

Fermée, la photo est désaturée et assombrie pour que le nom du service reste
lisible par-dessus ; ouverte, elle reprend ses couleurs. Le dégradé sous le
texte est calé sur les photos les plus claires (freinage, diagnostic) : en
changer une pour un cliché très lumineux demande de revérifier la lisibilité.

## Le défilement automatique

Le sélecteur avance d'un service toutes les 5 secondes. Le délai est la
constante `SVC_DELAY` dans le script, à changer aussi dans l'animation CSS
`svcFill` qui dessine la jauge.

Il s'interrompt dans quatre cas : au survol et au focus (reprise en sortant),
au clic sur un panneau ou sur le bouton « Suspendre » (reprise seulement sur
demande), hors du champ de vision, et il ne démarre pas du tout si le système
est réglé en mouvement réduit — la jauge et le bouton disparaissent alors.

## Le formulaire de rendez-vous

Il n'envoie rien à un serveur. Il assemble un message et ouvre WhatsApp avec le
texte déjà rédigé — le visiteur relit et envoie lui-même. Aucune donnée n'est
stockée, aucun RGPD à gérer, rien à héberger.

**Pour brancher un vrai envoi plus tard** (si le garage le demande) : dans
`index.html`, le handler `bookingForm.addEventListener('submit', …)` en bas de
page. Remplacer la construction de l'URL `wa.me` par un `fetch()` vers
l'endpoint choisi. Le reste de la page ne bouge pas.

Le numéro est dans la constante `GARAGE_WHATSAPP` — un seul endroit à changer.

---

## ⚠️ À obtenir du propriétaire avant mise en ligne

- [ ] **Horaires exacts** — la page affiche « Ouvert 6 jours sur 7 » en attendant.
      À corriger à **4 endroits** : la bande de stats, la ligne « Horaires » de
      « Nous trouver », la FAQ, et le bloc `openingHoursSpecification` (déjà
      rédigé en commentaire dans le `<head>`, il n'y a qu'à le décommenter).
- [ ] **Deux photos de service en meilleure définition** : `freinage` (764 px
      de large) et `filtres` (518 px) sont les seules sources sous 1000 px.
      Elles s'affichent correctement mais manquent de netteté sur grand écran.
      Toutes les autres sont en 1200–1400 px.
- [ ] **Photos du garage lui-même** (atelier, équipe, vitrine), si l'on veut
      remplacer les photos d'illustration des services par des prises de vue
      du lieu. La photo produit de la section « À propos », elle, a bien été
      faite sur place.
- [ ] **URL exacte de la fiche Google Business** (bouton « Partager » → lien
      court). À substituer partout où figure aujourd'hui une URL de *recherche*
      Google Maps : `hasMap`, `sameAs`, la carte, le bouton « Itinéraire », le
      bouton « Lire les avis ». Sans ça, la carte peut pointer légèrement à côté.
- [ ] **Coordonnées GPS** (latitude/longitude) pour le champ `geo` du JSON-LD.
      Volontairement absent : de fausses coordonnées enverraient les clients
      au mauvais endroit.
- [ ] **Comptes Instagram / TikTok**, s'ils existent. Les icônes ont été retirées
      parce qu'elles pointaient dans le vide.
- [ ] **3 avis clients réels** à copier depuis la fiche Google (nom affiché +
      texte exact). Le gabarit est prêt en commentaire dans la section Avis.
      Ne jamais reformuler un avis : on le cite mot pour mot ou on ne le met pas.

## ⚠️ À vérifier le jour du déploiement

La note **4,8/5 et les 817 avis** sont affichés dans la page *et* déclarés dans
le JSON-LD. Ces chiffres bougent. Google exige qu'ils correspondent à la réalité
et qu'ils soient visibles sur la page — les deux endroits doivent être mis à jour
ensemble.

---

## Le trou SEO — à trancher avant de remplacer le site actuel

Le site en ligne aujourd'hui n'est pas une page unique : il compte une quinzaine
de pages qui se positionnent sur des requêtes locales précises.

| Page actuelle | Requête visée |
|---|---|
| `vidange-rabat.html` | vidange voiture Rabat |
| `entretien-rabat.html` | entretien automobile Rabat |
| `garage-automobile-rabat.html` | choisir un garage à Rabat |
| `climatisation-recharge-gaz-temara.html` | recharge clim Témara |
| `reparation-boite-de-vitesses-temara.html` | boîte de vitesses Témara |
| `vente-pneus-temara.html` | pneus Témara |
| `diagnostic-electronique-maintenance-temara.html` | diagnostic électronique Témara |

**Mettre cette page unique à la place du site actuel ferait disparaître ce
trafic.** Trois options, à arbitrer avec le propriétaire :

1. **Reconstruire les pages locales** au nouveau design (le plus de travail, le
   meilleur résultat). La landing devient l'accueil, chaque service garde sa page.
2. **Rediriger en 301** chaque ancienne URL vers l'ancre correspondante de la
   nouvelle page. Rapide, conserve une partie de l'autorité, mais fait perdre le
   positionnement des pages détaillées.
3. **Déployer la landing sur une autre URL** et garder le site actuel en ligne.
   Zéro risque, mais deux sites à maintenir.

C'est aussi la première objection qu'un prospect (ou son référenceur) posera en
démo — mieux vaut arriver avec la réponse.

---

## Déploiement

Hébergement statique, n'importe lequel : Railway, Netlify, Cloudflare Pages.
Aucune variable d'environnement, aucun secret.

Avant de publier, remplacer le domaine `castrol-service-temara.com` s'il diffère,
à 4 endroits : `<link rel="canonical">`, `og:url`, `og:image`, `robots.txt`, et
`sitemap.xml`.
