[cron](https://www.linuxtricks.fr/wiki/cron-et-crontab-le-planificateur-de-taches) est un outil linux très répandus qui sert à automatiser l'execution de commande bash ou d'un script à interval régulier

## installation

```bash
sudo apt install cron
```

## configuration

Pour venir ajouter une tâche à faire tourner à interval régulier, on vient l'ajouter dans la **crontab**

```bash
sudo crontab -e
```
*J'utilise ici cron en mode sudo, afin que ses scripts se lancent aussi en mode sudo, ce n'est peut-être pas une pratique recommandable/optimisée*

Pour choisir l'occurence on va jouer sur ces **5 étoiles** en début de ligne

![](../../__images/Pasted%20image%2020260509185918.png)

Des outils comme [crontab.guru](https://crontab.guru/ ) pourront vous faciliter la vie pour vérifier vos configurations de *cron* 
