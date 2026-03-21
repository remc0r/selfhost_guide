
***WIP mais commande à executer d'urgence***

```bash
# installer firewall  
sudo apt install ufw  
  
# voir état  
sudo ufw status  
  
# politique par défaut  
sudo ufw default deny incoming  
sudo ufw default allow outgoing  
  
# autoriser Tailscale uniquement  
sudo ufw allow from 100.64.0.0/10  
  
# activer firewall  
sudo ufw enable  
  
# vérifier  
sudo ufw status verbose
```

![](../__images/Pasted%20image%2020260318220720.png)
