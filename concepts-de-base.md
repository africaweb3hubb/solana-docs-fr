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

---

## Transactions et Instructions  

Sur Solana, les utilisateurs envoient des **transactions** pour interagir avec le réseau. Une transaction contient une ou plusieurs **instructions** qui spécifient les opérations à exécuter. La logique d'exécution de ces instructions est stockée dans des **programmes** déployés sur le réseau Solana, chaque programme définissant son propre ensemble d'instructions.  

**Points clés à retenir :**  
- Si une transaction contient plusieurs instructions, elles sont exécutées **dans l'ordre** où elles sont ajoutées.  
- Les transactions sont **atomiques** : soit **toutes** les instructions réussissent, soit **aucun** changement n'est appliqué.  
- Une transaction est essentiellement une **demande de traitement** d'une ou plusieurs instructions. Imagine une transaction comme une **enveloppe** contenant des formulaires (les instructions). Envoyer la transaction, c'est comme poster l'enveloppe pour faire traiter les formulaires.  

![Transaction Simplifiée](lien-image-transaction-simplifiee.png)  

---

### **Exemple : Transfert de SOL**  
Le schéma ci-dessous représente une transaction avec une **seule instruction** pour transférer des SOL d'un expéditeur à un destinataire.  

Sur Solana, les « portefeuilles » (wallets) sont des comptes appartenant au **Programme Système**. Seul le programme propriétaire peut modifier les données d'un compte. Donc, pour transférer des SOL, il faut envoyer une transaction qui appelle le Programme Système.  

![Transfert SOL](lien-image-transfert-sol.png)  

Le compte expéditeur doit **signer** la transaction (`is_signer`) pour autoriser le Programme Système à retirer des lamports de son solde. Les comptes expéditeur et destinataire doivent être **modifiables** (`is_writable`) car leurs soldes vont changer.  

Après l'envoi, le Programme Système traite l'instruction de transfert et met à jour les soldes des deux comptes.  

![Processus de Transfert SOL](lien-image-processus-transfert.png)  

**Exemple de code (Kit) :**  
```typescript
import {
  airdropFactory,
  appendTransactionMessageInstructions,
  createSolanaRpc,
  createSolanaRpcSubscriptions,
  createTransactionMessage,
  generateKeyPairSigner,
  getSignatureFromTransaction,
  lamports,
  pipe,
  sendAndConfirmTransactionFactory,
  setTransactionMessageFeePayerSigner,
  setTransactionMessageLifetimeUsingBlockhash,
  signTransactionMessageWithSigners
} from "@solana/kit";
import { getTransferSolInstruction } from "@solana-program/system";

// Connexion à un cluster Solana (local ici)
const rpc = createSolanaRpc("http://localhost:8899");
const rpcSubscriptions = createSolanaRpcSubscriptions("ws://localhost:8900");

// Génération des clés pour l'expéditeur et le destinataire
const sender = await generateKeyPairSigner();
const recipient = await generateKeyPairSigner();

const LAMPORTS_PER_SOL = 1_000_000_000n;
const transferAmount = lamports(LAMPORTS_PER_SOL / 100n); // 0.01 SOL

// On crédite l'expéditeur via un airdrop
await airdropFactory({ rpc, rpcSubscriptions })({
  recipientAddress: sender.address,
  lamports: lamports(LAMPORTS_PER_SOL), // 1 SOL
  commitment: "confirmed"
});

// Soldes avant le transfert
const { value: preBalance1 } = await rpc.getBalance(sender.address).send();
const { value: preBalance2 } = await rpc.getBalance(recipient.address).send();

// Création de l'instruction de transfert
const transferInstruction = getTransferSolInstruction({
  source: sender,
  destination: recipient.address,
  amount: transferAmount
});

// Construction du message de transaction
const { value: latestBlockhash } = await rpc.getLatestBlockhash().send();
const transactionMessage = pipe(
  createTransactionMessage({ version: 0 }),
  (tx) => setTransactionMessageFeePayerSigner(sender, tx), // Payeur des frais
  (tx) => setTransactionMessageLifetimeUsingBlockhash(latestBlockhash, tx), // Bloc de référence
  (tx) => appendTransactionMessageInstructions([transferInstruction], tx) // Ajout de l'instruction
);

// Signature et envoi de la transaction
const signedTransaction =
  await signTransactionMessageWithSigners(transactionMessage);
await sendAndConfirmTransactionFactory({ rpc, rpcSubscriptions })(
  signedTransaction,
  { commitment: "confirmed" }
);
const transactionSignature = getSignatureFromTransaction(signedTransaction);

// Soldes après le transfert
const { value: postBalance1 } = await rpc.getBalance(sender.address).send();
const { value: postBalance2 } = await rpc.getBalance(recipient.address).send();

// Affichage des résultats
console.log("Solde expéditeur avant :", Number(preBalance1) / Number(LAMPORTS_PER_SOL));
console.log("Solde destinataire avant :", Number(preBalance2) / Number(LAMPORTS_PER_SOL));
console.log("Solde expéditeur après :", Number(postBalance1) / Number(LAMPORTS_PER_SOL));
console.log("Solde destinataire après :", Number(postBalance2) / Number(LAMPORTS_PER_SOL));
console.log("Signature de la transaction :", transactionSignature);
```

---

### **Instructions**  
Une instruction sur Solana peut être vue comme une **fonction publique** que n'importe qui peut appeler sur le réseau.  

Imagine un programme Solana comme un **serveur web** hébergé sur le réseau, où chaque instruction est comme un **point d'API public** que les utilisateurs peuvent appeler pour effectuer des actions spécifiques. Appeler une instruction, c'est comme envoyer une **requête POST** à une API.  

