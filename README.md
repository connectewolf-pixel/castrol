# Castrol Service Témara — refonte du site

Landing page statique, une seule page, **sans backend**. Aucune dépendance, aucun
build : on ouvre `index.html` et ça marche.

Démo prospect de la campagne d'acquisition Réponse Online.

> **Maquette, pas un site officiel.** Cette page est une proposition de refonte
> réalisée par [Réponse Online](https://reponseonline.com) à titre de démonstration.
> Elle n'est ni le site officiel de Castrol Service Témara, ni affiliée à Castrol
> ou à bp. Les marques citées appartiennent à leurs propriétaires respectifs.
> Le dépôt est publié comme pièce de portfolio — il n'est pas destiné à être servi
> en ligne à l'adresse du commerce.

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

## Le mur d'avis

Douze avis répartis en trois colonnes qui défilent en boucle à des vitesses
différentes (52 s, 64 s, 58 s), pour que l'ensemble ne se lise pas comme un
seul bloc qui glisse. Le script duplique chaque colonne au chargement : arrivée
à mi-course, la piste retrouve son point de départ et le raccord ne se voit
pas. La copie est marquée `aria-hidden` pour ne pas faire lire deux fois les
mêmes avis. Le `padding-bottom` des pistes vaut exactement leur `gap` — sans
lui, la boucle sauterait d'un demi-interstice à chaque tour.

Trois colonnes au-delà de 1080 px, deux à partir de 700 px, une seule en
dessous. Le défilement s'interrompt au survol, au focus et hors du champ de
vision ; en mouvement réduit, le mur devient une liste fixe que l'on parcourt
normalement. Il n'y a pas de bouton de pause : le survol et le focus suffisent
à retenir un avis le temps de le lire.

**Ce qui n'est pas affiché, volontairement** : aucune étoile par avis (la note
individuelle n'est pas connue, seule la moyenne l'est) et aucun portrait
(ce sont les initiales, comme Google le fait pour un compte sans photo —
mettre des visages d'une banque d'images serait un faux).

## Le défilement automatique des services

Le sélecteur avance d'un service toutes les 5 secondes. Le délai est la
constante `SVC_DELAY` dans le script, à changer aussi dans l'animation CSS
`svcFill` qui dessine la jauge — le seul indice visible que la bande tourne
toute seule, puisqu'il n'y a ni libellé ni bouton de pause.

Il s'interrompt dans quatre cas : au survol et au focus (reprise en sortant),
au clic ou à la flèche sur un panneau (le visiteur a choisi son service, la
bande ne repart plus), hors du champ de vision, et il ne démarre pas du tout
si le système est réglé en mouvement réduit — la jauge disparaît alors.

## Les filets de marque

Le vert et le rouge se partageaient une barre de 2 px en deux moitiés franches
— en tête des blocs (`.brand-stripe`) et sur la tranche du panneau de service
ouvert. Ça se lisait comme un drapeau collé sur la page, et aucune des deux
teintes n'y disait quoi que ce soit.

Un seul vert désormais, du clair au profond puis l'extinction : la lumière suit
le geste au lieu de le couper en deux. **Le rouge ne sert plus qu'à informer**
— le champ de formulaire en erreur. C'est la règle à tenir si un accent
revient : une couleur sur cette page doit dire quelque chose.

## Le menu

Le panneau ne glisse plus d'un bloc. Trois épaisseurs entrent l'une après
l'autre par la droite — un éclat vert, une couche carbone, puis le noir qui se
referme dessus (575 ms, décalage de 120 ms). Les liens montent ensuite de sous
leur ligne, inclinés de 7°, et se redressent en arrivant. À la fermeture, tout
repart dans l'ordre inverse : le noir d'abord, le vert en dernier.

Le déclencheur porte l'état : le mot bascule de « Menu » à « Fermer » derrière
une ligne masquée, et le signe `+` pivote de 315° pour devenir la croix. Le mot
reste affiché jusqu'à 320 px — seul, le `+` se lit comme « ajouter ».

Chaque entrée appelle une figure de fond (cercles, goutte, barres, diagonales,
trame) : une seule à la fois, au survol. **Sans curseur — donc sur téléphone —
aucune ne peut être appelée** : le menu en garde une posée en bas, à mi-voix,
plutôt que de laisser un vide sous les liens.

