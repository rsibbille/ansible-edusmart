# Démonstration technique EduSmart

Cette procédure privilégie des contrôles en lecture seule. Ne pas provoquer une panne de nœud juste avant la soutenance.

## 1. Validation Ansible non destructive

Depuis `srv-prd-par-ans-001` :

```bash
cd ~/ansible-edusmart
ansible-playbook -i inventories/production/hosts.yml playbooks/demo-validation.yml
```

Résultat attendu :

- tous les hôtes répondent ;
- `sshd -t` est valide ;
- `ssh.service` est actif ;
- le récapitulatif affiche `changed=0`, `unreachable=0` et `failed=0`.

Pour une démonstration plus courte :

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/demo-validation.yml --limit srv-prd-par-ans-001
```

## 2. État du cluster Proxmox

À exécuter dans le shell d'un nœud Proxmox :

```bash
pvecm status
pvecm nodes
pvesm status
ha-manager status
ha-manager config
pvesh get /cluster/resources --type vm
```

Points à commenter :

- trois nœuds présents et quorum obtenu ;
- stockage NFS OpenMediaVault accessible ;
- ressources HA et état des services ;
- répartition actuelle des VM/LXC entre les nœuds.

## 3. Démonstration de mobilité sans panne

Utiliser uniquement une VM ou un conteneur de test, jamais OPNsense, le NAS ou un service critique pendant la soutenance.

Dans l'interface Proxmox :

1. sélectionner la ressource de test ;
2. ouvrir **Migrate** ;
3. choisir un autre nœud ;
4. conserver la migration en ligne lorsque l'option est disponible ;
5. surveiller la tâche puis montrer que la ressource reste joignable.

Pendant la migration, conserver un terminal avec :

```bash
watch -n 2 'pvesh get /cluster/resources --type vm'
```

Et un second terminal avec un test applicatif ou réseau adapté à la ressource :

```bash
ping -c 1 ADRESSE_IP_DE_TEST
```

## 4. Message à donner au jury

La migration contrôlée démontre la mobilité des charges. La haute disponibilité ajoute la reprise automatique d'une ressource déclarée HA lorsqu'un nœud devient indisponible, à condition que son stockage et ses dépendances restent accessibles depuis les autres nœuds.
