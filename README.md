# Upgrade-Pell
## Testnet Upgrade Failure Recovery Guide

Follow these steps to recover from the upgrade.

## 1.0 Enter the Docker Container

If the previous container is still running, stop it:

```bash
docker stop <your_container_name>
```

If the container has been stopped, start a new container with this command:

```bash
docker run --rm -it \
    -v <your_local_path>:/root/.pellcored \
    --entrypoint /bin/bash \
    docker.io/pellnetwork/pellnode:v1.0.23-ignite-testnet
```

---

## 2.0 Backup Data Directory

Create a backup of the data directory:

```bash
cp -r /root/.pellcored/data /root/.pellcored/data.bak
```

---

## 3.0 Roll Back Blocks

Roll back the blockchain to block height 95518, one block at a time:

```bash
pellcored rollback --hard
```

---

## 4.0 Download and Replace the Binary

Download the patched binary using the following steps:

- **Remove the old binary and download the new one:**

```bash
BINARY_URL=https://github.com/0xPellNetwork/network-config/releases/download/v1.0.0-ignite-186-genesis/pellcored-v1.0.0-linux-amd64
rm /root/.pellcored/cosmovisor/genesis/bin/pellcored
wget $BINARY_URL -O /root/.pellcored/cosmovisor/genesis/bin/pellcored
chmod +x /root/.pellcored/cosmovisor/genesis/bin/pellcored
```

The result should look like this:

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/db093c5c-b26b-4635-9271-f0a24b703057/b52cebfd-a125-4003-a2bf-dff3335d4f96/image.png)

- **Copy the updated binary to the upgrade directory:**

```bash
UPGRADE_NAME=v1.0.20  # Upgrade name matching the governance proposal
BINARY_URL=https://github.com/0xPellNetwork/network-config/releases/download/${UPGRADE_NAME}-ignite/pellcored-${UPGRADE_NAME}-linux-amd64

mkdir -p /root/.pellcored/cosmovisor/upgrades/$UPGRADE_NAME/bin
wget $BINARY_URL -O /root/.pellcored/cosmovisor/upgrades/$UPGRADE_NAME/bin/pellcored
chmod +x /root/.pellcored/cosmovisor/upgrades/$UPGRADE_NAME/bin/pellcored
```

---

## 5.0 Restart `pellcored`

**Option 1: Restart the existing container:**

```bash
docker start <your_container_name>
```

**Option 2: Start a new container:**

```bash
docker run -d --name=pell-validator \
    -v <your_local_path>:/root/.pellcored \
    -e MONIKER="<your_node_name>" \
    -p 26656:26656 \
    -p 26660:26660 \
    --entrypoint /usr/local/bin/start-pellcored.sh \
    docker.io/pellnetwork/pellnode:v1.0.23-ignite-testnet
```
