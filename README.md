# Nibiru-Mainnet

# system update

    sudo apt update
    sudo apt install -y curl git jq lz4 build-essential

# Install go

    sudo rm -rf /usr/local/go
    curl -L https://go.dev/dl/go1.21.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
    echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile
    source .bash_profile

# Clone repository        

    git clone https://github.com/NibiruChain/nibiru.git
    cd nibiru
    git checkout v1.2.0
    make install

# Initialize Node

    nibid init < Your_Moniker> --chain-id=cataclysm-1

# Download genesis

    wget -O genesis.json https://raw.githubusercontent.com/Validator247/Nibiru-Mainnet/main/genesis.json


# Download addrbook

    wget -O addrbook.json https://raw.githubusercontent.com/Validator247/Nibiru-Mainnet/main/addrbook.json

# Set prunning

    pruning="custom" && \
    pruning_keep_recent="100" && \
    pruning_keep_every="0" && \
    pruning_interval="10" && \
    sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nibid/config/app.toml && \
    sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nibid/config/app.toml

# Create a service

    sudo tee /etc/systemd/system/nibid.service > /dev/null <<EOF
    [Unit]
    Description=nibid Daemon
    After=network-online.target
    [Service]
    User=$USER
    ExecStart=$(which nibid) start
    Restart=always
    RestartSec=3
    LimitNOFILE=65535
    [Install]
    WantedBy=multi-user.target
    EOF
    sudo systemctl daemon-reload
    sudo systemctl enable nibid

# Download Snapshot

Install lz4 if needed

    sudo apt update
    sudo apt install snapd -y
    sudo snap install lz4

Download the snapshot

    wget -O nibiru_4448231.tar.lz4 https://snapshots.polkachu.com/snapshots/nibiru/nibiru_4448231.tar.lz4 --inet4-only

Decompress the snapshot

    lz4 -c -d nibiru_4448231.tar.lz4  | tar -x -C $HOME/.nibid

Remove downloaded snapshot to free up space

    rm -v nibiru_4448231.tar.lz4

# Launch Node

    sudo systemctl restart nibid && journalctl -u nibid -f -o cat

# Create Wallet

Add New Wallet Key

    nibid keys add wallet

Recover existing key

    nibid keys add wallet --recover

List All Keys

    nibid keys list

Check Balance

    nibid query bank balances <Your_Wallet_Address>

# Create validator

     nibid tx staking create-validator \
     --amount 1000000unibi \
     --commission-max-change-rate "0.05" \
     --commission-max-rate "0.10" \
     --commission-rate "0.05" \
     --min-self-delegation "1" \
     --pubkey=$(nibid tendermint show-validator) \
     --moniker 'validator247.com' \
     --website "https://validator247.com" \
     --identity "xxxxxxxx" \
     --details "xxxxxxxx" \
     --security-contact="your_mail" \
     --chain-id cataclysm-1 \
     --from wallet
     -y

# Delete node

    sudo systemctl stop nibid
    sudo systemctl disable nibid
    rm /etc/systemd/system/nibid.service
    sudo systemctl daemon-reload
    cd $HOME
    rm -rf .nibid
    rm -rf nibiru
    rm -rf $(which nibid)

# Unjail

    nibid tx slashing unjail \
    --chain-id cataclysm-1 \
    --fees 5000unibi \
    --from wallet \
    -y

# Withdraw rewards

        nibid tx distribution withdraw-rewards $(nibid keys show wallet --bech val -a) --from wallet --commission --chain-id cataclysm-1 --gas auto --fees 5000unibi -y

# DONE        

