# Développement de Programmes Solana en Rust

Les programmes Solana sont principalement développés en utilisant le langage de programmation Rust. Cette page se concentre sur l'écriture de programmes Solana en Rust sans utiliser le framework Anchor, une approche souvent appelée développement en "Rust natif".

Le développement en Rust natif offre aux développeurs un contrôle direct sur leurs programmes Solana. Cependant, cette approche nécessite plus de configuration manuelle et de code boilerplate par rapport à l'utilisation du framework Anchor. Cette méthode est recommandée pour les développeurs qui :

- Recherchent un contrôle granulaire sur la logique du programme et les optimisations
- Souhaitent apprendre les concepts sous-jacents avant de passer à des frameworks de plus haut niveau

Pour les débutants, nous recommandons de commencer avec le framework Anchor. Voir la section Anchor pour plus d'informations.

## Prérequis

Pour des instructions d'installation détaillées, visitez la page d'installation.

Avant de commencer, assurez-vous d'avoir installé les éléments suivants :

- **Rust** : Le langage de programmation pour construire des programmes Solana.
- **Solana CLI** : L'outil en ligne de commande pour le développement Solana.

## Pour Commencer

L'exemple ci-dessous couvre les étapes de base pour créer votre premier programme Solana écrit en Rust. Nous allons créer un programme minimal qui affiche "Hello, world!" dans le journal du programme.

### Créer un nouveau programme

Commencez par créer un nouveau projet Rust en utilisant la commande standard `cargo new` avec le flag `--lib`.

**Terminal**
```bash
cargo new hello_world --lib
```

Naviguez vers le répertoire du projet. Vous devriez voir les fichiers par défaut `src/lib.rs` et `Cargo.toml`.

**Terminal**
```bash
cd hello_world
```

Mettez à jour le champ `edition` dans `Cargo.toml` vers `2021`. Sinon, vous pourriez rencontrer une erreur lors de la construction du programme.

### Ajouter la dépendance `solana-program`

Ensuite, ajoutez la dépendance `solana-program`. C'est la dépendance minimale requise pour construire un programme Solana.

**Terminal**
```bash
cargo add solana-program@=1.18.0 # Note: Version ajustée pour cohérence
```

### Ajouter le `crate-type`

Ensuite, ajoutez l'extrait suivant à `Cargo.toml`.

**Cargo.toml**
```toml
[lib]
crate-type = ["cdylib", "lib"]
```

Si vous n'incluez pas cette configuration, le répertoire `target/deploy` ne sera pas généré lorsque vous construirez le programme.

### Ajouter le code du programme

Ensuite, remplacez le contenu de `src/lib.rs` par le code suivant. Il s'agit d'un programme Solana minimal qui affiche "Hello, world!" dans le journal du programme lorsqu'il est invoqué.

La macro `msg!` est utilisée dans les programmes Solana pour afficher un message dans le journal du programme.

**src/lib.rs**
```rust
use solana_program::{
    account_info::AccountInfo,
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    pubkey::Pubkey,
};

entrypoint!(process_instruction);

pub fn process_instruction(
    _program_id: &Pubkey,
    _accounts: &[AccountInfo],
    _instruction_data: &[u8],
) -> ProgramResult {
    msg!("Hello, world!");
    Ok(())
}
```

### Construire le programme

Ensuite, construisez le programme en utilisant la commande `cargo build-sbf`.

**Terminal**
```bash
cargo build-sbf
```

Cette commande génère un répertoire `target/deploy` contenant deux fichiers importants :

- Un fichier **.so** (ex: `hello_world.so`) : C'est le programme Solana compilé qui sera déployé sur le réseau en tant que "smart contract".
- Un fichier **keypair** (ex: `hello_world-keypair.json`) : La clé publique de cette keypair est utilisée comme l'identifiant du programme (Program ID) lors du déploiement.

Pour voir l'identifiant du programme, exécutez la commande suivante dans votre terminal. Cette commande affiche la clé publique de la keypair au chemin de fichier spécifié :

**Terminal**
```bash
solana address -k ./target/deploy/hello_world-keypair.json
```

**Exemple de sortie :**
```
4Ujf5fXfLx2PAwRqcECCLtgDxHKPznoJpa43jUBxFfMz
```

### Ajouter les dépendances de test

Ensuite, testez le programme en utilisant la crate `solana-program-test`. Ajoutez les dépendances suivantes à `Cargo.toml`.

**Terminal**
```bash
cargo add solana-program-test --dev
cargo add solana-sdk --dev
```

### Tester le programme

Ajoutez le test suivant à `src/lib.rs`, en dessous du code du programme. Il s'agit d'un module de test qui invoque le programme hello world.

