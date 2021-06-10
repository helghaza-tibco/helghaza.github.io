How to Install VNC on an AWS EC2 Centos 7.2 AMI
Reference: http://devopscube.com/how-to-setup-gui-for-amazon-ec2-rhel-7-instance/
Reference #2 : https://www.itzgeek.com/how-tos/linux/centos-how-tos/install-xrdp-on-centos-7-rhel-7.html


1. Update the server using the following command.

sudo yum -y update

2. Install the gnome GUI components using the following command.

sudo yum groupinstall -y "Server with GUI"

3. Issue the following commands to start the GUI during boot.

sudo systemctl set-default graphical.target
sudo systemctl default

4. Install the VNC service
sudo yum install -y tigervnc-server

5. Setup VNC password for you user account, this is used when you login to your EC2 Desktop
sudo passwd EC2-user

5.1 [Optional] Setup VNC password for the root user account. Should not be needed, but in case it is
sudu su
passwd
exit

6. Connect to cloud server with tunnel forwarding for VNC desktop :1. :2 is 5902, :3 is 5903 and so on.
ssh -L 5901:localhost:5901 -i <your.pem> EC2-user@<public IP/DNS name>

7. Start VNC server
vncserver <your options>

8. Connect to VNC server 
Use localhost:1 as the server name

9. Changing resolution of the VNC desktop
Open xterm
#xrandr by itself shows all available resolutions.
xrandr -s <resultion>
