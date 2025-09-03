# Concepts de Base de Solana
### Modèle de Comptes de Solana  

Sur Solana, toutes les données sont stockées dans ce qu'on appelle des **« comptes »**. Tu peux imaginer les données sur Solana comme une grande base de données publique avec une seule table « Comptes », où chaque entrée de cette table est un « compte ». Chaque compte Solana partage la même structure de base.  

---

### **Comptes**  
![Comptes](lien-vers-image-comptes.png)  

**Points Clés à Retenir :**  
- Les comptes peuvent stocker jusqu'à **10 Mio** de données, qui contiennent soit du code exécutable (un programme), soit des données classiques.  
- Les comptes nécessitent un **dépôt de loyer** en lamports (SOL) proportionnel à la quantité de données stockées. Ce déposit est **récupérable** quand on ferme le compte.  
- Chaque compte a un **propriétaire** (un programme). Seul le programme propriétaire peut modifier les données du compte ou retirer des lamports. Mais n’importe qui peut ajouter des lamports.  
- Les **comptes Sysvar** sont des comptes spéciaux qui stockent l’état du réseau.  
- Les **comptes de programme** stockent le code exécutable des smart contracts.  
- Les **comptes de données** sont créés par les programmes pour stocker et gérer leur état.  

---

### **Compte**  
Chaque compte sur Solana a une **adresse unique de 32 octets**, souvent affichée sous forme de chaîne encodée en base58 (exemple : `14grJpemFaf88c8tiVb77W7TYg2W3ir6pfkKz3YjhhZ5`).  

La relation entre le compte et son adresse fonctionne comme une paire **clé-valeur** : l’adresse est la clé pour trouver les données sur la blockchain. L’adresse du compte agit comme un **« ID unique »** pour chaque entrée dans la table « Comptes ».  

![Adresse de Compte](lien-vers-image-adresse.png)  

La plupart des comptes Solana utilisent une clé publique **Ed25519** comme adresse.  

**Exemple de code (Kit) :**  
```typescript
import { generateKeyPairSigner } from "@solana/kit";

// Kit ne permet pas d’extraire les clés privées
const keypairSigner = await generateKeyPairSigner();
console.log(keypairSigner);
```

Cependant, Solana prend aussi en charge les **Adresses Dérivées de Programme (PDA)**. Les PDA sont des adresses spéciales qui peuvent être dérivées de manière déterministe à partir d’un ID de programme et d’entrées optionnelles (« seeds »).  

**Exemple de code (Kit) :**  
```typescript
import { Address, getProgramDerivedAddress } from "@solana/kit";

const programAddress = "11111111111111111111111111111111" as Address;

const seeds = ["helloWorld"];
const [pda, bump] = await getProgramDerivedAddress({
  programAddress,
  seeds
});

console.log(`PDA: ${pda}`);
console.log(`Bump: ${bump}`);
```

**Sortie console :**  
```
PDA: 46GZzzetjCURsdFPb7rcnspbEMnCBXe9kpjrsZAkKb6X
Bump: 254
```

---

### **Type de Compte**  
Les comptes ont une taille maximale de **10 Mio** et partagent tous la même structure de base.  

![Type de Compte](lien-vers-image-type-compte.png)  

Chaque compte sur Solana a les champs suivants :  

**Structure de Base d’un Compte :**  
```rust
pub struct Account {
    pub lamports: u64,        // Solde en lamports
    pub data: Vec<u8>,        // Données du compte
    pub owner: Pubkey,        // Adresse du programme propriétaire
    pub executable: bool,     // Si le compte est exécutable
    pub rent_epoch: Epoch,    // Ancien champ de loyer (plus utilisé)
}
```

#### **Champ Lamports**  
C’est le solde du compte en **lamports** (la plus petite unité de SOL : 1 SOL = 1 milliard de lamports).  

Un compte doit avoir un **solde minimum** de lamports proportionnel à la quantité de données stockées. Ce solde minimum s’appelle le **« loyer »**. Le solde est entièrement récupérable à la fermeture du compte.  