**src/lib.rs (ajout)**
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use solana_program_test::*;
    use solana_sdk::{transaction::Transaction, signature::{Signer, Keypair}, instruction::{Instruction, AccountMeta}};

    #[tokio::test]
    async fn test_hello_world() {
        let program_id = Pubkey::new_unique();
        let (mut banks_client, payer, recent_blockhash) = ProgramTest::new(
            "hello_world",
            program_id,
            processor!(process_instruction), // Utilise le point d'entrée
        ).start().await;

        let mut transaction = Transaction::new_with_payer(
            &[Instruction {
                program_id,
                accounts: vec![],
                data: vec![],
            }],
            Some(&payer.pubkey()),
        );
        transaction.sign(&[&payer], recent_blockhash);

        banks_client.process_transaction(transaction).await.unwrap();
        // Le test réussit si la transaction ne panique pas
    }
}
```

Exécutez le test en utilisant la commande `cargo test`. Le journal du programme affichera "Hello, world!".

**Terminal**
```bash
cargo test -- --nocapture
```

**Exemple de sortie :**

**Terminal**
```
running 1 test
Logs: [
    "Program 9528phXNvdWp5kkR4rgpoeZvR8ZWT5THVywK95YRprkk invoke [1]",
    "Program log: Hello, world!",
    "Program 9528phXNvdWp5kkR4rgpoeZvR8ZWT5THVywK95YRprkk consumed 211 of 200000 compute units",
    "Program 9528phXNvdWp5kkR4rgpoeZvR8ZWT5THVywK95YRprkk success",
]
test tests::test_hello_world ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.05s
```

### Déployer le programme

Ensuite, déployez le programme. Lors du développement local, nous pouvons utiliser le `solana-test-validator`.

Configurez d'abord le CLI Solana pour utiliser le cluster Solana local.

**Terminal**
```bash
solana config set --url localhost
```

**Exemple de sortie :**
```
Config File: /home/user/.config/solana/cli/config.yml
RPC URL: http://localhost:8899
WebSocket URL: ws://localhost:8900/ (computed)
Keypair Path: /home/user/.config/solana/id.json
Commitment: confirmed
```

Ouvrez un nouveau terminal et exécutez la commande `solana-test-validator` pour démarrer le validateur local.

**Terminal (Nouvelle fenêtre)**
```bash
solana-test-validator
```

Pendant que le validateur de test fonctionne, exécutez la commande `solana program deploy` dans un terminal séparé pour déployer le programme sur le validateur local.

**Terminal**
```bash
solana program deploy ./target/deploy/hello_world.so
```

**Exemple de sortie :**
```
Program Id: 4Ujf5fXfLx2PAwRqcECCLtgDxHKPznoJpa43jUBxFfMz
Signature: 5osMiNMiDZGM7L1e2tPHxU8wdB8gwG8fDnXLg5G7SbhwFz4dHshYgAijk4wSQL5cXiu8z1MMou5kLadAQuHp7ybH
```

Vous pouvez inspecter l'identifiant du programme et la signature de la transaction sur Solana Explorer.

Notez que le cluster sur Solana Explorer doit également être `localhost`. L'option "Custom RPC URL" sur Solana Explorer par défaut est `http://localhost:8899`.

### Créer un exemple de client

Ensuite, nous allons démontrer comment invoquer le programme en utilisant un client Rust.

Créez d'abord un répertoire `examples` et un fichier `client.rs`.

**Terminal**
```bash
mkdir -p examples
touch examples/client.rs
```

Ajoutez le code suivant à `examples/client.rs`. Il s'agit d'un script client Rust qui finance une nouvelle keypair pour payer les frais de transaction puis invoque le programme hello world.

**examples/client.rs**
```rust
use solana_client::rpc_client::RpcClient;
use solana_program::{pubkey::Pubkey, system_instruction};
use solana_sdk::{
    commitment_config::CommitmentConfig,
    signature::{Keypair, Signer},
    transaction::Transaction,
};
use std::str::FromStr;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let rpc_url = "http://localhost:8899".to_string();
    let connection = RpcClient::new_with_commitment(rpc_url, CommitmentConfig::confirmed());

    let payer = Keypair::new();
    let program_id = Pubkey::from_str("4Ujf5fXfLx2PAwRqcECCLtgDxHKPznoJpa43jUBxFfMz")?; // REMPLACER

    // Demander un airdrop pour le payer
    match connection.request_airdrop(&payer.pubkey(), 1_000_000_000) {
        Ok(sig) => {
            println!("Airdrop demandé. Signature: {}", sig);
            loop {
                let confirmed = connection.confirm_transaction(&sig)?;
                if confirmed {
                    println!("Airdrop confirmé!");
                    break;
                }
            }
        }
        Err(e) => println!("Erreur d'airdrop: {}", e),
    }

    // Créer une instruction pour invoquer le programme
    let instruction = solana_sdk::instruction::Instruction {
        program_id,
        accounts: vec![],
        data: vec![],
    };

    let recent_blockhash = connection.get_latest_blockhash()?;
    let transaction = Transaction::new_signed_with_payer(
        &[instruction],
        Some(&payer.pubkey()),
        &[&payer],
        recent_blockhash,
    );

    // Envoyer la transaction
    let signature = connection.send_and_confirm_transaction(&transaction)?;
    println!("Signature de la transaction: {}", signature);
    println!("Transaction soumise. Vérifiez les logs sur Solana Explorer.");

    Ok(())
}
```
*(Note: La version du client a été simplifiée et adaptée pour la clarté et la compatibilité.)*

**Avant d'exécuter le code client, remplacez l'identifiant du programme dans l'extrait de code par le vôtre.**

Vous pouvez obtenir votre identifiant de programme en exécutant la commande suivante.

**Terminal**
```bash
solana address -k ./target/deploy/hello_world-keypair.json
```

### Invoquer le programme

Exécutez le script client avec la commande suivante.

**Terminal**
```bash
cargo run --example client
```

**Exemple de sortie :**
```
Transaction Signature: 54TWxKi3Jsi3UTeZbhLGUFX6JQH7TspRJjRRFZ8NFnwG5BXM9udxiX77bAACjKAS9fGnVeEazrXL4SfKrW7xZFYV
```

Vous pouvez inspecter la signature de la transaction sur Solana Explorer (cluster local) pour voir "Hello, world!" dans le journal du programme.

---
