### Passi per Installare e Configurare `batman-adv` sulla Prima Raspberry Pi

#### 1. Preparare l'Ambiente

1. **Aggiorna il Sistema Operativo:**
   Assicurati che il sistema operativo sia aggiornato.
   ```sh
   sudo apt update
   sudo apt upgrade -y
   ```

2. **Installa le Dipendenze:**
   Installa `batctl` e abilita `batman-adv` come negli altri dispositivi.
   ```sh
   sudo apt install batctl -y
   ```

#### 2. Attiva `batman-adv`

1. **Carica il Modulo del Kernel `batman-adv`:**
   ```sh
   sudo modprobe batman-adv
   ```

2. **Rendi Permanente il Caricamento del Modulo:**
   Aggiungi `batman-adv` al file `/etc/modules` per caricarlo automaticamente all'avvio.
   ```sh
   echo "batman-adv" | sudo tee -a /etc/modules
   ```

#### 3. Configurare l'Interfaccia Ethernet e Wireless

1. **Configura l'Interfaccia Ethernet:**
   Assicurati che l'interfaccia Ethernet (ad esempio `eth0`) sia configurata per ottenere un indirizzo IP tramite DHCP o con un IP statico, in base alla tua rete.

2. **Configura l'Interfaccia Wi-Fi per la Rete Mesh:**
   Configura l'interfaccia Wi-Fi (`wlan0`) in modalità mesh (802.11s) o ad-hoc per la connessione con le altre Raspberry Pi.

   **Modalità Ad-Hoc:**
   ```sh
   sudo ip link set wlan0 down
   sudo iwconfig wlan0 mode ad-hoc
   sudo iwconfig wlan0 essid "mesh-network"
   sudo iwconfig wlan0 ap any
   sudo iwconfig wlan0 channel 1
   sudo ip link set wlan0 up
   ```
Se vi esce `Error for wireless request "Set Mode" (8B06) : SET failed on device wlan0 ; Device or resource busy.` seguite la procedura nella sezione **troubleshooting**.

3. **Aggiungi l'Interfaccia Wi-Fi a `batman-adv`:**
   ```sh
   sudo batctl if add wlan0
   sudo ip link set up dev bat0
   ```

4. **Configura l'Indirizzo IP sull'Interfaccia `bat0`:**
   Assegna un indirizzo IP statico all'interfaccia `bat0` nella stessa sottorete utilizzata dalle altre Raspberry Pi nella rete mesh.
   ```sh
   sudo ip addr add 192.168.10.1/24 dev bat0  # Modifica l'IP in base alla tua rete
   ```

#### 4. Configurare la Condivisione della Connessione Internet

1. **Abilita il Forwarding IP:**
   Modifica il file `/etc/sysctl.conf` per abilitare il forwarding IP.
   ```sh
   sudo nano /etc/sysctl.conf
   ```
   Trova la linea:
   ```sh
   #net.ipv4.ip_forward=1
   ```
   Rimuovi il `#` per abilitare l'opzione:
   ```sh
   net.ipv4.ip_forward=1
   ```
   Salva il file e applica le modifiche:
   ```sh
   sudo sysctl -p
   ```

2. **Imposta le Regole di iptables per il NAT:**
   Configura `iptables` per permettere la condivisione della connessione Internet tra l'interfaccia Ethernet (che ha accesso a Internet) e l'interfaccia mesh.
   ```sh
   sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
   ```
   Se vi esce `sudo: iptables: command not found` seguite le istruzioni nella sezione **troubleshooting**.

