## Resources

Dans les normes de sécurité actuelle, il est préférable d'utiliser une *phrase secrète* plutôt qu'un mot de passe robuste comme on a l'habitude d'en voir avec pléthore de caractères spéciaux.

![Pasted image 20260310174423](/__images/Pasted%20image%2020260310174423.png)

Article expliquant le choix d'un mot de passe robuste :[https://lecrabeinfo.net/tutoriels/choisir-un-bon-mot-de-passe-fort-et-securise-avec-diceware/](https://lecrabeinfo.net/tutoriels/choisir-un-bon-mot-de-passe-fort-et-securise-avec-diceware/ "https://lecrabeinfo.net/tutoriels/choisir-un-bon-mot-de-passe-fort-et-securise-avec-diceware/")

https://x.com/iv2fi/status/2031401887367397461?s=46

[https://diceware.fr/](https://diceware.fr/ "https://diceware.fr/")

Tester la robustesse de son mot de passe: [https://bitwarden.com/fr-fr/password-strength/](https://bitwarden.com/fr-fr/password-strength/ "https://bitwarden.com/fr-fr/password-strength/")

---
## VaultWarden

[Vaultwarden](https://github.com/dani-garcia/vaultwarden) est un fork open-source du gestionnaire de mot de passe [Bitwarden](https://bitwarden.com/fr-fr/)

### Installation

On peut facilement l'auto-héberger dans un docker avec la configuration suivante :

```yaml
services:
  vaultwarden:
      container_name: vaultwarden
      volumes:
          - /srv/appdata/vaultwarden:/data
     #environment:
         #- SIGNUPS_ALLOWED=false
      image: 'vaultwarden/server:latest'
      ports:
        - "8081:80"
      restart: always
```

Une fois configuré et déployé via le flux : [Créer son propre tunnel Cloudflare](../Configuration/Créer%20son%20propre%20tunnel%20Cloudflare.md) [Procédure d'installation d'un nouveau service](../Configuration/Procédure%20d'installation%20d'un%20nouveau%20service.md)

### Extension navigateur

Il ne reste plus qu'a installer [l'application de bureau ou l'extension de navigateur bitwarden](https://bitwarden.com/fr-fr/download/)
Dans l'extension, choisir `self-hosted`
![Pasted image 20260310174221](/__images/Pasted%20image%2020260310174221.png)

Et indiquer l'url de votre VaultWarden.

Sur l'extension on peut également générer des mots de passes, et choisir les conditions de génération pour ceux-ci

![Pasted image 20260310180003](/__images/Pasted%20image%2020260310180003.png)

### Interface Web et outils

Sur l'interface Web de Vaultwarden, on peut retrouver beaucoup d'outils et de rapports nous aidant à gérer nos mots de passe réutilisés, exposés, faibles etc.

![Pasted image 20260310174758](/__images/Pasted%20image%2020260310174758.png)

