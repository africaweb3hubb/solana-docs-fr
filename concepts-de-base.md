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
---

 
## Frais de Transaction  

Chaque transaction sur Solana nécessite le paiement de **frais de base** (en SOL) pour rémunérer les validateurs qui traitent la transaction. Tu peux aussi payer des **frais de priorisation** (optionnels) pour augmenter les chances que le leader (validateur) actuel traite ta transaction en priorité.  

**Points Clés à Retenir :**  
- Les **frais de base** sont de **5000 lamports par signature** sur la transaction.  
- Les **frais de priorisation** (optionnels) sont un supplément que tu payes au validateur pour être prioritaire.  
- Les frais de priorisation se calculent ainsi : **(Limite d'unités de calcul × Prix par unité de calcul)**.  
- La **limite d'unités de calcul** est le nombre maximum d'unités que ta transaction peut utiliser.  
- Le **prix par unité de calcul** est le prix par unité, en **micro-lamports**.  
  - *1 000 000 micro-lamports = 1 lamport*  
- Le **payeur des frais** doit être un compte appartenant au **Programme Système**.  

---

### **Frais de Base**  
Les frais de base représentent le coût pour envoyer une transaction. Ce coût est de **5000 lamports par signature** incluse dans la transaction.  

Les frais de base sont prélevés sur le compte du **payeur des frais**, qui est le **premier signataire** de la transaction. Ce payeur doit être un compte appartenant au Programme Système.  

- **50% sont brûlés** : La moitié des frais de base est détruite.  
- **50% sont distribués** : L'autre moitié est versée au validateur qui a traité la transaction.  

---

### **Frais de Priorisation**  
Les frais de priorisation sont **optionnels**. Ils permettent d'augmenter les chances que le leader actuel traite ta transaction en priorité.  

**SIMD-0096 :** Le validateur qui traite la transaction reçoit **100%** des frais de priorité.  

---

### **Unités de Calcul et Limites**  
Quand une transaction est traitée, elle utilise des ressources de calcul mesurées en **unités de calcul** (Compute Units - CU). Chaque instruction consomme une partie du « budget » d'unités de calcul de la transaction.  

- **Limite Maximale :** Une transaction peut utiliser jusqu'à **1,4 million** d'unités de calcul.  
- **Limite par Défaut :** Par défaut, chaque instruction peut utiliser jusqu'à **200 000** unités de calcul.  
- **Limite Personnalisée :** Tu peux demander une limite spécifique en ajoutant une instruction `SetComputeUnitLimit` à ta transaction.  

Pour plus de détails sur l'utilisation des unités de calcul, consulte le guide [How to Request Optimal Compute](lien-vers-le-guide).  

---

### **Prix par Unité de Calcul**  
Le prix par unité de calcul est un montant optionnel, spécifié en **micro-lamports**, que tu acceptes de payer pour chaque unité de calcul demandée. Ce prix est utilisé pour calculer les frais de priorisation de ta transaction.  

*1 000 000 micro-lamports = 1 lamport*  

Tu peux utiliser ces services pour obtenir des recommandations en temps réel sur le prix actuel :  

| Fournisseur | API des Frais de Priorité |  
|-------------|----------------------------|  
| **Helius** | [Documentation](lien-doc-helius) |  
| **QuickNode** | [Documentation](lien-doc-quicknode) |  
| **Triton** | [Documentation](lien-doc-triton) |  

Voir le guide [How to Use Priority Fees](lien-vers-le-guide) pour plus de détails.  

---

### **Calcul des Frais de Priorisation**  
Les frais de priorisation se calculent ainsi :  

**Frais de Priorisation = Limite d'Unités de Calcul × Prix par Unité de Calcul**  

La méthode recommandée pour définir les frais de priorité est la suivante :  
1.  **Simule** d'abord la transaction pour estimer les unités de calcul nécessaires.  
2.  Ajoute une **marge de sécurité de 10%** à cette estimation.  
3.  Utilise la valeur obtenue comme **Limite d'Unités de Calcul**.  

La **priorité** de la transaction, qui détermine son ordre de traitement par rapport aux autres, est calculée avec cette formule :  

**Priorité = ((Limite d'UC × Prix par UC) + Frais de Base) / (1 + Limite d'UC + UC des Signatures + UC des Verrous en Écriture)**  

Utilise ces instructions dans ta transaction pour définir la limite et le prix :  
- `SetComputeUnitLimit` : Pour définir une limite d'unités de calcul spécifique.  
- `SetComputeUnitPrice` : Pour définir le prix par unité de calcul.  

Si tu ne fournis pas ces instructions, la transaction utilisera la **limite par défaut** avec un **prix de 0** (aucun frais de priorisation).  

**Attention :** Les frais de priorité dépendent de la **limite** que tu demandes, et non des unités **réellement utilisées**. Si tu définis une limite trop élevée ou utilise la valeur par défaut, tu risques de payer pour des unités de calcul que tu n'utilises pas !  

---

### **Exemples**  
Les exemples ci-dessous montrent comment définir la limite et le prix des unités de calcul pour une transaction.  

**SDK** | **Référence du Code Source**  
---|---  
`solana-sdk` (Rust) | [ComputeBudgetInstruction](lien-doc-rust)  
`@solana/web3.js` (TypeScript) | [ComputeBudgetProgram](lien-doc-ts)  

**Exemple de Code (TypeScript) :**  
```typescript
// Instructions pour le budget de calcul
const limitInstruction = ComputeBudgetProgram.setComputeUnitLimit({
  units: 300_000 // Limite personnalisée à 300 000 unités
});

const priceInstruction = ComputeBudgetProgram.setComputeUnitPrice({
  microLamports: 1 // Prix de 1 micro-lamport par unité
});

// Instruction de transfert SOL classique
const transferInstruction = SystemProgram.transfer({
  fromPubkey: sender.publicKey,
  toPubkey: recipient.publicKey,
  lamports: 0.01 * LAMPORTS_PER_SOL
});

// On ajoute TOUTES les instructions à la transaction
const transaction = new Transaction()
  .add(limitInstruction)   // Doit être ajoutée en premier
  .add(priceInstruction)   // En second
  .add(transferInstruction); // Puis l'instruction métier

// Envoi et confirmation de la transaction
const signature = await sendAndConfirmTransaction(connection, transaction, [sender]);
console.log("Signature de la transaction :", signature);
```

**Déroulement :**  
1.  On crée les instructions pour définir la `limite` et le `prix`.  
2.  On crée l'instruction de transfert (ou toute autre instruction métier).  
3.  On ajoute **d'abord** les instructions de budget de calcul à la transaction, **puis** l'instruction métier.  
4.  On envoie et confirme la transaction comme d'habitude.  

C'est ainsi que tu optimises tes transactions pour qu'elles soient traitées rapidement !

## Programmes  

Sur Solana, les « smart contracts » s'appellent des **programmes**. Les programmes sont déployés sur la blockchain dans des **comptes** qui contiennent le code exécutable compilé du programme. Les utilisateurs interagissent avec les programmes en envoyant des **transactions** qui contiennent des **instructions** indiquant au programme quoi faire.  

**Points Clés à Retenir :**  
- Les programmes sont des comptes contenant du **code exécutable**, organisé en fonctions appelées **instructions**.  
- Bien que les programmes eux-mêmes soient **sans état** (« stateless »), ils peuvent inclure des instructions qui **créent et mettent à jour** d'autres comptes pour stocker des données.  
- Une **autorité de mise à jour** (« upgrade authority ») peut modifier les programmes. Si cette autorité est révoquée, le programme devient **immutable** (impossible à modifier).  
- Les utilisateurs peuvent **vérifier** que le code d'un programme sur la blockchain correspond à son code source public grâce aux **compilations vérifiables** (« verifiable builds »).  

---

### **Écrire des Programmes Solana**  
Les programmes Solana sont principalement écrits en **Rust**, avec deux approches de développement courantes :  

1.  **Anchor** : Un **framework** conçu pour le développement de programmes Solana. Il permet d'écrire des programmes plus rapidement et plus simplement en utilisant des macros Rust pour réduire le code répétitif. Pour les débutants, il est recommandé de commencer avec Anchor.  
2.  **Rust Natif** : Cette approche consiste à écrire des programmes Solana en Rust sans utiliser de framework. Elle offre plus de flexibilité mais est aussi plus complexe.  

---

### **Mettre à Jour les Programmes Solana**  
Pour en savoir plus sur le déploiement et la mise à jour des programmes, consulte la page [deploying programs](lien-vers-la-page).  

Les programmes peuvent être modifiés directement par un compte désigné comme **« autorité de mise à jour »**, qui est généralement le compte ayant initialement déployé le programme. Si cette autorité est révoquée et définie sur `None`, le programme devient **immutable** et ne peut plus être mis à jour.  

---

### **Programmes Vérifiables**  
Les **compilations vérifiables** permettent à n'importe qui de vérifier que le code d'un programme sur la blockchain correspond à son code source public. Cela permet de détecter les différences entre les versions source et déployées.  

La communauté de développeurs Solana a créé des outils pour supporter les compilations vérifiables, permettant aux développeurs et aux utilisateurs de vérifier que les programmes sur la blockchain reflètent fidèlement leur code source public.  

- **Recherche de Programmes Vérifiés :** Pour vérifier rapidement, les utilisateurs peuvent rechercher une adresse de programme sur [Solana Explorer](https://explorer.solana.com/). Voir un exemple de programme vérifié [ici](lien-exemple).  
- **Outils de Vérification :** Le [Solana Verifiable Build CLI](lien-vers-l-outil) par Ellipsis Labs permet de vérifier indépendamment les programmes sur la blockchain par rapport au code source publié.  
- **Support dans Anchor :** Anchor fournit un support intégré pour les compilations vérifiables. Les détails se trouvent dans la [documentation Anchor](lien-doc-Anchor).  

---

### **Berkeley Packet Filter (BPF)**  
Solana utilise **LLVM** (Low Level Virtual Machine) pour compiler les programmes en fichiers **ELF** (Executable and Linkable Format). Ces fichiers contiennent une version personnalisée du bytecode eBPF, appelée **« Solana Bytecode Format » (sBPF)**. Le fichier ELF contient le binaire du programme et est stocké sur la blockchain dans un **compte exécutable** lorsque le programme est déployé.  

---

### **Programmes Intégrés (« Built-in Programs »)**  

#### **Programmes de Chargement (« Loader Programs »)**  
Chaque programme est lui-même la propriété d'un autre programme, son **chargeur** (« loader »). Il existe actuellement cinq programmes de chargement :  

| Chargeur | Program ID | Notes | Instructions |  
|----------|------------|-------|--------------|  
| **native** | `NativeLoader1111111111111111111111111111111` | Possède les quatre autres chargeurs. | — |  
| **v1** | `BPFLoader1111111111111111111111111111111111` | Instructions de gestion désactivées, mais les programmes s'exécutent. | — |  
| **v2** | `BPFLoader2111111111111111111111111111111111` | Instructions de gestion désactivées, mais les programmes s'exécutent. | [Instructions](lien) |  
| **v3** | `BPFLoaderUpgradeab1e11111111111111111111111` | En cours d'abandon. | [Instructions](lien) |  
| **v4** | `LoaderV411111111111111111111111111111111111` | **v4 devrait devenir le chargeur standard.** | [Instructions](lien) |  

Ces chargeurs sont nécessaires pour créer et gérer des programmes personnalisés :  
- Déployer un nouveau programme ou « buffer ».  
- Fermer un programme ou un buffer.  
- Redéployer / mettre à jour un programme existant.  
- Transférer l'autorité sur un programme.  
- Finaliser un programme.  

Les chargeurs **v3** et **v4** prennent en charge les modifications des programmes après leur déploiement initial. La permission de le faire est régulée par l'**autorité** d'un programme, car la propriété de chaque compte programme appartient au chargeur.  

---

#### **Programmes Précompilés**  

##### **1. Programme Ed25519**  
| Programme | Program ID | Description | Instructions |  
|-----------|------------|-------------|--------------|  
| **Ed25519** | `Ed25519SigVerify111111111111111111111111111` | Vérifie les signatures ed25519. Si une signature échoue, une erreur est retournée. | [Instructions](lien) |  

Ce programme traite une instruction. Le premier `u8` est le nombre de signatures à vérifier, suivi d'un octet de remplissage. Ensuite, la structure suivante est sérialisée, une pour chaque signature à vérifier.  

**Structure Ed25519SignatureOffsets :**  
```rust
struct Ed25519SignatureOffsets {
    signature_offset: u16,             // offset de la signature ed25519 (64 octets)
    signature_instruction_index: u16,  // index de l'instruction pour trouver la signature
    public_key_offset: u16,            // offset de la clé publique (32 octets)
    public_key_instruction_index: u16, // index de l'instruction pour trouver la clé publique
    message_data_offset: u16,          // offset du début des données du message
    message_data_size: u16,            // taille des données du message
    message_instruction_index: u16,    // index de l'instruction pour trouver les données du message
}
```
**Logique de Vérification (Pseudo-code) :**  
Le programme vérifie chaque signature en récupérant la signature, la clé publique et le message à partir des offsets spécifiés dans les données d'instruction de la transaction. Si une vérification échoue, toute la transaction échoue.

##### **2. Programme Secp256k1**  
| Programme | Program ID | Description | Instructions |  
|-----------|------------|-------------|--------------|  
| **Secp256k1** | `KeccakSecp256k11111111111111111111111111111` | Vérifie les opérations de récupération de clé publique secp256k1 (`ecrecover`). | [Instructions](lien) |  

Ce programme traite une instruction dont le premier octet est le nombre de structures suivantes sérialisées dans les données d'instruction :  

**Structure Secp256k1SignatureOffsets :**  
```rust
struct Secp256k1SignatureOffsets {
    secp_signature_offset: u16,            // offset de [signature, recovery_id] (64+1 octets)
    secp_signature_instruction_index: u8,  // index de l'instruction pour trouver la signature
    secp_pubkey_offset: u16,               // offset de la pubkey ethereum_address (20 octets)
    secp_pubkey_instruction_index: u8,     // index de l'instruction pour trouver la pubkey
    secp_message_data_offset: u16,         // offset du début des données du message
    secp_message_data_size: u16,           // taille des données du message
    secp_message_instruction_index: u8,    // index de l'instruction pour trouver les données du message
}
```
**Logique de Vérification (Pseudo-code) :**  
Le programme récupère la signature, l'ID de récupération, la clé publique de référence et le hachage du message (Keccak256) depuis les données d'instruction. Il utilise `ecrecover` pour obtenir la clé publique à partir de la signature et du hachage du message, puis compare le résultat avec la clé publique de référence.

##### **3. Programme Secp256r1**  
| Programme | Program ID | Description | Instructions |  
|-----------|------------|-------------|--------------|  
| **Secp256r1** | `Secp256r1SigVerify1111111111111111111111111` | Vérifie jusqu'à 8 signatures secp256r1. Prend une signature, une clé publique et un message. Retourne une erreur si une échoue. | [Instructions](lien) |  

Ce programme traite une instruction. Le premier `u8` est le nombre de signatures à vérifier, suivi d'un octet de remplissage. Ensuite, la structure suivante est sérialisée, une pour chaque signature à vérifier :  

**Structure Secp256r1SignatureOffsets :**  
```rust
struct Secp256r1SignatureOffsets {
    signature_offset: u16,             // offset de la signature compacte secp256r1 (64 octets)
    signature_instruction_index: u16,  // index de l'instruction pour trouver la signature
    public_key_offset: u16,            // offset de la clé publique compressée (33 octets)
    public_key_instruction_index: u16, // index de l'instruction pour trouver la clé publique
    message_data_offset: u16,          // offset du début des données du message
    message_data_size: u16,            // taille des données du message
    message_instruction_index: u16,    // index de l'instruction pour trouver les données du message
}
```
**Logique de Vérification (Pseudo-code) :**  
Similaire aux autres, il vérifie chaque signature une par une. **Note :** Les valeurs 'S' basses sont imposées pour toutes les signatures pour éviter toute malléabilité accidentelle des signatures.

---

### **Programmes de Base (« Core Programs »)**  
Le démarrage (« genesis ») d'un cluster Solana inclut une liste de **programmes spéciaux** qui fournissent les fonctionnalités de base du réseau. Historiquement, ils étaient appelés programmes « natifs » et étaient distribués avec le code du validateur.  

| Programme | Program ID | Description | Instructions |  
|-----------|------------|-------------|--------------|  
| **System Program** | `11111111111111111111111111111111` | Crée de nouveaux comptes, alloue l'espace des comptes, attribue la propriété des comptes, transfert des lamports et paye les frais de transaction. | [SystemInstruction](lien) |  
| **Vote Program** | `Vote111111111111111111111111111111111111111` | Crée et gère les comptes qui suivent l'état des votes des validateurs et les récompenses. | [VoteInstruction](lien) |  
| **Stake Program** | `Stake11111111111111111111111111111111111111` | Crée et gère les comptes représentant les enjeux (« stake ») et les récompenses pour les délégations aux validateurs. | [StakeInstruction](lien) |  
| **Config Program** | `Config1111111111111111111111111111111111111` | Ajoute des données de configuration à la blockchain, suivies par la liste des clés publiques autorisées à les modifier. A une instruction implicite : "store". | [ConfigInstruction](lien) |  
| **Compute Budget Program** | `ComputeBudget111111111111111111111111111111` | Définit les limites et prix des unités de calcul pour les transactions, permettant aux utilisateurs de contrôler les ressources et les frais de priorisation. | [ComputeBudgetInstruction](lien) |  
| **Address Lookup Table Program** | `AddressLookupTab1e1111111111111111111111111` | Gère les tables de recherche d'adresses, qui permettent aux transactions de référencer plus de comptes que ne le permet normalement la liste de comptes d'une transaction. | [ProgramInstruction](lien) |  
| **ZK ElGamal Proof Program** | `ZkE1Gama1Proof11111111111111111111111111111` | Fournit la vérification de preuves zero-knowledge pour les données chiffrées ElGamal. | — |  

---

**Résumé :**  
- Les **programmes** sont le cœur des applications décentralisées sur Solana.  
- Ils peuvent être écrits en **Rust** natif ou avec le framework **Anchor**.  
- Les **programmes intégrés** fournissent les fonctionnalités essentielles du réseau (système, votes, enjeux, etc.).  
- Les **programmes précompilés** (Ed25519, Secp256k1, Secp256r1) offrent une vérification cryptographique efficace et sécurisée directement sur la blockchain.  
- La **vérifiabilité** assure la confiance dans le code déployé.
