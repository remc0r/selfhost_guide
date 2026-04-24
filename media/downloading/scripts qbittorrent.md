Une fois [qbittorent & gluetun - docker](qbittorent%20&%20gluetun%20-%20docker.md) installé, on peut choisir d'éxécuter un script après chaque téléchargement de torrent; dans mon cas cela va me servir à déplacer les fichiers fraîchement téléchargés vers mon NAS

Dans `Tools` -> `Options` -> `Downloads`, on retrouve l'option `Run on torrent finished`, on voit également quel paramètre on peut prendre en compte dans notre script

![](../../__images/Pasted%20image%2020260423220055.png)

Supported parameters (case sensitive):

- %N: Torrent name
- %L: Category
- %G: Tags (separated by comma)
- %F: Content path (same as root path for multifile torrent)
- %R: Root path (first torrent subdirectory path)
- %D: Save path
- %C: Number of files
- %Z: Torrent size (bytes)
- %T: Current tracker
- %I: Info hash v1
- %J: Info hash v2
- %K: Torrent ID

Tip: Encapsulate parameter with quotation marks to avoid text being cut off at whitespace (e.g., "%N")

Voici un exemple de script qui permets de déplacer des fichiers après téléchargements.

```bash
#!/bin/bash

TORRENT_NAME="$1"
CONTENT_PATH="$4"
SAVE_PATH="$6"

LOG_FILE="/downloads/logs/qbittorrent-script.log"

# Destination selon le chemin de sauvegarde
if [[ "$SAVE_PATH" == *"/downloads/films"* ]]; then
    DESTINATION_PATH="/media/movies"
elif [[ "$SAVE_PATH" == *"/downloads/series"* ]]; then
    DESTINATION_PATH="/media/series"
elif [[ "$SAVE_PATH" == *"/downloads/music"* ]]; then
    DESTINATION_PATH="/media/music"
elif [[ "$SAVE_PATH" == *"/downloads/books"* ]]; then
    DESTINATION_PATH="/media/books"
else
    DESTINATION_PATH="/media/other"
fi

mkdir -p "$DESTINATION_PATH" 2>/dev/null
mkdir -p "$(dirname "$LOG_FILE")" 2>/dev/null

# Déplacement simple
if mv "$CONTENT_PATH" "$DESTINATION_PATH/"; then
    echo "$(date) - ✓ $TORRENT_NAME → $DESTINATION_PATH" >> "$LOG_FILE" 2>/dev/null
    # Nettoyer les dossiers vides
    find "$SAVE_PATH" -type d -empty -delete 2>/dev/null
else
    echo "$(date) - ✗ ERREUR: $TORRENT_NAME" >> "$LOG_FILE" 2>/dev/null
    exit 1
fi

```