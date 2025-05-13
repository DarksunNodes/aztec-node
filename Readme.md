
# Aztec Network Sequencer Node

A step-by-step guide to run a **Sequencer Node** on the Aztec Network testnet and earn the `Apprentice` role.

---

## 🌐 Node Types

| Node Type   | Description                                                                                   |
|-------------|-----------------------------------------------------------------------------------------------|
| `Sequencer` | Proposes and validates blocks, votes on upgrades.                                             |
| `Prover`    | Generates ZK proofs that attest to roll-up integrity. *(Prover nodes are for large infra setups, not home use.)* |

---

## 🛡️ Discord Role

Running a **Sequencer node** and syncing it gives you the **Apprentice role** in Discord.

👉 [Get Role Instructions](https://github.com/0xmoei/aztec-network/blob/main/README.md#get-apprentice-discord-role)

---

## 📦 Hardware Requirements

| RAM       | CPU Cores | SSD        |
|-----------|-----------|------------|
| 8–16 GB   | 4–9 Cores | 100+ GB SSD |

> **Prover Nodes**: Need ~40x machines with 16 cores and 128 GB RAM. Skip if you're not running data center infrastructure.

---

## 🧰 For Windows Users

Install Ubuntu using this [guide](https://github.com/0xmoei/Install-Linux-on-Windows).

## 🌐 For VPS Users

Get started on a VPS with **4 vCores + 8 GB RAM**  
👉 [Purchase VPS](https://my.hostbrr.com/order/forms/a/NTMxNW==)


---

## 1. Install Dependencies

```bash
sudo apt-get update && sudo apt-get upgrade -y

sudo apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip -y
````

### Docker Installation

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo $VERSION_CODENAME) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo systemctl enable docker
sudo systemctl restart docker

# Test
sudo docker run hello-world
```

---

## 2. Install Aztec CLI

```bash
bash -i <(curl -s https://install.aztec.network)
echo 'export PATH="$HOME/.aztec/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
aztec
```

---

## 3. Update to Alpha Testnet

```bash
aztec-up alpha-testnet
```

---

## 4. Get RPC & BEACON URLs

* **RPC URL**: [Alchemy](https://dashboard.alchemy.com/)
* **Beacon URL**: [drpc](https://drpc.org/)
* Optional: Self-host Geth + Prysm → [Guide](https://github.com/0xmoei/geth-prysm-node)

---

## 5. Ethereum Wallet Keys

* Create/fetch an **EVM Wallet**
* Save both:

  * `Private Key`
  * `Public Address`

---

## 6. Get Sepolia ETH

* Use a Sepolia faucet to fund your wallet.

---

## 7. Get Your IP

```bash
curl ipv4.icanhazip.com
```

---

## 8. Enable Firewall and Open Ports

```bash
ufw allow 22
ufw allow ssh
ufw allow 40400
ufw allow 8080
ufw enable
```

---

## 9. Run Sequencer Node

```bash
screen -S aztec
```

```bash
aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls <RPC_URL> \
  --l1-consensus-host-urls <BEACON_URL> \
  --sequencer.validatorPrivateKey 0x<YOUR_PRIVATE_KEY> \
  --sequencer.coinbase 0x<YOUR_PUBLIC_ADDRESS> \
  --p2p.p2pIp <YOUR_IP> \
  --p2p.maxTxPoolSize 1000000000
```

**Replace:**

* `<RPC_URL>` & `<BEACON_URL>` = from Step 4
* `<YOUR_PRIVATE_KEY>` = from Step 5
* `<YOUR_PUBLIC_ADDRESS>` = from Step 5
* `<YOUR_IP>` = from Step 7

---

## 🔄 Sync Status

```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number"
```

Check latest Aztec block: [https://aztecscan.xyz/](https://aztecscan.xyz/)

---

## 11. Register Validator

```bash
aztec add-l1-validator \
  --l1-rpc-urls <RPC_URL> \
  --private-key 0x<YOUR_PRIVATE_KEY> \
  --attester 0x<YOUR_PUBLIC_ADDRESS> \
  --proposer-eoa 0x<YOUR_PUBLIC_ADDRESS> \
  --staking-asset-handler 0xF739D03e98e23A7B65940848aBA8921fF3bAc4b2 \
  --l1-chain-id 11155111
```

> ⚠️ Only 5 validators allowed per day. Retry next day if it fails.

---

## 12. Get Node Peer ID

```bash
sudo docker logs $(docker ps -q --filter ancestor=aztecprotocol/aztec:alpha-testnet | head -n 1) 2>&1 | grep -i "peerId" | grep -o '"peerId":"[^"]*"' | cut -d'"' -f4 | head -n 1
```

* Then search it on: [https://aztec.nethermind.io/](https://aztec.nethermind.io/)

---

## 🔃 Update Your Node

```bash
# Stop node
docker stop $(docker ps -q --filter "ancestor=aztecprotocol/aztec")
docker rm $(docker ps -a -q --filter "ancestor=aztecprotocol/aztec")

# Kill screens
screen -ls | grep -i aztec | awk '{print $1}' | xargs -I {} screen -X -S {} quit

# Update node
aztec-up alpha-testnet

# Delete old data
rm -rf ~/.aztec/alpha-testnet/data/
```

Then re-run the node by returning to **Step 9**.

---

## 🧠 Tips

**Screen Commands:**

* Detach: `Ctrl + A + D`
* Reattach: `screen -r aztec`
* Kill from inside: `Ctrl + C`
* Kill from outside: `screen -XS aztec quit`

---

## 🏁 Done!

You are now running an Aztec Sequencer Node. 🎉
Keep it running 24/7 to be eligible for the Apprentice role and future rewards!


