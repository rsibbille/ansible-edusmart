# Accès SSH nominatifs avec Teleport

## État validé le 29 juillet 2026

Les comptes Linux nominatifs sont présents sur les serveurs EduSmart, membres du
groupe `eds_admins` et disposent de leurs clés SSH.

Les utilisateurs Teleport conservent leur authentification locale et leur MFA.
Teleport délivre ensuite un certificat SSH temporaire correspondant aux logins
Linux autorisés.

## Politique appliquée

| Identité Teleport | Logins Linux autorisés |
|---|---|
| `r.sibbille` | `r.sibbille`, `eds_admin` |
| `s.mousset` | `s.mousset` |
| `p.feret` | `p.feret` |
| `n.sokry` | `n.sokri` |

L'identité Teleport `n.sokry` conserve temporairement son ancienne orthographe
afin de ne pas imposer un nouvel enrôlement MFA à Nesly. Le compte Linux utilisé
est correctement nommé `n.sokri`.

Le login partagé `eds_admin` reste exceptionnel et accessible uniquement à
Ruben. Les autres membres utilisent exclusivement leur compte nominatif.

## Persistance

Ces paramètres sont enregistrés dans la base interne de Teleport située sous
`/var/lib/teleport`.

Les rôles Ansible `teleport_server` et `teleport_agent` ne gèrent actuellement
pas les utilisateurs Teleport et ne remplacent donc pas cette configuration.

Une sauvegarde a été créée dans :

`/root/teleport-backup-20260729/`

Elle contient l'état des utilisateurs et des rôles avant la migration vers les
logins nominatifs.
