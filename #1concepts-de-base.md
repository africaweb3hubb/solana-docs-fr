# Premier Concepts de Base de SOLANA

### Commencez à apprendre à partir d'ici avec **Africaweb3hub*

## `Il est recommandé d'avoir une compréhension de base des concepts fondamentaux de Solana.`

Pour construire sur Solana, il est essentiel de comprendre plusieurs concepts clés uniques à son développement. Cette section couvre les concepts fondamentaux que tu dois connaître pour commencer à bâtir sur Solana, y compris les comptes, les transactions, les programmes, et plus encore.  

---

### **Modèle de Comptes de Solana**  
Sur Solana, toutes les données sont stockées dans ce qu'on appelle des **« comptes »**. La façon dont les données sont organisées sur la blockchain Solana ressemble à un **magasin clé-valeur**, où chaque entrée dans la base de données est un « compte ».  

📖 **En savoir plus sur les [Comptes](lien-vers-la-page-comptes) ici.**  

---

### **Transactions et Instructions**  
Sur Solana, on envoie des **transactions** pour interagir avec le réseau. Les transactions contiennent une ou plusieurs **instructions**, chacune représentant une opération spécifique à exécuter. La logique d'exécution des instructions est stockée dans des **programmes** déployés sur le réseau Solana, chaque programme définissant son propre ensemble d'instructions.  

📖 **En savoir plus sur les [Transactions et Instructions](lien-vers-la-page-transactions) ici.**  

---

### **Frais sur Solana**  
La blockchain Solana a quelques types différents de **frais** et de **coûts** pour utiliser le réseau. On peut les segmenter en plusieurs types spécifiques :  

- **Frais de Transaction** : Frais pour que les validateurs traitent les transactions/instructions.  
- **Frais de Priorisation** : Frais optionnels pour augmenter la priorité de traitement des transactions.  
- **Loyer (« Rent »)** : Un solde bloqué pour garder les données stockées sur la blockchain.  

📖 **En savoir plus sur les [Frais sur Solana](lien-vers-la-page-frais) ici.**  

---

### **Programmes sur Solana**  
Sur Solana, les « smart contracts » s'appellent des **programmes**. Chaque programme est stocké dans un compte sur la blockchain et contient du **code exécutable** qui définit des instructions spécifiques. Ces instructions représentent les fonctionnalités du programme et peuvent être invoquées en envoyant des transactions au réseau.  

📖 **En savoir plus sur les [Programmes sur Solana](lien-vers-la-page-programmes) ici.**  

---

### **Adresses Dérivées de Programme (PDA)**  
Les **Adresses Dérivées de Programme** (PDA) offrent aux développeurs sur Solana deux cas d'utilisation principaux :  

1.  **Adresses de Compte Déterministes :** Les PDA fournissent un mécanisme pour dériver de manière déterministe une adresse en utilisant une combinaison de « seeds » (entrées prédéfinies optionnelles) et d'un ID de programme spécifique.  
2.  **Permettre la Signature par un Programme :** Le runtime Solana permet aux programmes de « signer » pour les PDA qui sont dérivés de leur propre ID de programme.  

Tu peux voir les PDA comme un moyen de créer des structures de type **hashmap** sur la blockchain à partir d'un ensemble prédéfini d'entrées (ex. : chaînes de caractères, nombres, d'autres adresses de compte).  

📖 **En savoir plus sur les [Adresses Dérivées de Programme](lien-vers-la-page-pda) ici.**  

---

### **Invocation Croisée de Programmes (CPI)**  
Une **Invocation Croisée de Programmes** (CPI) se produit lorsqu'un programme invoque les instructions d'un autre programme. Ce mécanisme permet la **composabilité** des programmes Solana.  

Tu peux imaginer les instructions comme des **points d'API** qu'un programme expose au réseau, et une CPI comme un API qui en appelle un autre en interne.  

📖 **En savoir plus sur l'[Invocation Croisée de Programmes](lien-vers-la-page-cpi) ici.**  

---

### **Jetons sur Solana**  
Les **jetons** sont des actifs numériques qui représentent la propriété sur diverses catégories d'actifs. La « tokenisation » permet la numérisation des droits de propriété et est un élément fondamental pour gérer à la fois des actifs fongibles et non fongibles.  

- **Jetons Fongibles** : Représentent des actifs interchangeables et divisibles du même type et de la même valeur (ex. : USDC).  
- **Jetons Non Fongibles (NFT)** : Représentent la propriété d'actifs indivisibles (ex. : œuvres d'art).  

📖 **En savoir plus sur les [Jetons sur Solana](lien-vers-la-page-jets) ici.**  

---

### **Clusters et Points de Terminaison (Endpoints)**  
La blockchain Solana a plusieurs groupes différents de validateurs, appelés **Clusters**. Chacun a un but différent et contient des nœuds dédiés pour répondre aux requêtes JSON-RPC.  

Il existe trois clusters principaux sur le réseau Solana, avec les points de terminaison publics suivants :  

- **Mainnet** - `https://api.mainnet-beta.solana.com` (réseau de production)  
- **Devnet** - `https://api.devnet.solana.com` (expérimentation pour les développeurs)  
- **Testnet** - `https://api.testnet.solana.com` (tests des validateurs)  

📖 **En savoir plus sur les [Clusters et Endpoints](lien-vers-la-page-clusters) ici.**  

---

**Résumé de la Section Concepts de Base :**  
Cette section t'a présenté les **briques fondamentales** pour comprendre et développer sur Solana. Chaque concept est crucial pour construire des applications décentralisées robustes.  

La suite du guide te montrera comment **appliquer** ces concepts en pratique pour créer tes premiers programmes et interactions !
