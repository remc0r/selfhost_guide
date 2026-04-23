l'outil [beets](https://github.com/beetbox/beets) est un autotagger qui permets de ranger, renommer sa musique et télécharger des metadatas.
Le tout connecté a des APIs de classification musicale comme [MusicBrainz](https://musicbrainz.org/) ou des APIs de récupération de lyrics.

## Installation
On peut l'installer facilement avec le gestionnaire de paquet `pip` ou `pipx` selon les environnements.

```bash
pip install beets
```

## Configuration 

Une fois installé, on peut récupérer le chemin vers le fichier de configuration avec la commande suivante :

```bash
beet config -p
```

on va ensuite venir éditer ce fichier pour personnaliser `beets` à notre usage :

```yml
directory: /mnt/ssd/media/music/sorted
library: /home/remcor/.config/beets/musiclibrary.db

import:
  move: yes
  write: yes
  autotag: yes
  quiet_fallback: asis
  ignore: sorted

plugins: musicbrainz fetchart embedart scrub lyrics chroma filetote spotify

lyrics:
  auto: yes
  sources: [lrclib]
  synced: yes
paths:
  default: $albumartist/$album%aunique{}/$track $title
  singleton: Non-Album/$artist/$title
  comp: Compilations/$album%aunique{}/$track $title

fetchart:
  auto: yes
  minwidth: 500
  sources: filesystem coverart itunes amazon albumart fanarttv

asciify_paths: no

filetote:
  extensions: .lrc
  pairing:
    enabled: true
    extensions: ".lrc"

```

*Ceci est mon fichier de configuration et réponds à mes usages, n'hésitez pas à le personnaliser*

## Plugins

La force de `beets` réside dans les nombreux plugins qu'on peut lui injecter.

Voici une liste non exhaustive de mes plugins les plus utiles :

### musicbrainz

Connexion à la bibliothèque en ligne `musicbrainz` qui vient reconnaître la plupart des sorties musicales

### filetote

Permets d'associer les `.lrc` existants lors du téléchargements (fichier de paroles en mode karaoké) aux différentes musiques.

### chroma

Chroma permets de faire la reconnaissance musicale en fonction de l'empreinte de la musique plutôt que de sa metadata.

Je n'ai pas pu le tester en action à proprement parlé mais je pense que c'est un bon plugins à avoir en *fallback* aux différentes APIs musicales

Il nécessite une installation avec : 

```bash
pipx inject beets pyacoustid
sudo apt install chromaprint
sudo apt install libchromaprint-tools
sudo apt install ffmpeg
```

## Utilisation 

Pour utiliser `beets` je recommande une architecture de ce type 

```
└── music\
    ├── inbox\
    └── sorted\
```

```bash
beet import music/inbox
```

L'outil nous proposera ensuite de valider ses choix depuis la ligne de commande 

Voici un extrait de la documentation officielle de beets avec les choix possibles lors de l'import
## Choices

When beets needs your input about a match, it says something like this:

Tagging:
    Beirut - Lon Gisland
(Similarity: 94.4%)
* Scenic World (Second Version) -> Scenic World
[A]pply, More candidates, Skip, Use as-is, as Tracks, Enter search, enter Id, or aBort?

When beets asks you this question, it wants you to enter one of the capital letters: A, M, S, U, T, G, E, I or B. That is, you can choose one of the following:

- _A_: Apply the suggested changes shown and move on.
    
- _M_: Show more options. (See the Candidates section, below.)
    
- _S_: Skip this album entirely and move on to the next one.
    
- _U_: Import the album without changing any tags. This is a good option for albums that aren’t in the MusicBrainz database, like your friend’s operatic faux-goth solo record that’s only on two CD-Rs in the universe.
    
- _T_: Import the directory as _singleton_ tracks, not as an album. Choose this if the tracks don’t form a real release—you just have one or more loner tracks that aren’t a full album. This will temporarily flip the tagger into _singleton_ mode, which attempts to match each track individually.
    
- _G_: Group tracks in this directory by _album artist_ and _album_ and import groups as albums. If the album artist for a track is not set then the artist is used to group that track. For each group importing proceeds as for directories. This is helpful if a directory contains multiple albums.
    
- _E_: Enter an artist and album to use as a search in the database. Use this option if beets hasn’t found any good options because the album is mistagged or untagged.
    
- _I_: Enter a metadata backend ID to use as search in the database. Use this option to specify a backend entity (for example, a MusicBrainz release or recording) directly, by pasting its ID or the full URL. You can also specify several IDs by separating them by a space.
    
- _B_: Cancel this import task altogether. No further albums will be tagged; beets shuts down immediately. The next time you attempt to import the same directory, though, beets will ask you if you want to resume tagging where you left off.
    

Note that the option with `[B]rackets` is the default—so if you want to apply the changes, you can just hit return without entering anything.