3. **Salva le Regole di iptables:**
   Salva le regole in modo che siano caricate automaticamente all'avvio.
   ```sh
   sudo sh -c "iptables-save > /etc/iptables/rules.v4"
   ```
   Potrebbe succedere che vi esca l`errore `sh: 1: cannot create /etc/iptables/rules.v4: Directory nonexistent` questo perché le cartelle di default non vengono create correttamente, seguite la sezione **troubleshooting**.

#### 5. Verifica la Configurazione

Per questa sezione potrebbe essere necessario farla in un secondo momento, quando saranno installate anche le raspberry della rete mesh.

1. **Controlla la Configurazione di `batman-adv`:**
   Usa `batctl` per assicurarti che l'interfaccia mesh sia correttamente configurata.
   ```sh
   sudo batctl n  # Mostra i vicini nella rete mesh
   sudo batctl o  # Mostra la tabella di routing
   ```

2. **Testa la Connettività alla Rete Mesh:**
   Verifica che la prima Raspberry Pi possa comunicare con le altre Raspberry Pi nella rete mesh.
   ```sh
   ping 192.168.10.2  # Sostituisci con l'indirizzo IP di un'altra Raspberry Pi
   ```

3. **Testa la Condivisione della Connessione Internet:**
   Controlla che le altre Raspberry Pi possano accedere a Internet tramite la prima Raspberry Pi.
   ```sh
   ping google.com
   ```

# Troubleshoot
## Error for wireless request "Set Mode" (8B06) : SET failed on device wlan0 ; Device or resource busy.

L'errore "Device or resource busy" durante l'esecuzione del comando `sudo iwconfig wlan0 mode ad-hoc` significa che l'interfaccia wireless (`wlan0`) è attualmente in uso o gestita da un'altra applicazione o servizio, come il gestore di rete (`NetworkManager`) o `wpa_supplicant`.

Per risolvere questo problema, puoi seguire questi passaggi:

### 1. Disattivare il Gestore di Rete
Se stai usando un gestore di rete come `NetworkManager`, devi disabilitarlo temporaneamente per configurare manualmente la tua rete mesh.
```sh
sudo systemctl stop NetworkManager
```
Oppure, se stai usando `dhcpcd`:
```sh
sudo systemctl stop dhcpcd
```
### 2. Disconnettere `wpa_supplicant`
`wpa_supplicant` potrebbe gestire l'interfaccia Wi-Fi. Fermalo prima di cambiare la modalità dell'interfaccia:
```sh
sudo systemctl stop wpa_supplicant
```
### 3. Disattivare l'Interfaccia Wireless
Assicurati che l'interfaccia wireless (`wlan0`) sia disattivata prima di cambiarne la modalità:
```sh
sudo ip link set wlan0 down
```
### 4. Cambiare Modalità a Ad-Hoc
Ora prova di nuovo a impostare la modalità ad-hoc:
```sh
sudo iwconfig wlan0 mode ad-hoc
```
### 5. Configurare l'Interfaccia
Se il comando funziona, continua con la configurazione dell'interfaccia:
```sh
sudo iwconfig wlan0 essid "mesh-network"
sudo iwconfig wlan0 ap any
sudo iwconfig wlan0 channel 1
```
Infine, riattiva l'interfaccia:
```sh
sudo ip link set wlan0 up
```
### 6. Riavviare i Servizi di Rete (Facoltativo)
Se vuoi utilizzare nuovamente il gestore di rete o `wpa_supplicant` dopo aver configurato la rete mesh, puoi riavviarli:
```sh
sudo systemctl start NetworkManager
```
Oppure:
```sh
sudo systemctl start dhcpcd
```

## sudo: iptables: command not found
L'errore "sudo: iptables: command not found" indica che `iptables`, il tool per la gestione delle tabelle di filtraggio dei pacchetti IP in Linux, non è installato sulla tua Raspberry Pi.

Ecco come installare `iptables`:

### Passaggi per Installare `iptables`

1. **Aggiorna il Gestore dei Pacchetti:**
   Prima di installare `iptables`, assicurati che l'elenco dei pacchetti sia aggiornato.
   ```sh
   sudo apt update
   ```

2. **Installa `iptables`:**
   Installa `iptables` utilizzando il comando seguente:
   ```sh
   sudo apt install iptables -y
   ```

3. **Verifica l'Installazione:**
   Una volta completata l'installazione, verifica che `iptables` sia stato installato correttamente.
   ```sh
   iptables --version
   ```

Se il comando restituisce la versione di `iptables`, significa che è stato installato con successo.

### Continuare la Configurazione

Dopo aver installato `iptables`, puoi ripetere i comandi necessari per configurare il NAT (Network Address Translation) e la condivisione della connessione Internet:

```sh
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i bat0 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o bat0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Questi comandi configurano `iptables` per permettere la condivisione della connessione Internet tra l'interfaccia Ethernet (`eth0`) e l'interfaccia mesh (`bat0`).

### Salva le Regole di `iptables`

Infine, salva le regole per fare in modo che vengano applicate automaticamente all'avvio:

```sh
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

Ora, la tua Raspberry Pi dovrebbe essere configurata correttamente per condividere la connessione Internet con la rete mesh.

## sh: 1: cannot create /etc/iptables/rules.v4: Directory nonexistent

L'errore "sh: 1: cannot create /etc/iptables/rules.v4: Directory nonexistent" indica che la directory `/etc/iptables/` non esiste sul tuo sistema. Questo potrebbe accadere se `iptables` non è stato completamente installato o se le directory di configurazione predefinite non sono state create.

Ecco come risolvere questo problema:

### Creare la Directory Mancante

1. **Crea la Directory per le Regole di `iptables`:**

   Crea manualmente la directory necessaria per salvare le regole di `iptables`:

   ```sh
   sudo mkdir -p /etc/iptables
   ```

2. **Salva le Regole di `iptables`:**

   Dopo aver creato la directory, salva le regole di `iptables` nel file `rules.v4`:

   ```sh
   sudo iptables-save | sudo tee /etc/iptables/rules.v4
   ```

### Assicurare che le Regole siano Applicate all'Avvio

Per fare in modo che le regole di `iptables` siano applicate automaticamente all'avvio, puoi usare `iptables-persistent`, un pacchetto che carica automaticamente le regole di `iptables` durante l'avvio del sistema.

3. **Installa `iptables-persistent`:**

   ```sh
   sudo apt install iptables-persistent -y
   ```

Durante l'installazione, ti verrà chiesto di salvare le regole correnti di `iptables`. Conferma selezionando "Yes".

4. **Verifica l'Installazione di `iptables-persistent`:**

   Puoi verificare che `iptables-persistent` sia configurato correttamente controllando lo stato del servizio:

   ```sh
   sudo systemctl status netfilter-persistent
   ```

5. **Ricarica le Regole di `iptables`:**

   Se necessario, ricarica le regole di `iptables` manualmente:

   ```sh
   sudo netfilter-persistent reload
   ```

### Conclusione

Dopo aver completato questi passaggi, la tua Raspberry Pi sarà configurata per applicare automaticamente le regole di `iptables` all'avvio. Ora la condivisione della connessione Internet e la configurazione della rete mesh dovrebbero funzionare correttamente.
