# Notice pédagogique — Atelier écoconception sur « Mes démarches en ligne »

Ce document est réservé au formateur (ou peut être distribué en fin d'atelier
comme corrigé). Il décrit les choix volontairement non optimisés intégrés
au site, organisés pour servir de support à une démarche d'écoconception
en 4 temps : **ACV simplifiée → référentiel de bonnes pratiques → plan
d'action et stratégie de mesure → optimisations prioritaires**.

---

## 1. Le site et ses unités fonctionnelles (UF)

Pour une ACV simplifiée d'un service numérique, l'unité fonctionnelle doit
décrire un **service rendu**, mesurable, et non une simple page. Trois UF
candidates, correspondant aux trois services :

| UF | Définition proposée | Parcours (écrans) |
|---|---|---|
| **UF1** | Obtenir l'envoi d'une demande de carte nationale d'identité, pièces justificatives jointes, pour un utilisateur connecté | Accueil → Identité → État civil → Pièces (upload) → Récapitulatif → Confirmation (6 pages) |
| **UF2** | Obtenir l'envoi d'une demande d'inscription sur les listes électorales avec justificatif conforme, pour un utilisateur connecté | Accueil → Identité → Modèles de pièces → Téléversement → Récapitulatif → Confirmation (6 pages) |
| **UF3** | Obtenir l'envoi d'une demande d'acte de naissance, pour un utilisateur connecté | Accueil → Bénéficiaire → Type de document → Récapitulatif → Confirmation (5 pages) |

Faire choisir/discuter cette définition par les apprenants est volontaire :
une UF mal cadrée (ex. « afficher une page ») fausse la suite. Faire émerger
qu'il faut inclure la page d'accueil partagée par les 3 UF (charge mutualisée
à répartir), et que la mascotte flottante et le bandeau partenaires/actus
sont présents sur **toutes** les pages du parcours, donc comptent dans
**chaque** UF (poids invisible mais réel).

## 2. Inventaire des flux par UF (pour le périmètre de l'ACV)

À faire establir par les apprenants à l'aide des outils navigateur
(onglet Réseau), pas seulement en lisant le code :
- nombre de pages vues sur le parcours complet,
- poids transféré par page (HTML, CSS, JS, images, polices) et poids cumulé sur l'UF,
- nombre de requêtes HTTP par page (dont requêtes tierces : Google Fonts),
- volumétrie des données ressaisies/transmises (formulaires),
- fréquence d'usage estimée du service (qui pondère l'impact total annuel — à faire débattre : un service rare mais très lourd vs un service fréquent mais léger).

## 3. Anti-patterns volontairement intégrés (grille de repérage)

### a) Transverses (présents sur toutes les pages)

| # | Anti-pattern | Où le trouver | Bonne pratique associée (référentiel RGESN-like) |
|---|---|---|---|
| T1 | 4 familles de polices chargées via Google Fonts (`Archivo Expanded`, `Archivo`, `Source Sans 3`, `Caveat` — cette dernière non utilisée à l'écran) avec plusieurs graisses chacune | `css/style.css`, `@import` en première ligne | Limiter le nombre de polices et de graisses ; utiliser les polices système quand c'est suffisant ; ne charger que les graisses réellement utilisées |
| T2 | Mascotte flottante en `position:fixed`, animée en boucle infinie, présente sur 100 % des pages, sans valeur d'usage | `.mascotte-flottante`, toutes les pages | Supprimer les éléments strictement décoratifs ; si conservé, ne pas l'animer en continu (consommation CPU/GPU permanente) |
| T3 | Compteur de visiteurs « temps réel » : `setInterval` toutes les 2 s qui force un reflow, sans utilité pour l'utilisateur | `js/main.js`, fonction `compteurVisiteursDecoratif` | Supprimer les widgets purement décoratifs qui maintiennent la page active inutilement (impact batterie/CPU) |
| T4 | Carrousel de logos partenaires en autoplay CSS infini, sur la page d'accueil | `.bandeau-partenaires`, `index.html` | Préférer un défilement déclenché par l'utilisateur (boutons), ou une grille statique |
| T5 | Pas de sauvegarde de progression entre les écrans d'un même formulaire (perte de toute la saisie en cas de retour arrière navigateur ou de fermeture d'onglet) | Tous les parcours, `js/main.js` (état uniquement en mémoire DOM) | Persister l'état du formulaire (stockage local, brouillon serveur) pour éviter la ressaisie et l'abandon |
| T6 | Simulation d'attente artificielle avant soumission (`simulerVerificationServeur`, 3 × 600 ms sans nécessité) | `js/main.js` | Ne pas introduire de latence artificielle ; informer sans bloquer |
| T7 | Accessibilité non traitée : contraste non vérifié sur certains éléments, mention « Accessibilité : non conforme » dans le pied de page, focus visible minimal | Pied de page, ensemble du site | Une démarche d'accessibilité réduit aussi le risque de re-développement et les parcours d'erreur (lien écoconception/accessibilité) |

### b) UF1 — Demande de carte d'identité (upload)

| # | Anti-pattern | Où le trouver | Bonne pratique associée |
|---|---|---|---|
| U1-1 | Zone d'upload acceptant des formats très larges (JPG, PNG, PDF, HEIC, TIFF) **sans limite de taille indiquée**, et message incitant à envoyer « la résolution d'origine » | `pages/carte-identite/etape-1.html`, écran 3 | Définir un format et un poids cible (ex. JPEG ≤ 2 Mo), redimensionner/compresser côté client avant envoi |
| U1-2 | Aucune compression ni redimensionnement côté client avant l'aperçu/l'envoi : l'image est lue en pleine résolution (`FileReader.readAsDataURL`) pour générer une simple miniature de 54×54 px | `js/main.js`, fonction `initZoneUpload` | Générer la miniature à partir d'une version réduite (canvas) plutôt que de manipuler le fichier complet ; compresser avant transfert réseau |
| U1-3 | Pas de prévisualisation guidée ni de critères de conformité affichés avant l'upload → risque fort de rejet et de ressaisie complète du parcours | écran 3 du parcours | Donner des critères clairs avant l'action d'upload (taille de fichier, cadrage) pour limiter les essais-erreurs, donc les requêtes réseau redondantes |

