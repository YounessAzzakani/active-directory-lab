# active-directory-lab
# Déploiement d’un Active Directory & Laboratoire Offensif

> **Avertissement** : contenu strictement pédagogique. Toutes les attaques et PoC décrits dans ce dépôt ont été effectués **uniquement** dans un environnement de laboratoire isolé et contrôlé. Ne reproduisez jamais ces actions sur des systèmes dont vous n'avez pas l'autorisation explicite. 

---

## Objectif
Ce dépôt regroupe les rapports et preuves de concept (TP1, TP2, TP3) autour du déploiement et de la sécurisation d’un domaine **Active Directory**, ainsi que la démonstration de techniques offensives visant à comprendre les risques et proposer des contre-mesures. Les objectifs pédagogiques incluent : installation et administration d’un DC, configuration de GPO, identification et exploitation de vulnérabilités SMB/Netlogon, et compréhension des attaques post-compromission (extraction de secrets, abus Kerberos). 

---

## Résumé

### Scope of Testing (Périmètre des tests)
Les tests ont été réalisés sur un laboratoire composé de VMs (contrôleur de domaine Windows Server, postes clients Windows 7/10, VM attaquante). Les actions incluent : configuration AD et GPO (TP1), scans/énumération et exploitation SMB/Netlogon (TP2), et techniques post-compromission ciblant la récupération de credentials et l’abus Kerberos (TP3). 

### Méthodologie
1. **Préparation** : déploiement des VMs et configuration réseau isolé. 
2. **Reconnaissance** : scans Nmap, énumération SMB/NetBIOS, vérification de partages et SPN.
3. **Exploitation contrôlée** : utilisation de PoC/outils (Metasploit pour EternalBlue, outils zerologon) sur cibles vulnérables.   
4. **Post-compromission** : dump mémoire/LSASS (Mimikatz), extraction NTDS.dit, PtH/PtT/Golden Ticket, Kerberoasting.   
5. **Analyse & mitigation** : évaluation des impacts et recommandations de durcissement.

### Constats généraux
- TP1 : AD correctement déployé ; GPO efficaces pour restreindre l’usage (ex. blocage de cmd, règles SRP). 
- TP2 : Présence de systèmes vulnérables (SMBv1 / Windows 7) exploitable via EternalBlue ; Zerologon testé et confirmé sur le DC dans le laboratoire (impact critique).   
- TP3 : Après compromission initiale, il est possible d’extraire des credentials (LSASS dump), d’exécuter PtH/PtT, de forger Golden Tickets et d’exfiltrer `NTDS.dit`, entraînant une compromission complète du domaine si non maîtrisé. 

### Détails d’exploitation
- **EternalBlue (MS17-010)** : détection par script Nmap / exploitation via Metasploit → session Meterpreter avec privilèges SYSTEM sur poste vulnérable.
- **Zerologon (CVE-2020-1472)** : test/PoC démontrant la possibilité de réinitialiser le mot de passe machine du DC, aboutissant à un contrôle total du domaine dans le labo (PoC exécuté et confirmé).
- **Post-compromission** : dump LSASS (Mimikatz ou minidump via PowerSploit), secretsdump pour extraire NTLM, PsExec/PtH pour mouvement latéral, génération de Golden/Silver Tickets et Kerberoasting suivis de bruteforce des TGS. 

---

## Outils et techniques
- **Outils de découverte & exploitation** : `nmap`, `Metasploit`, scripts PoC (zerologon_tester, zerologon-NullPass).   
- **Outils d’extraction et post-exploitation** : `mimikatz`, `impacket` (secretsdump.py), `PsExec`, `PowerSploit`, `hashcat`.   
- **Administration AD / durcissement** : `gpmc.msc`, `gpupdate`, `gpresult`, PowerShell pour la gestion d’OU/utilisateurs et création de GPO. 

---

## Constats clés
1. **Vulnérabilités critiques identifiées** : EternalBlue et Zerologon permettent respectivement RCE et compromission du DC — impact de type prise de contrôle complète du domaine.   
2. **Exposition due à des systèmes non patchés / configurations faibles** : SMBv1 activé, comptes de service ou mots de passe faibles, absence d’isolation.   
3. **Post-compromission puissant** : une fois l’accès initial obtenu, techniques comme Mimikatz, PtH/PtT et Golden Ticket permettent une persistance et un contrôle étendu.   
4. **Contrôles GPO efficaces mais dépendants d’une bonne configuration** : GPO appliquées ont permis des protections basiques (politiques de mot de passe, SRP), mais ne suffisent pas seules contre des vulnérabilités critiques non corrigées.

---

## Recommandations
1. **Patch management immédiat** : appliquer les correctifs Microsoft pour Netlogon (Zerologon) et SMB (MS17-010) ; désactiver SMBv1.   
2. **Sécuriser les comptes privilégiés** : mots de passe forts, rotation régulière (notamment `krbtgt`), limiter les comptes de service et appliquer le principe du moindre privilège.   
3. **Protection LSASS / défense endpoint** : activer Credential Guard ou protections équivalentes, déployer EDR et empêcher les outils d’extraction en mémoire. 
4. **Surveillance & détection** : centraliser logs (SIEM), détecter anomalies Kerberos (ex. pics de requêtes TGS), monitorer créations/modifications inhabituelles sur le DC. 
5. **Segmentation réseau & backups** : isoler DCs, segmenter réseaux, conserver snapshots/verrouillage d’accès et procédures de restauration testées.
6. **Durcissement AD** : revoir GPO sensibles, limiter l’accès aux partages SYSVOL/NETLOGON, appliquer SRP et restreindre l’usage de comptes locaux administratifs.

---

## Conclusion
Les TPs montrent clairement que **la combinaison de systèmes non patchés, de configurations faibles et d’outils d’attaque accessibles** peut conduire à une compromission complète d’un domaine Active Directory. La meilleure défense repose sur une approche multi-couche : correctifs rapides, durcissement des comptes et des endpoints, surveillance active et segmentation du réseau. Ces travaux illustrent l’importance d’exécuter des exercices offensifs en laboratoire contrôlé pour mieux préparer des mesures défensives réalistes. 

---

### à consulter
- TP1 — Mise en place et gestion d’Active Directory & GPO. :contentReference[oaicite:28]{index=28}  
- TP2 — Attaques sur SMB / EternalBlue / Zerologon. :contentReference[oaicite:29]{index=29}  
- TP3 — Attaques post-compromission (Mimikatz, PtH/PtT, Golden Ticket, Kerberoasting, exfiltration NTDS.dit). :contentReference[oaicite:30]{index=30}
