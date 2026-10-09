# Méthodologie

## Démarche adoptée

Plutôt que de choisir une CVE à l'avance, la démarche a suivi les phases classiques d'un test d'intrusion (PTES), en laissant la reconnaissance orienter la recherche de vulnérabilités :

1. **Reconnaissance** — scan de ports, identification de service et de version
2. **Scanning / énumération** — applications exposées, services actifs
3. **Analyse des vulnérabilités** — recherche croisée NVD / CISA KEV / Exploit-DB
4. **Exploitation contrôlée** — validation (`check`) avant exploitation effective
5. **Post-exploitation** — tentative d'élévation de privilèges
6. **Reporting** — preuves techniques et recommandations

## Reconnaissance détaillée

```bash
nmap -sV -sC -p- 192.168.82.20
curl -I http://192.168.82.20:8080
curl http://192.168.82.20:8080/docs/RELEASE-NOTES.txt
```

Règle appliquée : **ne jamais se fier à une seule source** pour confirmer une version. Une CVE recherchée sur une version erronée invalide toute la démarche qui en découle — la bannière HTTP a donc toujours été recoupée avec une seconde source (fichier de version, page d'erreur).

## Triage des CVE candidates

Chaque CVE potentiellement applicable a été évaluée sur quatre critères avant sélection :

| Critère | Pourquoi |
|---|---|
| Plage de versions affectées | Une correspondance textuelle du nom ne suffit pas |
| Score CVSS | Priorisation de l'impact |
| Présence au catalogue CISA KEV | Preuve d'exploitation confirmée en conditions réelles |
| Correspondance avec la reconnaissance | Le vecteur doit être réellement exposé sur la cible |

**Exemple de CVE écartée** : `CVE-2024-50379` (Tomcat 9.0.0.M1) nécessitait un filesystem insensible à la casse — condition non remplie (`ext4` confirmé via `df -T`). L'investigation a été arrêtée dès l'échec de cette condition cumulative, sans tester les suivantes.

## Principe directeur

Ne jamais fabriquer artificiellement une condition d'exploitation (compte ajouté pour l'occasion, mot de passe connu à l'avance) lorsque celle-ci n'est pas une faille réellement découverte sur le système — y compris lorsque ce choix conduit à un résultat négatif ou partiel (voir `findings/CVE-2020-1938-ghostcat.md`, section chaînage RCE).

## Cadre MITRE ATT&CK

| Tactique | Technique | Application |
|---|---|---|
| Reconnaissance (TA0043) | T1595.002 — Active Scanning | Scan de ports et services |
| Accès initial (TA0001) | T1190 — Exploit Public-Facing Application | Exploitation des 3 CVE |
| Exécution (TA0002) | — | RCE via dépôt de fichier / injection OGNL |
