# Server Migration Guide for a Shido Validator

Migrating your Shido validator to a new server requires careful execution to prevent issues such as double signing. This guide provides streamlined steps to ensure a smooth and secure migration process. Follow each step diligently to maintain the integrity and performance of your validator.

## Prerequisites

Before initiating the migration, ensure you have the following:

- **Access to Both Servers:** You should have SSH or physical access to both the old and new servers.
- **Shido Node Setup on New Server:** The new server should have the Shido node installed and configured as per the [Shido Node Setup Guide](https://github.com/MavNode/shidovalidator/blob/main/shidonodesetup.md)
- **Secure Storage:** Safeguard your mnemonic phrases and private keys during the migration process.

## Migration Steps

### 1. Prepare the New Server

Begin by setting up your new server following the [Shido Node Setup Guide](https://github.com/MavNode/shidovalidator/blob/main/shidonodesetup.md). Ensure that the node is fully synchronized and running correctly before proceeding.

### 2. Backup and Secure the Old Server

On **Server 1 (Old Server)**, perform the following actions:

#### a. Backup the Private Validator Key JSON

Create a secure backup of your `priv_validator_key.json`:
```
cp ~/.shidod/config/priv_validator_key.json ~/priv_validator_key.json.backup
```
> [!IMPORTANT]  
> Ensure this backup is stored in a safe location, preferably offline or on a secure external storage device.

#### b. Backup Your Mnemonic and Validator Keys
Ensure you have a secure backup of your mnemonic phrase and any additional keys required for validator signing. Store these backups in a secure location, preferably offline.

### 3. Update the Old Server
> [!CAUTION]
> To prevent double signing, you need to remove the private validator key from the old server.

#### a. Remove the Private Validator Key JSON
Delete the priv_validator_key.json file from the old server:
```
rm ~/.shidod/config/priv_validator_key.json
```
#### b. Stop the Shido Node
```
sudo systemctl stop shidod
```
#### c. Disable the Shido Node Service
```
sudo systemctl disable shidod
```
> [!NOTE]  
> Disabling the service ensures it does not start automatically in the future, providing an extra layer of security against accidental double signing.

### 4. Configure the New Server
On Server 2 (New Server), execute the following steps:

#### a. Stop the Shido Service
```
sudo systemctl stop shidod
```
#### b. Restore the Private Validator Key JSON
Open the priv_validator_key.json file on the new server and replace its content with your backed-up key:
```
nano ~/.shidod/config/priv_validator_key.json
```
##### Inside the Editor:
- Delete all existing content.
- Paste the contents from your priv_validator_key.json.backup.
- Save and exit (CTRL + O, then CTRL + X in Nano).

#### c. Recover Your Keys Using the Mnemonic
```
shidod keys add your_key_name --recover
```
When prompted, enter your mnemonic phrase to recover your validator keys.

### 5. Finalize the Migration

#### a. Restart the Shido Service
Start the Shido service to apply the changes:
```
sudo systemctl restart shidod
```

#### b. Check the Status of the Shido Service
Verify that the service is active and running without errors:
```
sudo systemctl status shidod
```
You should see an output indicating that the service is active (running).

#### c. Monitor the Logs
To ensure that your node is operating correctly, monitor the logs:
```
sudo journalctl -u shidod -f --no-hostname -o cat
```
This command will display real-time logs, allowing you to verify that the node is functioning as expected.

### Important Notes
- Avoid Double Signing:
  By removing the priv_validator_key.json from the old server, you prevent the validator from signing blocks on both servers simultaneously. This is crucial to avoid penalties associated with double signing.
- Secure Your Keys:
  Always handle your private keys and mnemonic phrases with the utmost security. Exposure of these can compromise your validator and funds.
- Verify Synchronization:
  After migration, confirm that the new node is fully synchronized with the network to maintain validator performance and reliability.
- Backup Regularly:
  Regularly back up your keys and configurations to prevent data loss in the future.

By following this guide meticulously, you can successfully migrate your Shido validator to a new server without encountering double signing issues.
If you experience any challenges or have further questions, please reach out through the repository for additional assistance with the migration process, feel free to ask!
