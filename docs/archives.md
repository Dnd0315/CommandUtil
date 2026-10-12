# Archives

[← Index](../README.md)

Options tar : c créer, x extraire, t lister, r ajouter, f fichier archive (toujours en dernier, suivi du nom), v verbeux

## tar

- `tar -tf archive.tar` → liste le contenu sans extraire
- `tar -cf archive.tar file1 dir/` → crée une archive (non compressée)
- `tar -rf archive.tar file2` → ajoute un fichier à une archive existante (pas possible si compressée)
- `tar -xf archive.tar -C /tmp/` → extrait dans /tmp (sans -C : dans le dossier courant)
- `tar -czf a.tar.gz dir/` → archive + compression gzip (z)
- `tar -cjf a.tar.bz2 dir/` → compression bzip2 (j)
- `tar -cJf a.tar.xz dir/` → compression xz (J majuscule)
- `tar -xf a.tar.gz` → extraction : tar détecte la compression tout seul
- `tar -I 'gzip -9' -cf a.tar.gz -C dossier .` → crée une archive gzip avec le niveau de compression choisi (-I = programme de compression) ; -C dossier . = archive le CONTENU du dossier sans le chemin

## gzip

- `gzip file` / `gunzip file.gz` → compresse / décompresse UN fichier (⚠️ remplace l'original : file devient file.gz)
- `gzip -k file` → compresse en GARDANT l'original (-k = keep : on obtient file ET file.gz) ; marche aussi avec `gunzip -k`, `bzip2 -k`, `xz -k`
- `gzip --best file` = `gzip -9 file` → meilleure compression (la plus lente)
- `gzip -l a.tar.gz` → taille compressée / non compressée et taux de compression

## bzip2, xz et lecture sans décompresser

- `bzip2 file` / `bunzip2 file.bz2` → compresse / décompresse en bzip2 ; `xz file` / `unxz file.xz` → idem en xz (mêmes options -k et -9 que gzip)
- `zcat file.gz` → affiche le contenu d'un fichier gzip sans le décompresser ; `zless file.gz` → le lit page par page ; `zgrep 'motif' file.gz` → cherche dedans (pratique pour les logs tournés). Équivalents `bzcat` et `xzcat`

## tar : compléments

- `tar -xf a.tar.gz chemin/dans/archive` → extrait UN seul fichier ou dossier (chemin tel que l'affiche `tar -tf`)
- `tar --exclude='*.log' -czf a.tar.gz dir/` → archive en excluant des fichiers

## zip

- `zip -r archive.zip dir/` → crée une archive zip (`sudo apt install zip unzip`)
- `unzip archive.zip -d /tmp/` → extrait dans /tmp ; `unzip -l archive.zip` → liste le contenu

## Synchroniser et sauvegarder (rsync)

- `rsync -av src/ dest/` → copie / synchronise (-a archive : récursif + droits + dates + liens ; -v verbeux). Ne copie que ce qui a changé
- ⚠️ Slash final : `src/` = le CONTENU de src copié dans dest ; `src` (sans slash) = le dossier src lui-même copié DANS dest
- `rsync -avn src/ dest/` → simulation (-n / --dry-run) : montre ce qui serait copié, sans rien faire
- `rsync -av --delete src/ dest/` → miroir exact : supprime dans dest ce qui n'existe plus dans src (à tester d'abord avec -n)
- `rsync -av --exclude='*.log' src/ dest/` → exclut des fichiers
- `rsync -avz src/ john@host:/backup/` → vers une autre machine par SSH, avec compression (-z) ; `rsync -av -e 'ssh -p 2222' src/ john@host:/backup/` → autre port SSH
