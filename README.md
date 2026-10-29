# Index VD+ LSR

**Page en ligne : https://TheAlpineGrinch.github.io/VD-Index-LSR/**

Outil pour former des groupes de niveau homogènes sur les sorties LSR.

Chaque rider charge son historique Strava. Ses sorties vélo sont placées sur un graphique vitesse / pente moyenne, avec les zones de couleur du club (gris, rouge, bleu, vert, blanc). Il voit quelle couleur choisir selon le terrain, et obtient son VD+.

## Zones de couleur

Vitesse sur le plat : Gris 35 à 39, Rouge 31 à 35, Bleu 27 à 31, Vert 23 à 27, Blanc 19 à 23 km/h.

Elles décroissent avec la pente :

V(pente) = V(0) / (1 + 15 × pente), pente en décimal (3,5 % = 0,035).

## Terrains

| Terrain | Pente moyenne |
|---|---|
| Plat | moins de 0,5 % |
| Vallonné | 0,5 à 1,5 % |
| Escarpé | 1,5 à 2,5 % |
| Abrupte | 2,5 à 3,5 % |

Les sorties de plus de 3,5 % de pente moyenne sont écartées, trop spécifiques pour situer un niveau. Les trajets marqués « Commute » sont exclus par défaut.

## Ce que la page calcule pour un rider

- **Profil** : régression polynomiale d'ordre 2 de sa vitesse en fonction de la pente. Elle donne sa couleur sur chaque terrain. Un profil n'est pas forcément aligné avec les zones.
- **VD+** : vitesse à 0 % de pente, ordonnée à l'origine de la régression linéaire.
- **Coefficient VD+** : pente de cette droite, en km/h perdus par point de pente.
- **xx équivalent** : le coefficient qui ajuste le mieux ses sorties sur le modèle des zones, à comparer au 15 des zones.

Un bouton copie ces chiffres pour les envoyer. Rassemblés sur plusieurs riders, les couples VD+ / coefficient VD+ servent à ajuster le xx des zones, voire à utiliser un xx différent selon le VD+.

## Utilisation

1. Sur strava.com › Paramètres › Mon compte › **Télécharger ou supprimer votre compte** › **Demander une archive**.
2. Strava envoie un lien par e-mail. Décompresser l'archive.
3. Déposer `activities.csv` sur la page. Le fichier est lu dans le navigateur : rien n'est envoyé ni enregistré.

## Statut

Prototype. Le xx = 15 des zones reste à calibrer avec les données de plusieurs riders.

Page unique, sans build : ouvrir `index.html` ou passer par GitHub Pages.
