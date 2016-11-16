---
layout: page
title: Hardwave
permalink: /caracteristiques/hard/
---


Le fonctionnement du hardware des Neo-Geo CD 
Par Furrtek

Toutes les versions de la NeoGeo (AES, MVS, CD) sont basées sur un processeur Motorola 68000 (68K), et un Zilog Z80 comme processeur auxiliaire, s'occupant uniquement du son.

Elles possèdent toutes un BIOS, un programme qui fournit des fonctions courament utilisées par les jeux afin de simplifier certaines tâches relatives au hardware (lecture CD, gestion des cartes mémoire...). 
La synthèse sonore sur les versions cartouche est assurée par un YM2610 couplé à un convertisseur numérique/analogique YM3016 (qui ont été ensuite physiquement regroupés dans la même puce).
La NeoGeo CD possède un peu plus de 7Mo de DRAM pour charger des fichiers depuis le CD, ils sont répartis comme suit: .
2Mo de programme 68K, 
4Mo de sprites, 
1Mo de sons, 
128ko de graphismes pour les textes
64ko de programme Z80.
Elle est capable de lire des pistes audio CDDA en parallèle avec le son du YM2610.
La NeoGeo est capable de faire le rendu d'une image de 320 par 224 pixels, avec 4096 couleurs affichables simultanément (sans tricher) sur une palette de 65536. La plupart des jeux n'utilisent que 304 pixels en largeur, laissant une marge de 8 pixels noirs à gauche et à droite.
Le rendu se fait d'une manière assez spéciale:
Il y a un plan fixe (souvent appellé "Fix layer") qui peut comporter des pixels transparents et qui recouvre tout l'écran (prioritaire).
Il utilise des tiles de 8x8 pixels, qui viennent du ROM S (ou des fichiers .FIX sur NeoGeo CD). Il sert généralement à afficher du texte comme les copyrights, les scores, les barres de vie...

Tout le reste est considéré comme des sprites.
Les fonds des jeux habituellement placés dans des couches spéciales (backgrounds), sont ici formés par une suite de sprites combinés.
Les sprites sont constitués de tiles de 16x16 pixels qui viennent des ROMS C ou des fichiers .SPR sur NeoGeo CD.



Une règle essentielle à retenir concernant les couleurs: il n'y a jamais plus de 15 couleurs par tile (que ce soit sur le fix, ou dans les sprites), la 16ème couleur sert de transparence. Les sprites et le fix sont parametrés dans la RAM vidéo (VRAM), elle est accessible qu'à travers des registres gérés par le GPU et elle est toujours adressée en mots de 16 bits. La VRAM ne contient pas les graphismes, uniquement la position et les attributs des sprites et du fix. Les graphismes demeurent soit dans la cartouche, soit dans la DRAM sur NeoGeo CD.

Il n'y a pas de framebuffer: ce n'est pas possible d'accédèr à la sortie vidéo pixel-par-pixel. Le GPU ne possède que deux "buffers de ligne" internes qui sont chargés et vidés alternativement durant l'affichage.

Les sprites sont controllés par 4 zones dans la VRAM, appelées Sprite Control Banks (SCB).
Le fix n'a qu'une simple map.


Une règle essentielle à retenir concernant les couleurs: jamais plus de 16 couleurs par tile (que ce soit sur le fix, ou dans les sprites). Tous les graphismes ont leurs couleurs codées sur 4 bits. Les sprites et le fix sont parametrés dans la RAM vidéo (VRAM). Elle est complètement indépendante de la RAM du 68K. Elle est accessible qu'à travers des registres et elle est toujours adressé en words (16 bits).

Le framebuffer n'est pas directement accessible: ce n'est pas possible d'accédèr à la sortie vidéo pixel-par-pixel sur les versions cartouche.
C'est peut être possible sur NeoGeo CD, mais ça serait probablement très lent et donc inutile.

Les sprites sont controllés par 4 zones dans la VRAM, appelées Sprite Control Banks, ou SCB.
Le fix n'a qu'une simple map.


Les sprites sont constitués d'un ou plusieurs tiles de 16x16 pixels, provenant des ROMs C (ou des fichiers .SPR sur NeoGeo CD).
Chaque sprite est en réalité une map (un tableau à une colonne) , et son affichés sous forme de bandes verticales de tiles, allant de 1x1 tiles à 1x32 tiles (soit 16x16 à 16x512 pixels).
La largeur des sprites est toujours fixée à 1 tile, seule leur hauteur peut être modifiée. La NeoGeo est capable de faire le rendu de 96 sprites par ligne horizontale au maximum, et 384 sur tout l'écran.
Par exemple, bien qu'étant un seul et même objet tennant dans un rectangle de 4x6 tiles, cet object ne peut pas être constitué que d'un seul sprite:



