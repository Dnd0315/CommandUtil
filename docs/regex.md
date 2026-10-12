# Regex

[← Index](../README.md)

À utiliser avec `grep` / `sed` / `awk` (voir [Texte](texte.md)).

## Regex de base

- `^` → début de ligne : '^root' = lignes qui commencent par root
- `$` → fin de ligne : 'bash$' = lignes qui finissent par bash
- `.` → n'importe quel caractère unique : 'c.t' = cat, cut, c9t…
- `*` → élément précédent 0 fois ou plus : 'lo*l' = ll, lol, lool…
- `\` → rend un caractère spécial littéral : '\.' = un vrai point
- `[a-z] [0-9] [abz954]` → un caractère parmi la plage ou la liste
- `[^0-9]` → un caractère qui n'est PAS dans la liste

## Regex étendues

Nécessitent `grep -E` (en regex basique il faut les échapper : `\+ \? \| \{n\}`) :

- `+` → élément précédent 1 fois ou plus : '[0-9]+' = au moins un chiffre
- `?` → élément précédent optionnel : 'colou?r' = color ou colour
- `x|y` → l'un ou l'autre : 'error|fail'
- `{n}` exactement n fois / `{n,}` au moins n / `{,n}` au plus n / `{n,m}` entre n et m : '[0-9]{3}' = 3 chiffres
- `(...)` → groupe : '(ab)+' = ab, abab…
