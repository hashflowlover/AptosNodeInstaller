# AptosNodeInstaller
A simple guide

#!/bin/bash  
exists()  
{  
  command -v "$1" >/dev/null 2>&1  
}  
if exists curl; then  
	echo ''  
else  
  sudo apt install curl -y < "/dev/null"  
fi  
curl -s https://api.nodes.guru/logo.sh | bash && sleep 2  
curl -s https://api.nodes.guru/swap8.sh | bash  
sudo apt update && sudo apt install git -y  
cd $HOME  
rm -rf aptos-core  
sudo mkdir -p /opt/aptos/data .aptos/config  
git clone https://github.com/aptos-labs/aptos-core.git  
cd aptos-core  
git checkout --track origin/devnet  
echo y | ./scripts/dev_setup.sh  
source ~/.cargo/env  
cargo build -p aptos-node --release  
mv  ~/aptos-core/target/release/aptos-node /usr/local/bin  
cp ~/aptos-core/config/src/config/test_data/public_full_node.yaml ~/.aptos/config/fullnode.yaml  
wget -q -O $HOME/.aptos/config/genesis.blob https://devnet.aptoslabs.com/genesis.blob  
wget -q -O $HOME/.aptos/config/waypoint.txt https://devnet.aptoslabs.com/waypoint.txt  
sleep 2   
sed -i.bak -e "s/127.0.0.1/0.0.0.0/" $HOME/.aptos/config/fullnode.yaml  
sed -i "s|genesis_file_location: .*|genesis_file_location: \"$HOME/.aptos/config/genesis.blob\"|" $HOME/.aptos/config/fullnode.yaml
sed -i "s|from_file: .*|from_file: \"$HOME/.aptos/config/waypoint.txt\"|" $HOME/.aptos/config/fullnode.yaml  
  
  
echo "[Unit]  
Description=Aptos  
After=network.target  

[Service]  
User=$USER  
Type=simple  
ExecStart=/usr/local/bin/aptos-node -f $HOME/.aptos/config/fullnode.yaml  
Restart=on-failure  
LimitNOFILE=65535  
  
[Install]  
WantedBy=multi-user.target" > $HOME/aptosd.service  
mv $HOME/aptosd.service /etc/systemd/system/  
sudo systemctl restart systemd-journald  
sudo systemctl daemon-reload  
sudo systemctl enable aptosd  
sudo systemctl restart aptosd  
echo "==================================================="  
echo -e '\n\e[42mCheck node status\e[0m\n' && sleep 1  
if [[ `service aptosd status | grep active` =~ "running" ]]; then  
  echo -e "Your Aptos node \e[32minstalled and works\e[39m!"  
  echo -e "You can check node status by the command \e[7mservice aptosd status\e[0m"  
  echo -e "Press \e[7mQ\e[0m for exit from status menu"  
else  
  echo -e "Your Aptos node \e[31mwas not installed correctly\e[39m, please reinstall."  
fi  