Chaque sprite étant limité à un tile de largeur, il faut en mettre plusieurs côte-à-côte pour créer des objets plus larges. Si les 4 sprites étaient séparés de quelques pixels, cet objet ressemblerait à ceci.



Quand plusieurs sprites doivent être groupés, ils doivent avoir des numéros successifs (d'où la notation en n+x...). Chacun des 32 tiles pouvant constituer un sprite est modifiable, ainsi que sa palette, son orientation (effets mirroir) et ses caractèristiques d'auto-animation. Les attributs d'un sprite sont: sa position (X/Y), sa hauteur en tiles, sa map, s'il est pilotant ou piloté, et ses coefficients de rétrécissement (X/Y).
Un sprite est positionné à partir de son coin haut gauche (origine). D'un point de vue performances, la NeoGeo ne fait pas la différence entre un sprite de 1x1 tile et un sprite de 1x32 tiles. La taille (hauteur) du sprite doit être définie. Chaque tile constituant le sprite possède des paramètres qui lui sont propres (effet de mirroir horizontal, vertical, numéro de palette à utiliser...). Notez bien que les palettes sont individuellement attribuées aux tiles constituant le sprite, et non au sprite globalement.
Un sprite peut très bien contenir 16 tiles, utilisant chacun une palette différente (soit 16*16 = 256 couleurs en tout dans le sprite).



Une fonction très pratique permet de "grouper" les sprites, pour palier à la limite d'un seul tile en largeur.
Un bit à 1 dans la configuration du sprite permet de le lier au précédent. Sa position sera alors automatiquement calculée (X+16 pixels, Y identique), il aura la même hauteur et le même rétrécisseur Y, ceci est expliqué en détail après...
Par exemple:
Ici le sprite A est dit "pilotant", le sprite B y est lié (flèche blanche), il est donc de la même hauteur, et est collé 16 pixels à sa droite. Le sprite C est lié au sprite B et ainsi de suite, dans une certaine limite (32 ? A vérifier).
C'est à dire que par exemple, on peut très bien coller 15 sprites de 28 tiles de haut côte-à-côte pour controler un bloc de 240x448 pixels.

Notez également qu'un sprite reste un sprite. Ce n'est pas parce qu'il est lié à un autre qu'il ne compte plus comme tel.





 

Mélange des différentes couches, pour montrer à quoi ressemblent les sprites en bandes dans un vrai jeu:

On peut voir comment son constitués les objets en désactivant les sprites un par un dans les "screenshot factory" de certains émulateurs, comme WinKawaks. On peut aussi voir qu'ils servent aussi à constituer les fonds.

Remarque: Il est fréquent de voir des graphismes qui ne sont jamais affichés à l'écran.
Par exemple, on ne voit jamais le bas des trois tourelles dans cette partie de Metal Slug (une dizaine de pixels).
Quand on a la possibilité de remplir 64Mo de ROM dans une cartouche, on est plus à 1Ko près...



Sur NeoGeo CD, le chargement des données initiales est renseigné par un fichier texte (IPL.TXT), pris en charge par le BIOS dès le démarrage de la console.
La NeoGeo CD utilise un gestionnaire de périphériques (Sanyo LC8953) très peu documenté. Il gère le DMA (traitements de masse sur la mémoire) et les interruptions.
Contrairement aux versions cartouche, le 68K peut ici virtuellement écrire dans toutes les mémoires (graphismes, son, programme et RAM Z80) par le biais de fonctions fournies par le BIOS.

Le code, les graphismes et les sons du BIOS sont physiquement contenus dans le même ROM de 512ko.
Les différentes données sont chargées dans les mémoires appropriées au démarrage de la console.

Le BIOS possède les mêmes fonctions que les versions cartouche, mais avec celles concernant le chargement de fichiers et le DMA en plus.



Schéma initialement réalisé par Progfr, avec quelques modifications.


Les palettes servent à définir quelles seront les couleurs affichables dans un tile. Elle sont constituées de 16 entrées, la première (zéro) étant la transparence.
Une palette est attribuée à un ou plusieurs tiles, ils ne peuvent donc contenir que les 15 couleurs de celle-ci.

Les palettes ne sont pas stockées en VRAM, mais dans une RAM séparé accessible directement depuis le 68K.
Il y a 256 palettes de 16 entrées (15 couleurs).
Chaque couleur est définie sur 16bits, avec 5 bits pour chaque couleur primaire et un bit commun à toutes (LSB inversé).

Ce format simplifie beaucoup l'écriture en hexadécimal.
Le premier nibble (Frvb) peut être constitué de tête et importe peu sur la couleur, les trois suivants sont les composantes rouge, verte, et bleue.
Le bit 15 sert à rendre la couleur légèrement plus foncée.

 

Site Furrtek 
Wiki Furrtek (mise à jour)


