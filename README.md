# Secureserver
PDF de méthode de Protection 

Linux le coeur d’un serveur
Création : le 6 février 2015
Bien qu’étant un archlinuxien acharner ,j’ai décidé qu’il tournerait sous Debian , autre distribution pour
laquelle j’ai beaucoup d’affection.
Sans vouloir m’embarquer dans une longue prose concernant la sécurité d’un serveur et les marches à
suivre concernant celle-ci (le web en pullule, de toutes façons), voici plutôt la présentation et la
configuration d’une des facettes de celle-ci (mais ne peut suffire, soyez-en conscient) comme j’ai pu la
faire (avec peut-être quelques différences) sur mon serveur.
I can haz firewall
Refusant de jouer péniblement avec iptables, j’ai cherché un logiciel qui automatiserait mes règles de
firewall sans me prendre excessivement la tête. C’est pourquoi j’ai jeté mon dévolu sur quelque frontend
à iptables : APF Firewall, de et par R-Fx Networks.
APF Firewall : introduction
Outre la facilité de configuration d’APF, ce qui m’a séduit c’est sa capacité à importer directement et
automatiquement des listes d’adresses sur Project Honey Pot, Spamhaus, DShield, … et bloquer celles-ci
afin d’offrir une première protection contre les spammeurs en tout genre, qu’importe le CMS utilisé.
Que vous installiez depuis l’archive ou les répertoires de Debian, la configuration est similaire à quelques
détails près, donc à votre guise. Pour ma part, j’ai préféré la version des dépôts, par facilité (aptitude >
wget && tar && sh), mais libre à vous de vous amuser avec la version archivée, que vous pouvez trouver
sur cette page. Après avoir installé le paquet, il ne reste plus qu’à configurer au mieux le programme, et
c’est, dans mon cas, le fichier /etc/apf-firewall/conf.apf que je dois éditer.
APF Firewall : configuration
Pour débuter, laissez inchangée la ligne DEVEL_MODE="1", celle-ci pourrait vous sauver la mise en cas
de problème de configuration : cela désactivera automatiquement le pare-feu au bout de 5 minutes au
moyen d’une tâche cron. Une fois que tout est bien configuré, il suffit de passer sa valeur à 0 et le tour est
joué.
Il est évident que la configuration suivante n’est qu’un exemple, j’invite plus que vivement à lire les
commentaires du fichier de configuration afin de permettre une configuration des plus fines et des plus
adaptées.
Sur ma configuration, iptables fait partie d’un noyau monolithique, donc le paramètre
SETMONOKERN="1" est vital sans quoi rien ne fonctionne dans mon cas. Pareillement, il est essentiel
d’adapter en conséquence les paramètres IFACEIN et IFACE_OUT avec la valeur adéquate donnée par
ifconfig. Dans mon cas il s’agit de venet0.
En vrac, voici ce que j’ai activé :
SET_VNET="1" ;
SET_TRIM="0" pour éviter d’être limité en nombre de règles ;
PKT_SANITY="1", PKT_SANITY_INV="1", PKT_SANITY_FUDP="1",
PKT_SANITY_PZERO="1", PKT_SANITY_STUFFED="1" ;
BLK_MCATNET="1", BLK_IDENT="1" ;
SYSCTL_TCP="1", SYSCTL_SYN="1", SYSCTL_ROUTE="1",
SYSCTL_LOGMARTIANS="1", SYSCTL_SYNCOOKIES="1" ;
Dans le port 80, y’a des paquets qui passent
Ensuite, il convient de configurer les ports à ouvrir, puisque tout autre port sera, lui, bloqué. Dans mon
cas, j’ai ouvert les ports 22 (SSH et SFTP), 25 (SMTP) 53 (DNS), 80 (HTTP), 110 (POP3) , 123 (NTP),
143 (IMAP), 443 (HTTPS), 5222 (XMPP – client), 5269 (XMPP – serveur), 9418 (git). Il est évident que
cette configuration est essentielle pour les services que vous utilisez (les ports 20 et 21 pour FTP, ou
11371 pour HKP par exemple). Pour plus d’informations concernant les numéros de ports ou les
protocoles TCP et UDP, cette liste des ports logiciels est plus qu’utile.
Dans mon cas, donc, les filtres pour le trafic entrant sont les suivants :
IG_TCP_CPORTS="22,25,53,80,110,143,443,5222,5269,9418" ;
IG_UDP_CPORTS="53,123,9418".
Tandis que pour les trafic sortant j’ai ceci :
EGF="1" afin d’en activer la gestion ;
EG_TCP_CPORTS="22,25,53,80,110,143,443,5222,5269" ;
EG_UDP_CPORTS="53".
Remote Lists
Comme je l’ai introduit, l’une des particularités d’APF est de permettre le téléchargement automatique de
règles d’exclusion, ce qu’il faut bien évidemment activer :
DLIST_PHP="1" pour le Project Honey Pot ;
DLIST_SPAMHAUS="1" pour la liste DROP de Spamhaus ;
DLIST_DSHIELD="1" pour DShield ;
DLIST_RESERVED="1".
Lancement et mise en production sur un VPS Debian
Une fois que tout est bien configuré, il est temps d’essayer le tout. Pour ce faire, deux commandes à
retenir :
sudo service apf-firewall restart ;
sudo apf -r.
S’il y a un bug, on recommence jusqu’à ce qu’il n’y ait plus d’erreur. Pour vérifier l’ampleur des règles,
un petit iptables -L -n suffit, tandis que sudo apf -h permet de jeter un oeil aux possibilités de la
commande.
DEVEL_MODE="0" ;
SET_FASTLOAD="1" histoire de gagner en temps de chargement.
Cas particulier des VPS : modules iptables
Si vous obtenez l'erreur "iptables: No chain/target/match by that name", il est indiqué dans le README
les différents modules à charger (ipt_recent est facultatif, par exemple, et avait causé de gros soucis chez
moi). Demandez à votre hébergeur / fournisseur si vous ne pouvez les gérer vous-même.
ip_tables
iptable_filter
iptable_mangle
ip_conntrack
ip\_conntrack\_irc
ip\_conntrack\_ftp
ipt_state
ipt_multiport
ipt_limit
ipt_recent
ipt_LOG
ipt_REJECT
ipt_ecn
ipt_length
ipt_mac
ipt_multiport
ipt_owner
ipt_state
ipt_ttl
ipt_TOS
ipt_TCPMSS
ipt_ULOG
Knock knock knockin on server’s door
Who's there ?
Après avoir installé et configuré APF, il convient maintenant d’en configurer l’extension normale et
logique : BFD, par R-Fx Networks également. Cette fois, il n’y a pas de paquet dans les dépôts, donc
c’est téléchargement, extraction, et configuration à la main.
wget http://www.rfxn.com/downloads/bfd-current.tar.gz ;
tar xvfz bfd-current.tar.gz ;
cd bfd-X.Y où X.Y est la version affichée par tar ;
./install.sh pour installer.
Ce serait tout beau, tout mignon, s’il n’y avait un petit problème mineur : cette version ne reconnaît pas
celle d’APF installée depuis les dépôts. Pour que tout fonctionne, donc, il faut éditer
/usr/local/bfd/conf.bfd comme suit :
EMAIL_ALERTS="1" histoire d’être averti s’il y a quelqu’un qui fait du grabuge ;
EMAIL_ADDRESS="tonjoliemail@tonjolidomaine" histoire de savoir qui prévenir ;
BAN_COMMAND="/usr/sbin/apf -d $ATTACK_HOST {bfd.$MOD}" histoire de pointer sur un
programme qui existe (pour savoir où est le programme chez vous, un petit whereis apf s'impose,
bien évidemment)
Le reste peut être laissé en l’état. Lancez le bousin !
sudo /usr/local/sbin/bfd -s
DDoS Deflate : COME AT ME BRO(s) !
Certes si votre configuration est un poil maigrichonne, se croire inattaquable est une utopie digne de la
logique la plus médiocre. Nulle raison pourtant de ne pas tenter quelques mesures, dont l'installation de
DDoS Deflate, oui monsieur (ou madame, pardon), afin d'envoyer un beau "Go fuck yourself" au moins
temporaire à quelques robots / zouaves s'amusant dans vos contrées.
Téléchargement et installation
La chose est résolument simple : télécharger le script, installer le script, ce qui donne :
wget http://www.inetbase.com/scripts/ddos/install.sh ;
sh install.sh ;
Rien de plus simple sinon d'accepter la licence de publication du script, et de noter les chemins
d'installation histoire de pouvoir éditer la configuration.
Configuration et édition
La configuration dans mon cas se fait dans le fichier /usr/local/ddos/ddos.conf, que j'ai édité de la sorte
afin de prendre en compte mes différents programmes :
##### Paths of the script and other files
PROGDIR="/usr/local/ddos"
PROG="/usr/local/ddos/ddos.sh"
IGNORE\_IP\_LIST="/usr/local/ddos/ignore.ip.list"
CRON="/etc/cron.d/ddos.cron"
APF="/usr/sbin/apf"
IPT="/sbin/iptables"
Mais j'ai aussi dû éditer /usr/local/ddos/ddos.sh qui avait quelques problèmes : d'abord le shebang
#!/bin/sh en #!/bin/bash - ainsi que toutes les occurences de /bin/sh tant qu'à faire -, et crond en cron :
#!/bin/bash
########################################
# DDoS-Deflate version 0.6 Author: Zaf #
########################################
(...)
unbanip()
{
UNBAN_SCRIPT=`mktemp /tmp/unban.XXXXXXXX`
TMP_FILE=`mktemp /tmp/unban.XXXXXXXX`
UNBAN_IP_LIST=`mktemp /tmp/unban.XXXXXXXX`
echo '#!/bin/bash' > $UNBAN_SCRIPT
echo "sleep $BAN_PERIOD" >> $UNBAN_SCRIPT
if [ $APF_BAN -eq 1 ]; then
while read line; do
echo "$APF -u $line" >> $UNBAN_SCRIPT
echo $line >> $UNBAN_IP_LIST
done < $BANNED_IP_LIST
else
while read line; do
echo "$IPT -D INPUT -s $line -j DROP" >> $UNBAN_SCRIPT
echo $line >> $UNBAN_IP_LIST
done < $BANNED_IP_LIST
fi
echo "grep -v --file=$UNBAN_IP_LIST $IGNORE_IP_LIST > $TMP_FILE" >>
$UNBAN_SCRIPT
echo "mv $TMP_FILE $IGNORE_IP_LIST" >> $UNBAN_SCRIPT
echo "rm -f $UNBAN_SCRIPT" >> $UNBAN_SCRIPT
echo "rm -f $UNBAN_IP_LIST" >> $UNBAN_SCRIPT
echo "rm -f $TMP_FILE" >> $UNBAN_SCRIPT
. $UNBAN_SCRIPT &
}
add\_to\_cron()
{
rm -f $CRON
sleep 1
service cron restart
sleep 1
echo "SHELL=/bin/bash" > $CRON
if [ $FREQ -le 2 ]; then
echo "0-59/$FREQ * * * * root /usr/local/ddos/ddos.sh >/dev/null 2>&1" >> $CRON
else
let "START_MINUTE = $RANDOM % ($FREQ - 1)"
let "START_MINUTE = $START_MINUTE + 1"
let "END_MINUTE = 60 - $FREQ + $START_MINUTE"
echo "$START_MINUTE-$END_MINUTE/$FREQ * * * * root /usr/local/ddos/ddos.sh
>/dev/null 2>&1" >> $CRON
fi
service cron restart
}
(...)
Pour finir, un simple /usr/local/ddos/ddos.sh -c permet d'ajouter les règles au crontab, que tout système a -
du moins habituellement et pourra bloquer de façon cyclique ceux qui s'amusent un peu trop sur vos
pages.
