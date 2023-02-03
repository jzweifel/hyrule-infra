sudo apt install openvpn unzip
cd /etc/openvpn
sudo wget https://downloads.nordcdn.com/configs/archives/servers/ovpn.zip
sudo unzip ovpn.zip
sudo rm ovpn.zip
cd ovpn_udp
sudo cp be205.nordvpn.com.udp.ovpn ../belgium205.conf
cd ..
sudo nano belgium205.conf
 -> change auth-user-pass to auth-user-pass auth.txt
create auth.txt with nordvpn creds
sudo nano /etc/default/openvpn
 -> change #AUTOSTART to AUTOSTART="belgium205"