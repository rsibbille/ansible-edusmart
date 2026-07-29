# EduSmart — automatisation Ansible

Dépôt d'automatisation de la maquette EduSmart : serveurs Debian 12/13, OpenLDAP/BIND9, messagerie, relais SMTP, Teleport, Docker, supervision et durcissement Linux.

## Principes

- inventaire de production structuré par rôle ;
- comptes et accès SSH gérés par Ansible ;
- déploiements progressifs et idempotents ;
- durcissement inspiré du niveau renforcé ANSSI, avec garde-fous pour Docker, Teleport, LDAP, SMTP, NFS et VPN ;
- secrets stockés hors du dépôt ou chiffrés avec Ansible Vault.

## Validation rapide

```bash
ansible-inventory -i inventories/production/hosts.yml --graph
ansible-playbook -i inventories/production/hosts.yml playbooks/demo-validation.yml
```

Le playbook de démonstration est non destructif et doit produire `changed=0` lorsque l'infrastructure est stable.

## Durcissement Linux

Test canari :

```bash
ansible-playbook -i inventories/production/hosts.yml playbooks/hardening-linux.yml \
  --limit srv-prd-par-ans-001 \
  --check --diff \
  -e 'anssi_disable_ipv6=true anssi_acknowledge_disruptive_changes=true'
```

Les mesures susceptibles de perturber le routage, Docker, les VPN, le chargement des modules, sudo ou AppArmor sont désactivées par défaut et nécessitent une validation explicite.

## Démonstration

La procédure de soutenance est disponible dans [`docs/demo-soutenance.md`](docs/demo-soutenance.md).

## Terraform

Le code Terraform de création des VM Proxmox est conservé dans un dépôt IaC séparé afin de distinguer le provisionnement de l'infrastructure de sa configuration Ansible. Les fichiers d'état, variables sensibles, plans, clés et jetons ne doivent jamais être publiés.
