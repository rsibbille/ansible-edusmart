# Rôle `hardening_anssi_robust`

Ce rôle applique un **profil Debian 12/13 inspiré du niveau renforcé** du guide
ANSSI-BP-028 v2.0. Il ne constitue pas une certification de conformité ANSSI :
les recommandations dépendent du rôle de chaque serveur et certaines ne peuvent
être réalisées qu'à l'installation (UEFI/Secure Boot, partitionnement, options de
montage) ou par le noyau fourni/compilé.

## Principes de sécurité

- déploiement séquentiel (`serial: 1`) ;
- aucune mise à niveau globale, purge, autoremove ou reboot automatique ;
- aucune activation automatique d'UFW/nftables ;
- aucun changement global de `ip_forward`, IPv6, `rp_filter`, user namespaces ou
  chargement des modules sans acquittement explicite ;
- audit des fichiers world-writable/setuid au lieu d'une correction récursive ;
- AppArmor en enforce uniquement pour une liste de profils testés ;
- auditd sans capture de tous les `execve` ni verrouillage `-e 2` par défaut ;
- sudo compatible avec Ansible (`requiretty`, `noexec` et I/O logging désactivés).

## Mesures à fort risque

Toute mesure disruptive requiert :

```yaml
anssi_acknowledge_disruptive_changes: true
```

Cette validation ne rend pas la mesure sûre. Elle confirme seulement qu'une revue
par rôle et un test canari ont été réalisés.

### Risques connus pour EduSmart

- `net.ipv4.ip_forward=0` : casse Docker, les routeurs et certains VPN ;
- `rp_filter=1` : peut casser routage asymétrique, multi-homing, VPN et conteneurs ;
- désactivation IPv6 : peut casser résolution, bind, services ou conteneurs ;
- `kernel.modules_disabled=1` : irréversible jusqu'au reboot et peut empêcher
  NFS, nftables, overlay/Docker ou un pilote de fonctionner ;
- `Defaults requiretty/noexec` : casse Ansible et de nombreuses automatisations ;
- sudo I/O logging : peut enregistrer des secrets et saturer le disque ;
- audit global `execve` : très volumineux ;
- AppArmor global `aa-enforce /etc/apparmor.d/*` : dangereux et incorrect ;
- firewall générique « SSH seulement » : couperait LDAP/DNS, SMTP, Teleport,
  monitoring, NFS et les publications Docker.

## Déploiement recommandé

Audit et simulation sur le contrôleur Ansible :

```bash
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/hardening-linux.yml \
  --limit srv-prd-par-ans-001 \
  --check --diff
```

Application sur la machine canari :

```bash
ansible-playbook -i inventories/production/hosts.yml \
  playbooks/hardening-linux.yml \
  --limit srv-prd-par-ans-001
```

Puis validation SSH, DNS, NTP, LDAP/SSSD, SMTP, Docker, Teleport, Prometheus et
montages NFS avant d'élargir groupe par groupe.

## Recommandations ANSSI non automatisées ici

- matériel, BIOS/UEFI, Secure Boot et clés de démarrage ;
- protection de GRUB/initramfs et Unified Kernel Image ;
- IOMMU sur l'hôte physique ;
- choix des options de compilation du noyau ;
- repartitionnement et options de montage `noexec/nosuid/nodev/hidepid` ;
- durcissement applicatif spécifique de BIND9, OpenLDAP, Postfix/Dovecot,
  Docker, Teleport, NFS et des services web ;
- centralisation distante et protection hors hôte de la base AIDE.