Tout est en CSS ; le JS ne fait que poser l'état, déplacer le focus vers la
croix à l'ouverture et le rendre au bouton à la fermeture. En mouvement réduit,
le rideau ne joue plus et les figures ne s'affichent pas.

Le menu reste réservé au **format ≤ 1024 px**. Au-dessus, la barre garde ses
liens, son numéro et son bouton de rendez-vous visibles : les cacher derrière un
geste coûterait des appels.

## La carte

La carte Google ne se charge plus au chargement de la page : **elle arrive au
clic**. Une visite qui ne va pas jusqu'à « Nous trouver » ne déclenche aucune
requête vers Google, aucun cookie tiers. Une fois créée, l'iframe reste — la
rouvrir ne recharge rien.

Au repos, le repère est dessiné sur place : papier millimétré, deux axes, trois
cercles de distance qui se tracent à l'arrivée dans le champ, et le point. Sous
le curseur, la carte s'incline (6° au maximum) et le point dérive légèrement ;
le mouvement se traîne derrière la souris au lieu de lui coller.

**Ce plan ne relève rien** : c'est un repère, pas une carte. Il le dit — « plan
indicatif — ouvrir la carte » — et la vraie carte est à un geste. Aucune rue
n'est nommée, aucun tracé ne prétend correspondre au terrain.

L'URL de la carte vit dans l'attribut `data-src` du bouton `#mapCard`, plus dans
un `<iframe>` du HTML.

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

## L'assistant — aperçu, pas encore branché

Le bouton en bas à **gauche** ouvre un panneau qui annonce l'assistant du garage.
**Rien ne répond derrière** : c'est une pièce de démonstration pour la campagne
d'acquisition, montrée telle quelle au prospect.

Elle est volontairement honnête — aucune fausse conversation, aucun faux « en
ligne ». Le statut dit « Bientôt disponible » en ambre (le vert dirait qu'il
répond), le panneau montre les questions qu'il saura traiter, et renvoie vers les
deux canaux qui répondent vraiment : WhatsApp et le téléphone.

**Le placement.** L'assistant tient le coin bas-gauche, WhatsApp garde le
bas-droit. Deux zones de pouce distinctes plutôt qu'une pile de bulles : sur un
écran de 320px, deux ronds empilés à droite occupaient 130px de haut et
recouvraient le contenu. Le lanceur est un rond de 56px au mobile, une pastille
avec le libellé « Assistant » à partir de 700px. Le panneau s'ouvre en **feuille
par le bas** au mobile (voile, verrou de défilement, poignée) et en **carte
ancrée au lanceur** au-dessus de 700px.

**Pour le brancher plus tard** : remplacer le contenu de `.assist-body` par le fil
de discussion. L'ossature — lanceur, feuille, voile, focus, Échap, verrou de
défilement — ne bouge pas. Le statut ambre « Bientôt disponible » est à retirer ce
jour-là ; le pied garde ses deux boutons, qui restent utiles même une fois
l'assistant en service.

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
      Google Maps : `hasMap`, `sameAs`, le `data-src` du bouton de la carte, le
      bouton « Itinéraire », le bouton « Lire les avis ». Sans ça, la carte peut
      pointer légèrement à côté.
- [ ] **Coordonnées GPS** (latitude/longitude) pour le champ `geo` du JSON-LD.
      Volontairement absent : de fausses coordonnées enverraient les clients
      au mauvais endroit.
- [ ] **Comptes Instagram / TikTok**, s'ils existent. Le pied de page n'a plus
      de rangée d'icônes du tout : celles d'Instagram et TikTok pointaient dans
      le vide, et celle de WhatsApp doublait le bouton flottant posé juste
      au-dessus d'elle. Les rétablir demande de recréer la rangée.
- [ ] **Le texte français d'origine des avis.** Ceux affichés viennent de la
      version anglaise que produit Google et ont été rendus en français. La
      plupart ont pourtant été écrits en français : le bouton « Voir l'original »
      de la fiche donne le texte exact, qui vaut mieux qu'une traduction de
      traduction. Les avis sont dans le HTML de la section `#avis`.

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