#### **Champ Data**  
Un tableau d’octets qui stocke des données arbitraires.  
- Pour les comptes de **programme** (smart contracts), ce champ contient le code exécutable.  
- Pour les comptes **non exécutables**, il stocke généralement des données à lire.  

Pour lire les données d’un compte Solana, il faut :  
1. Récupérer le compte en utilisant son adresse.  
2. **Désérialiser** le champ `data` (octets bruts) dans la structure de données appropriée, définie par le programme propriétaire.  

#### **Champ Owner**  
L’adresse du **programme propriétaire** de ce compte. Seul le programme propriétaire peut :  
- Modifier les données du compte (`data`).  
- Retirer des lamports du solde.  

Les instructions définies dans un programme déterminent comment les données et le solde peuvent être modifiés.  

#### **Champ Executable**  
- Si `true`, le compte est un **programme exécutable**.  
- Si `false`, le compte est un **compte de données**.  

Pour les comptes exécutables, le champ `owner` contient l’adresse d’un **programme de chargement** (« loader »), qui est responsable de charger et gérer le code exécutable.  

#### **Champ Rent Epoch**  
C’est un champ **obsolète** qui n’est plus utilisé. À l’origine, il indiquait quand un compte devait payer un loyer.  

---

### **Loyer (Rent)**  
Pour stocker des données sur la blockchain, les comptes doivent avoir un solde en lamports (SOL) proportionnel à la quantité de données stockées. Ce solde s’appelle le **« loyer »**, mais cela fonctionne plus comme un **dépôt** car il est intégralement remboursé à la fermeture du compte.  

Le terme « loyer » vient d’un ancien mécanisme qui retirait régulièrement des lamports des comptes qui n’avaient pas assez de solde. Ce mécanisme n’est plus actif.  

---

### **Propriétaire du Programme**  
Sur Solana, les « smart contracts » s’appellent des **programmes**. Chaque compte a un programme désigné comme propriétaire. Seul le programme propriétaire peut :  
- Changer le champ `data` du compte.  
- Déduire des lamports du solde.  

Chaque programme définit la structure des données stockées dans le champ `data` d’un compte.  

---

### **Programme Système**  
Par défaut, tous les nouveaux comptes sont propriétés du **Programme Système**. Ce programme effectue les actions clés suivantes :  

| Fonction | Description |
|----------|-------------|
| Création de nouveaux comptes | Seul le Programme Système peut créer de nouveaux comptes. |
| Allocation d’espace | Définit la capacité (en octets) du champ `data` de chaque compte. |
| Attribution de la propriété | Peut réattribuer la propriété d’un compte à un autre programme. |
| Transfert de SOL | Transfert des lamports (SOL) entre les comptes système. |

Tous les comptes de **portefeuille** (« wallet ») sur Solana sont des **comptes système** propriétés du Programme Système. Le solde en lamports de ces comptes représente le SOL détenu par le portefeuille. Seuls les comptes système peuvent payer les frais de transaction.  

![Compte Système](lien-vers-image-system-account.png)  

Quand du SOL est envoyé à une nouvelle adresse pour la première fois, un compte est automatiquement créé à cette adresse, propriété du Programme Système.  

**Exemple de code (Kit) :**  
```typescript
import {
  airdropFactory,
  createSolanaRpc,
  createSolanaRpcSubscriptions,
  generateKeyPairSigner,
  lamports
} from "@solana/kit";

// Connexion à un cluster Solana
const rpc = createSolanaRpc("http://localhost:8899");
const rpcSubscriptions = createSolanaRpcSubscriptions("ws://localhost:8900");

// Génère une nouvelle paire de clés
const keypair = await generateKeyPairSigner();
console.log(`Clé Publique : ${keypair.address}`);

// Envoyer des SOL à une adresse crée automatiquement un compte
const signature = await airdropFactory({ rpc, rpcSubscriptions })({
  recipientAddress: keypair.address,
  lamports: lamports(1_000_000_000n),
  commitment: "confirmed"
});

const accountInfo = await rpc.getAccountInfo(keypair.address).send();
console.log(accountInfo);
```

