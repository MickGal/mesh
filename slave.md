### Passaggi per Configurare le Raspberry Pi sulla Rete Mesh

#### 1. Configurare la Seconda e Terza Raspberry Pi

Per la seconda e terza Raspberry Pi, devi configurarle per partecipare alla rete mesh e accedere a Internet tramite la prima Raspberry Pi (gateway).

Ripeti i seguenti passaggi su entrambe le Raspberry Pi (seconda e terza):

1. **Installa `batman-adv` e `batctl`:**

   ```sh
   sudo apt update
   sudo apt upgrade -y
   sudo apt install batctl -y
   ```

2. **Carica il Modulo del Kernel `batman-adv`:**

   ```sh
   sudo modprobe batman-adv
   ```

3. **Rendi Permanente il Caricamento del Modulo:**

   ```sh
   echo "batman-adv" | sudo tee -a /etc/modules
   ```

4. **Configura l'Interfaccia Wireless per la Rete Mesh:**

   - **Modalità Mesh (802.11s):**
     ```sh
     sudo ip link set wlan0 down
     sudo iw dev wlan0 set type mp
     sudo ip link set wlan0 up
     sudo iw dev wlan0 mesh join "mesh-network" freq 2412

	 ```
	 In caso di errore `sudo iw dev wlan0 set type mp command failed: Operation not supported (-95)` vedere il **troubleshooting**.

5. **Aggiungi l'Interfaccia Wireless a `batman-adv`:**

   ```sh
   sudo batctl if add wlan0
   sudo ip link set up dev bat0
   ```

6. **Configura un Indirizzo IP Statico sull'Interfaccia `bat0`:**

   Assegna un indirizzo IP statico diverso per ciascuna Raspberry Pi, ma all'interno della stessa sottorete:

   - **Seconda Raspberry Pi:**
     ```sh
     sudo ip addr add 192.168.10.2/24 dev bat0
     ```

   - **Terza Raspberry Pi:**
     ```sh
     sudo ip addr add 192.168.10.3/24 dev bat0
     ```

#### 2. Verificare la Configurazione della Rete Mesh

1. **Controlla lo Stato della Rete Mesh su Tutte le Raspberry Pi:**

   Utilizza `batctl` per verificare che le Raspberry Pi siano collegate correttamente alla rete mesh:

   ```sh
   sudo batctl n  # Mostra i vicini della rete mesh
   sudo batctl o  # Mostra la tabella di routing
   ```

   Dovresti vedere le altre Raspberry Pi come vicine nella rete mesh.

2. **Aggiungere gateway**
```
sudo route add default gw 192.168.100.1 #Sostituire con l`ip del gateway
```
3. **Testa la Connettività di Rete:**

   Esegui un ping da ciascuna Raspberry Pi alle altre per assicurarti che la connettività sia stabilita:

   - Dalla seconda Raspberry Pi:
     ```sh
     ping 192.168.10.1  # Ping alla prima Raspberry Pi (gateway)
     ping 192.168.10.3  # Ping alla terza Raspberry Pi
     ```

   - Dalla terza Raspberry Pi:
     ```sh
     ping 192.168.10.1  # Ping alla prima Raspberry Pi (gateway)
     ping 192.168.10.2  # Ping alla seconda Raspberry Pi
     ```

4. **Testa l'Accesso a Internet:**

   Verifica che la seconda e la terza Raspberry Pi possano accedere a Internet tramite la prima Raspberry Pi:

   ```sh
   ping google.com
   ```

# Troubleshooting

## sudo iw dev wlan0 set type mp command failed: Operation not supported (-95)
L'errore "Operation not supported (-95)" indica che il chipset Wi-Fi o il driver della tua Raspberry Pi non supporta la modalità **mesh (802.11s)**. Non tutti i chipset Wi-Fi sono compatibili con la modalità mesh, e questo può variare in base al driver del dispositivo o alla versione del kernel Linux.

Tuttavia, puoi ancora configurare una rete mesh utilizzando la modalità **ad-hoc**. La modalità ad-hoc è supportata da un numero maggiore di chipset e potrebbe essere una soluzione alternativa per creare una rete mesh tra le tue Raspberry Pi.

### Passaggi per Configurare la Rete Mesh in Modalità Ad-Hoc

1. **Disabilita l'Interfaccia Wireless:**

   Spegni l'interfaccia wireless (`wlan0`) prima di modificarne la configurazione:

   ```sh
   sudo ip link set wlan0 down
   ```

2. **Configura l'Interfaccia Wireless in Modalità Ad-Hoc:**

   Utilizza `iwconfig` per impostare l'interfaccia in modalità ad-hoc:

   ```sh
   sudo iwconfig wlan0 mode ad-hoc
   sudo iwconfig wlan0 essid "mesh-network"
   sudo iwconfig wlan0 channel 1
   sudo ip link set wlan0 up
   ```

   - **`essid`** è il nome della tua rete mesh; tutte le Raspberry Pi dovrebbero utilizzare lo stesso `essid`.
   - **`channel`** dovrebbe essere lo stesso su tutte le Raspberry Pi per garantire che possano comunicare tra loro.

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

## Se il comando `iwconfig wlan0` mostra `ESSID: off/any`, significa che l'interfaccia wireless (`wlan0`) non è connessa a nessuna rete e non è configurata correttamente per la modalità ad-hoc. Questo può accadere per diversi motivi, tra cui configurazioni errate, servizi di rete che interferiscono o problemi hardware.

### Passi per Risolvere il Problema

#### 1. Disattivare Servizi di Rete Che Potrebbero Interferire

Servizi come `NetworkManager` o `wpa_supplicant` possono interferire con la configurazione manuale della rete. Assicurati che questi servizi siano disattivati:

```sh
sudo systemctl stop NetworkManager
sudo systemctl stop wpa_supplicant
```

#### 2. Disattivare e Riattivare l'Interfaccia Wireless

Per configurare correttamente l'interfaccia wireless, è importante iniziare con l'interfaccia `wlan0` disattivata:

```sh
sudo ip link set wlan0 down
```

#### 3. Configurare l'Interfaccia Wireless in Modalità Ad-Hoc

Ora, configura l'interfaccia wireless in modalità ad-hoc con il seguente comando:

```sh
sudo iwconfig wlan0 mode ad-hoc
sudo iwconfig wlan0 essid "mesh-network"
sudo iwconfig wlan0 channel 1  # Assicurati che tutte le Raspberry Pi siano sullo stesso canale
```

- **`mode ad-hoc`**: imposta la modalità ad-hoc.
- **`essid "mesh-network"`**: specifica il nome della rete ad-hoc.
- **`channel 1`**: seleziona il canale (tutte le Raspberry Pi devono utilizzare lo stesso canale).

#### 4. Riattivare l'Interfaccia Wireless

Riattiva l'interfaccia wireless:

```sh
sudo ip link set wlan0 up
```

#### 5. Verifica la Configurazione

Verifica nuovamente la configurazione:

```sh
iwconfig wlan0
```

Dovresti vedere un output simile a:

```
wlan0     IEEE 802.11  ESSID:"mesh-network"
          Mode:Ad-Hoc  Frequency:2.412 GHz  Cell: AA:BB:CC:DD:EE:FF
          ...
```

Se l'`ESSID` è impostato su "mesh-network" e la modalità è "Ad-Hoc", la configurazione è corretta.