### c) UF2 — Inscription sur les listes électorales (exemples de pièces)

| # | Anti-pattern | Où le trouver | Bonne pratique associée |
|---|---|---|---|
| U2-1 | Images d'exemple en haute résolution native (1400×900 px, JPEG qualité 95) alors qu'affichées dans des cartes de quelques centaines de pixels de large | `img/exemple-cni-*.jpg`, galerie de l'écran 2 | Servir des images redimensionnées/recadrées à la taille d'affichage réelle (et `srcset` pour le responsive) |
| U2-2 | Les **4** images d'exemple (conforme + 3 non conformes) sont chargées **deux fois** : une fois dans le carrousel, une fois dans la galerie détaillée, sur le même écran | écran 2 du parcours | Réutiliser un seul jeu d'images, ou ne montrer qu'un des deux dispositifs (carrousel ou galerie, pas les deux) |
| U2-3 | Carrousel en autoplay (changement toutes les 3,5 s) qui tourne même si l'utilisateur ne regarde pas l'écran, sans bouton pause | `js/main.js`, fonction `initCarrouselExemples` | Autoplay déclenché/arrêtable par l'utilisateur ; couper l'intervalle quand l'onglet n'est pas visible (`visibilitychange`) |
| U2-4 | Toutes les images d'exemple sont chargées dès l'arrivée sur l'écran, qu'elles soient vues ou non (pas de lazy-loading, pas de `loading="lazy"`) | écran 2 du parcours | Ajouter `loading="lazy"` et/ou charger les variantes uniquement au moment où elles entrent dans le viewport |

### d) UF3 — Demande d'acte de naissance (formulaire « simple »)

| # | Anti-pattern | Où le trouver | Bonne pratique associée |
|---|---|---|---|
| U3-1 | Ressaisie demandée : les noms des parents sont demandés à l'écran 1 **puis reconfirmés** à l'écran 2 (champs dupliqués sans pré-remplissage) | `pages/acte-naissance/etape-1.html`, écrans 1 et 2 | Pré-remplir automatiquement un champ déjà saisi plutôt que de le redemander ; limiter le nombre d'écrans et de champs au strict nécessaire |
| U3-2 | Aucune validation de format en amont des champs (date, nom) avant le passage à l'écran suivant, ce qui repousse la détection d'erreur jusqu'au récapitulatif final | parcours complet | Valider au plus tôt (au niveau du champ) pour éviter des allers-retours et relances serveur supplémentaires |

## 4. Trame d'atelier proposée (séquencement)

1. **Cadrage / ACV simplifiée** (1/2 journée)
   - Faire définir les 3 UF par les apprenants (cf. §1), puis comparer à la proposition ci-dessus.
   - Faire inventorier, pour chaque UF, le nombre de pages, le poids par page et les requêtes (onglet réseau du navigateur), en isolant la part « commune » (header/footer/mascotte/polices) de la part « propre » à chaque service.
   - Faire formuler des indicateurs simples : poids total transféré par parcours complété, nombre de requêtes par parcours, volume de données ressaisies.

2. **Référentiel de bonnes pratiques** (1/2 journée)
   - À partir des constats, faire construire une grille (s'inspirer du RGESN — Référentiel Général d'Écoconception de Services Numériques) avec des critères classés par thème : contenus, images, polices, formulaires, scripts/animations, accessibilité.
   - Confronter cette grille à la liste d'anti-patterns ci-dessus, sans la leur donner directement : les faire (re)découvrir par l'audit.

3. **Plan d'action et stratégie de mesure** (1/2 journée)
   - Faire prioriser les anti-patterns trouvés (matrice effort/impact).
   - Faire définir, pour 2 ou 3 actions, un indicateur avant/après mesurable (ex. poids moyen de page sur le parcours UF2, nombre de requêtes images, nombre d'écrans de saisie sur UF3).
   - Outils suggérés : onglet Réseau / Lighthouse / EcoIndex / GreenIT-Analysis — à choisir selon le matériel disponible.

4. **Mise en œuvre des optimisations prioritaires** (1/2 à 1 journée)
   - Faire implémenter 2 à 4 correctifs réels sur une copie du projet, par exemple :
     - retirer la mascotte et le compteur décoratif (T2, T3) ;
     - redimensionner/compresser les images d'exemple et ne garder qu'un seul dispositif d'affichage (U2-1, U2-2) ;
     - ajouter `loading="lazy"` sur les images non critiques (U2-4) ;
     - pré-remplir les champs dupliqués de l'écran 2 du service acte de naissance (U3-1) ;
     - réduire à une ou deux familles de polices, ne charger que les graisses utilisées (T1).
   - Refaire la mesure définie à l'étape 3 pour objectiver le gain.

## 5. Remarque sur le réalisme des anti-patterns

Volontairement, aucun anti-pattern n'est caricatural au point d'être visible
au premier coup d'œil (pas de vidéo 4K en autoplay, pas de méga-bibliothèque
JS inutile) : l'objectif est de reproduire des choix que l'on retrouve
couramment dans des services réels — bons sur la forme, mais coûteux en
ressources une fois mesurés. Cela force les apprenants à mesurer avant de
juger, plutôt qu'à repérer « ce qui se voit ».