---

### **Comptes Sysvar**  
Les comptes Sysvar sont des comptes spéciaux à des adresses prédéfinies qui fournissent des données sur l’état du réseau (« cluster »). Ces comptes se mettent à jour dynamiquement.  

**Exemple de code (Kit) :**  
```typescript
import { createSolanaRpc } from "@solana/kit";
import { fetchSysvarClock, SYSVAR_CLOCK_ADDRESS } from "@solana/sysvars";

const rpc = createSolanaRpc("https://api.mainnet-beta.solana.com");

const accountInfo = await rpc
  .getAccountInfo(SYSVAR_CLOCK_ADDRESS, { encoding: "base64" })
  .send();
console.log(accountInfo);

// Récupère et désérialise automatiquement les données du compte
const clock = await fetchSysvarClock(rpc);
console.log(clock);
```

---

### **Compte de Programme**  
Déployer un programme Solana crée un **compte de programme exécutable**. Ce compte stocke le code exécutable du programme. Les comptes de programme sont propriétés d’un **Programme de Chargement** (« Loader »).  

![Compte de Programme](lien-vers-image-program-account.png)  

Pour faire simple, tu peux considérer le compte de programme comme le programme lui-même. Quand on appelle les instructions d’un programme, on spécifie l’adresse du compte de programme (souvent appelée « ID de Programme »).  

**Exemple de code (Kit) :**  
```typescript
import { Address, createSolanaRpc } from "@solana/kit";

const rpc = createSolanaRpc("https://api.mainnet-beta.solana.com");

const programId = "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA" as Address;

const accountInfo = await rpc
  .getAccountInfo(programId, { encoding: "base64" })
  .send();
console.log(accountInfo);
```

Quand tu déploies un programme Solana, il est stocké dans un compte de programme. Il existe plusieurs versions de programmes de chargement (« loaders »). La plupart stockent le code exécutable directement dans le compte de programme, mais le **loader-v3** stocke le code exécutable dans un **compte de données de programme** séparé.  

---

### **Compte de Données**  
Sur Solana, le code exécutable d’un programme est stocké dans un compte différent de son **état** (ses données). C’est comme sur un ordinateur où les programmes et leurs données sont dans des fichiers séparés.  

Pour gérer un état, les programmes créent des **comptes de données** distincts qu’ils possèdent. Chacun de ces comptes a sa propre adresse unique et peut stocker n’importe quelle donnée définie par le programme.  

![Compte de Données](lien-vers-image-data-account.png)  

Note : seul le **Programme Système** peut créer de nouveaux comptes. Ensuite, il peut transférer la propriété du nouveau compte à un autre programme.  

Créer un compte de données pour un programme personnalisé se fait en **deux étapes** :  
1. Appeler le Programme Système pour créer un compte, puis lui transférer la propriété.  
2. Appeler le programme personnalisé (qui est maintenant propriétaire) pour initialiser les données du compte.  

Ce processus est souvent simplifié, mais il est utile de comprendre comment ça fonctionne en dessous.  

**Exemple de code (Kit) : Création d’un compte de jeton (Token Mint)**  
```typescript
// [Code long exemple – voir texte original pour les détails]
// En résumé : cela crée un nouveau compte pour un jemon (mint) et l’initialise.
```

---

**Résumé :**  
- Les **comptes** sont la base du stockage de données sur Solana.  
- Ils ont une structure commune avec des champs pour le solde, les données, le propriétaire, etc.  
- Le **Programme Système** est crucial pour la création et la gestion des comptes.  
- Les programmes utilisent des **comptes de données** pour stocker leur état.  
- Comprendre ce modèle est essentiel pour développer sur Solana !
