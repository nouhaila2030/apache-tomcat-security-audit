# Recommandations de durcissement

## Par vulnérabilité

### Ghostcat (CVE-2020-1938)
- Mettre à jour Tomcat vers une version corrigée (7.0.100+, 8.5.51+, 9.0.31+)
- À défaut, désactiver le connecteur AJP s'il n'est pas utilisé, ou le restreindre à `127.0.0.1` avec `secretRequired="true"`

### CVE-2017-12615 (WebDAV/PUT)
- Désactiver la méthode HTTP `PUT` sur le Default Servlet (`readonly=true`)
- Interdire le dépôt de fichiers exécutables (`.jsp`) via une configuration stricte du serveur

### CVE-2017-5638 (Struts2)
- Mettre à jour Struts2 vers une version corrigée (2.3.32 ou 2.5.10.1+), qui échappe correctement les en-têtes avant évaluation OGNL

## Durcissement générique de Tomcat

| Mesure | Effet observé dans ce lab |
|---|---|
| Compte de service dédié, sans `sudo` | A contenu l'impact des deux RCE au seul périmètre applicatif |
| Suppression des apps par défaut (`/examples`, `/docs`, `/host-manager`) | Réduit la surface d'attaque |
| Authentification forte + restriction IP sur le Manager App | Empêche le chaînage Ghostcat → RCE via upload WAR |
| Mise à jour régulière (Tomcat + frameworks applicatifs) | Corrige les 3 CVE démontrées |
| Masquage des bannières de version | Complique la reconnaissance |

## Intégration recommandée avec un SOC

La corrélation d'événements (Wazuh/Suricata) permettrait de détecter :
- des requêtes AJP anormales,
- des méthodes HTTP `PUT` inhabituelles,
- des en-têtes `Content-Type` contenant des motifs OGNL suspects.

*(Piste documentée mais non mise en œuvre dans ce projet — voir le dépôt [soc-lab-wazuh-suricata-thehive](../soc-lab-wazuh-suricata-thehive) pour la plateforme de détection correspondante.)*