Pour appeler l'instruction d'un programme, tu dois construire une structure `Instruction` avec trois informations :  

1.  **Program ID** : L'adresse du programme qui contient la logique de l'instruction.
2.  **Comptes** : La liste de **tous les comptes** que l'instruction lit ou modifie.
3.  **Données de l'instruction** : Un tableau d'octets qui spécifie *quelle* instruction appeler sur le programme et contient les arguments nécessaires.

**Structure d'une Instruction :**
```rust
pub struct Instruction {
    pub program_id: Pubkey,       // Adresse du programme
    pub accounts: Vec<AccountMeta>, // Liste des comptes
    pub data: Vec<u8>,            // Données (instruction + arguments)
}
```

![Instruction de Transaction](lien-image-instruction-transaction.png)  

#### **AccountMeta**  
Pour chaque compte requis par une instruction, tu dois fournir une structure `AccountMeta` qui spécifie :  

- `pubkey` : L'adresse du compte.
- `is_signer` : Si le compte doit **signer** la transaction.
- `is_writable` : Si l'instruction **modifie** les données ou le solde du compte.

**Structure AccountMeta :**
```rust
pub struct AccountMeta {
    pub pubkey: Pubkey,
    pub is_signer: bool,
    pub is_writable: bool,
}
```
En spécifiant à l'avance quels comptes une instruction lit ou écrit, le runtime Solana peut exécuter **en parallèle** les transactions qui ne modifient pas les mêmes comptes.  

En pratique, tu n'as généralement pas à construire une Instruction manuellement. La plupart des programmes fournissent des **bibliothèques clientes** avec des fonctions d'aide qui créent les instructions pour toi.  

![AccountMeta](lien-image-accountmeta.png)  

**Exemple de Structure d'Instruction :**  
*(Voir le code dans l'onglet console de l'exemple précédent pour voir la sortie JSON d'une instruction de transfert SOL)*

---

### **Transactions**  
Une fois que tu as créé les instructions à invoquer, l'étape suivante est de créer une **Transaction** et d'y ajouter les instructions.  

Une transaction Solana est composée de :  
- **Signatures** : Un tableau de signatures de tous les comptes requis comme signataires pour les instructions de la transaction.
- **Message** : Le message de transaction, qui inclut la liste des instructions à traiter de manière atomique.

**Structure d'une Transaction :**
```rust
pub struct Transaction {
    pub signatures: Vec<Signature>, // Les signatures
    pub message: Message,           // Le message
}
```

![Format de Transaction](lien-image-format-transaction.png)  

#### **Structure du Message**  
Le message a une structure bien définie :  
- **En-tête du message (Message Header)** : Spécifie le nombre de comptes signataires et en lecture seule.
- **Adresses des comptes** : Un tableau de toutes les adresses de comptes requis par les instructions.
- **Bloc de référence (Recent Blockhash)** : Agit comme un horodatage pour la transaction.
- **Instructions** : Un tableau des instructions à exécuter.

**Structure du Message :**
```rust
pub struct Message {
    pub header: MessageHeader,        // En-tête
    pub account_keys: Vec<Pubkey>,    // Adresses des comptes
    pub recent_blockhash: Hash,       // Bloc de référence
    pub instructions: Vec<CompiledInstruction>, // Instructions compilées
}
```

#### **Taille des Transactions**  
Les transactions Solana ont une **limite de taille de 1232 octets**. Cette limite vient de la taille MTU d'IPv6 (1280 octets), moins 48 octets pour les en-têtes réseau. La taille totale (signatures + message) doit respecter cette limite.  

#### **Bloc de Référence (Recent Blockhash)**  
Chaque transaction a besoin d'un **bloc de référence récent**. Il a deux utilités :  
1.  Il agit comme un **horodatage** pour la création de la transaction.
2.  Il **empêche** les transactions d'être rejouées (dupliquées).

Un bloc de référence **expire après 150 blocs** (environ 1 minute). Après cela, la transaction est considérée comme expirée et ne peut plus être traitée. Tu peux utiliser la méthode RPC `getLatestBlockhash` pour obtenir le bloc de référence actuel.  

#### **Tableau des Instructions**  
Les instructions dans le message sont dans un format compilé (`CompiledInstruction`). Chaque instruction compilée contient :  
- **Program ID Index** : Un index qui pointe vers l'adresse du programme dans le tableau des adresses de comptes.
- **Account Indexes** : Un tableau d'index qui pointent vers les comptes requis pour cette instruction.
- **Instruction Data** : Un tableau d'octets qui spécifie l'instruction à appeler et ses arguments.

**Structure CompiledInstruction :**
```rust
pub struct CompiledInstruction {
    pub program_id_index: u8,   // Index du programme
    pub accounts: Vec<u8>,      // Index des comptes
    pub data: Vec<u8>,          // Données de l'instruction
}
```

![Tableau Compact d'Instructions](lien-image-tableau-instructions.png)  

**Exemple de Structure de Transaction :**  
*(Voir le code dans l'onglet console pour voir la sortie JSON d'un message de transaction)*

---

### **Après l'Envoi**  
Après avoir soumis une transaction, tu peux récupérer ses détails en utilisant la méthode RPC `getTransaction` avec sa **signature** (qui est simplement la première signature de la transaction, celle du payeur des frais). Tu peux aussi inspecter la transaction sur **Solana Explorer**.  

La réponse contiendra des informations comme :  
- `blockTime` : Quand la transaction a été traitée.
- `meta.fee` : Les frais de transaction payés.
- `meta.err` : Une erreur éventuelle.
- `meta.logMessages` : Les logs du programme.
- `preBalances` / `postBalances` : Les soldes des comptes avant et après.
