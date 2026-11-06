# Index VD+ LSR

**Page en ligne : https://TheAlpineGrinch.github.io/VD-Index-LSR/**

Outil pour former des groupes de niveau homogènes sur les sorties LSR.

Chaque rider charge son historique Strava. Ses sorties vélo sont placées sur un graphique vitesse / pente moyenne, avec les zones de couleur du club (noir, rouge, bleu, vert, blanc). Il voit quelle couleur choisir selon le terrain, et obtient son VD+.

## Zones de couleur

Vitesse sur le plat : Noir 35 à 39, Rouge 31 à 35, Bleu 27 à 31, Vert 23 à 27, Blanc 19 à 23 km/h.

Elles décroissent avec la pente :

V(pente) = V(0) / (1 + 15 × pente), pente en décimal (3,5 % = 0,035).

## Terrains

| Terrain | Pente moyenne |
|---|---|
| Plat | moins de 0,5 % |
| Vallonné | 0,5 à 1,5 % |
| Escarpé | 1,5 à 2,5 % |
| Abrupte | 2,5 à 3,5 % |

Seules les sorties de l'année en cours et de l'année précédente sont prises en compte. Sur le graphique, les points sont violets pour les 3 derniers mois, bleus pour le reste de l'année en cours et verts pour l'année précédente.

Les sorties de plus de 3,5 % de pente moyenne sont écartées, trop spécifiques pour situer un niveau. Les trajets marqués « Commute » sont exclus par défaut.

## Ce que la page calcule pour un rider

- **Profil** : régression polynomiale d'ordre 2 de sa vitesse en fonction de la pente. Elle donne sa couleur sur chaque terrain. Un profil n'est pas forcément aligné avec les zones.
- **VD+** : vitesse à 0 % de pente, ordonnée à l'origine de la régression linéaire. Il est affiché avec son écart type (erreur standard de l'ordonnée à l'origine, ± 1σ), par exemple 32,5 ± 1,3.
- **Coefficient VD+** : le xx équivalent, c'est-à-dire le coefficient de décroissance qui ajuste le mieux ses sorties sur le modèle des zones, à comparer au 15 des zones. Il vient d'une régression linéaire de 1/V en fonction de la pente (en %) : xx = 100 × pente de la droite / ordonnée à l'origine.
- **Type de rider**, affiché en grand et en bleu sous le titre du bloc « Ton profil VD+ », selon le coefficient VD+ noté xxR et le xx des zones noté XX (15) :
  - Rouleur si xxR > XX × 1,2 (plus de 18)
  - Baroudeur si XX × 0,8 ≤ xxR ≤ XX × 1,2 (de 12 à 18)
  - Grimpeur si xxR < XX × 0,8 (moins de 12)

Un bouton copie ces chiffres pour les envoyer. Rassemblés sur plusieurs riders, les couples VD+ / coefficient VD+ servent à ajuster le xx des zones, voire à utiliser un xx différent selon le VD+.

## Utilisation

1. Sur strava.com › Paramètres › Mon compte › **Télécharger ou supprimer votre compte** › **Demander une archive**.
2. Strava envoie un lien par e-mail. Décompresser l'archive.
3. Déposer `activities.csv` sur la page. Le fichier est lu dans le navigateur et n'est jamais envoyé. Les rides sont gardés dans le stockage local du navigateur (`localStorage`), sur l'appareil seulement, pour ne pas avoir à recharger le fichier à chaque visite. Le bouton « Effacer mes rides » les supprime. Le chargement n'est possible qu'après avoir coché la case d'accord (voir « Collecte des statistiques »).

## Statut

Prototype. Le xx = 15 des zones reste à calibrer avec les données de plusieurs riders.

Page unique, sans build : ouvrir `index.html` ou passer par GitHub Pages.

## Collecte des statistiques (Supabase)

Avec l'accord explicite du rider (case à cocher, obligatoire avant le chargement du fichier), la page envoie à une base Supabase uniquement :

- son VD+ et l'écart type du VD+ ;
- son coefficient VD+ ;
- un identifiant aléatoire créé dans son navigateur, pour éviter les doublons d'un même utilisateur, et un numéro de version.

Ces données servent uniquement à ajuster les zones de couleur de LSR. Elles sont pseudonymisées par l'identifiant aléatoire. Le fichier, les rides et le prénom ne sont jamais envoyés. L'identifiant est affiché sur la page. Les demandes de suppression de données sont à faire auprès de Lyon Social Rides.

La clé publique (`sb_publishable_…`) présente dans `index.html` est publique par conception. La sécurité repose sur la RLS : la table n'accepte que des insertions, ni lecture ni modification depuis le navigateur. La clé secrète ne doit jamais être mise dans le dépôt.

Table à créer dans l'éditeur SQL de Supabase :

```sql
create table public.vdplus_stats (
  id bigint generated always as identity primary key,
  created_at timestamptz not null default now(),
  client_id uuid not null,
  vd_plus numeric(4,1) not null check (vd_plus between 10 and 60),
  vd_plus_se numeric(4,1) check (vd_plus_se between 0 and 20),
  coef_vd numeric(5,1) not null check (coef_vd between -20 and 80),
  version text
);
alter table public.vdplus_stats enable row level security;
revoke all on public.vdplus_stats from anon, authenticated;
grant insert on public.vdplus_stats to anon;
create policy "insertion anonyme" on public.vdplus_stats
  for insert to anon with check (true);
```

Lecture des statistiques (une ligne par rider, la plus récente) :

```sql
select distinct on (client_id) * from vdplus_stats
order by client_id, created_at desc;
```
