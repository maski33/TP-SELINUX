## TP-SELinux

### Conf firewalld:

![Configuration firewalld](https://imgur.com/a/MO4Yl0w)

### Conf sshd

![Configuration sshd 1](https://imgur.com/GNjw6yE)
![Configuration sshd 2](https://imgur.com/WKVcRyJ)
![Configuration sshd 3](https://imgur.com/1lcwWRp)


### SELinux dispose de différents modes, quels sont-ils ? Est a quoi sert chaque mode ?

```bash
Disabled (Désactivé) : L’accès n’est pas restreint, et rien n’est
enregistré dans les logs

Permissive (permissif) : Les règles SELinux sont interrogées,
les erreurs d’accès sont enregistrées dans les logs, mais l’accès
ne sera pas bloqué.

Enforcing (strict) : Les accès sont restreints en fonction des
règles SELinux en vigueur sur la machine.
```

### Que se passe-t-il si un profil Selinux est configuré en mode « enforce » et qu’il ne convient pas parfaitement au binaire associé ?

```bash
Lorsqu'un profil SELinux est configuré en mode "enforcing" et qu'il ne correspond pas parfaitement au comportement attendu d'un programme ou binaire spécifique, cela peut entraîner des restrictions d'accès ou des erreurs d'exécution.
```

### Quel est le contexte des différents fichiers du serveur web Apache ?

```bash
[root@vbox ~]# ls -Z /var/www/html
unconfined_u:object_r:httpd_sys_content_t:s0 index.html

[root@vbox ~]# ls -Z /etc/httpd
system_u:object_r:httpd_config_t:s0 conf                     system_u:object_r:etc_t:s0 modules
system_u:object_r:httpd_config_t:s0 conf.d                   system_u:object_r:etc_t:s0 run
system_u:object_r:httpd_config_t:s0 conf.modules.d           system_u:object_r:etc_t:s0 state
system_u:object_r:etc_t:s0 logs

[root@vbox ~]# ls -Z /var/log/httpd
system_u:object_r:httpd_log_t:s0 access_log  system_u:object_r:httpd_log_t:s0 error_log

[root@vbox ~]# ls -Z /usr/sbin/httpd
system_u:object_r:httpd_exec_t:s0 /usr/sbin/httpd
```

### Quel est le contexte du service Apache ?

```bash
[root@vbox ~]# ps -eZ | grep httpd
system_u:system_r:httpd_t:s0        760 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0        831 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0        832 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0        833 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0        836 ?        00:00:00 httpd
system_u:system_r:httpd_t:s0       3099 ?        00:00:00 httpd
```

### Activez le mode « enforce » sur le profil apache, le serveur web fonctionne-t-il ?

```bash
En mode "enforcing", SELinux applique strictement la politique.

Si la configuration et les contextes (par exemple, les fichiers dans /var/www/html labellisés en httpd_sys_content_t) sont corrects, Apache continue de fonctionner normalement.

Sinon, SELinux bloquera certaines actions (accès aux fichiers ou aux sockets) et le serveur web pourra rencontrer des erreurs.
```

### Désactivez le mode « enforce » puis modifiez la configuration du serveur Apache pour placer le path du serveur web dans /srv/srv/srv_1/. Redémarrez ensuite le service Apache.

```bash
En passant en mode "permissive" (via sudo setenforce 0), SELinux ne bloque pas les actions même si les fichiers n'ont pas le bon contexte.

On modifie alors la configuration d’Apache (par exemple dans le fichier de configuration principal) pour définir le DocumentRoot sur /srv/srv/srv_1/.

Après avoir enregistrer les modifications et fait un :
sudo systemctl restart httpd

En mode permissif, le service démarre sans être bloqué par SELinux, même si les nouveaux fichiers dans /srv/srv/srv_1/ n’ont pas le contexte attendu.
```

### Activez de nouveau le mode « enforce » sur le profil Apache. Le serveur web est-il de nouveau accessible ? Pourquoi ?

```bash
Une fois en mode enforcing avec la commande setenforce 1 SELinux réapplique ses règles.

Les fichiers dans le nouveau répertoire /srv/srv/srv_1/ n’ont pas automatiquement le bon contexte.

SELinux bloquera l’accès d’Apache à ces fichiers et le site ne sera plus accessible.

La raison principale est que la politique SELinux attend que le contenu servi par Apache porte le bon label. Sans ajustement, le nouveau chemin n’est pas autorisé.
```

### Ajuster le profil SELinux avec « sealert » pour correspondre à la nouvelle configuration du service Apache. Expliquez brièvement l’utilité et la méthode d’utilisation de « sealert ».

```bash
[root@vbox ~]# sudo sealert -a /var/log/audit/audit.log
Cette commande génère un rapport expliquant les AVC (Access Vector Cache) et propose des solutions.

[root@vbox ~]# sudo semanage fcontext -a -t httpd_sys_content_t '/srv/srv/srv_1(/.*)?'
cette commande applique les corrections
```

### Une fois le profil modifié et activé en mode enforce, le service Apache est-il accessible depuis le navigateur ?

```bash
Après avoir ajusté le profil SELinux en appliquant le module personnalisé qui autorise Apache à accéder aux fichiers dans le nouveau chemin et en remettant SELinux en mode "enforcing" le service Apache sera de nouveau accessible.

La modification du profil avec sealert a permis d'autoriser explicitement l'accès au contenu dans /srv/srv/srv_1/.

Ainsi, SELinux n’interfère plus avec l’accès aux fichiers, et Apache peut servir le contenu normalement.
```