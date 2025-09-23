# Deploy Flask behind Apache on Metropolia Ecloud (Rocky Linux)

Note: We use the RockyLinux LAMP template primarily for its preinstalled Apache. The application we deploy is a Python/Flask app running behind Apache via reverse proxy.

## Viewing web pages on the virtual machine from outside the school network

1. Use [Metropolia's VPN connection](https://wiki.metropolia.fi/display/tietohallinto/VPN-yhteys+GlobalProtect-palvelun+kautta)
2. Other options (e.g. during Zoom session to reduce VPN bandwidth)

   1. To test your app/webpages, create an SSH SOCKS tunnel and configure one of your browsers through it (use your Metropolia username and password). See: [SSH tunneling](https://tietohallinto.metropolia.fi/display/tietohallinto/SSH-tunnelointi)

      ```bash
      ssh -Nf -D 8888 <met-username>@shell.metropolia.fi
      ```

   2. For terminal access to your server, you can use two-hop SSH via the Metropolia shell host:

      ```bash
      # First hop to Metropolia shell
      ssh <your-metropolia-username>@shell.metropolia.fi

      # From shell host to your server
      ssh <your-server-username>@<your-server-IP>
      ```

      Or use ProxyJump in one command:

      ```bash
      ssh -J <your-metropolia-username>@shell.metropolia.fi <your-server-username>@<your-server-IP>
      ```

## Important things to remember

1. You have to remember all usernames and passwords that you create throughout the whole course. Otherwise you need to start from the beginning. You have been warned.
2. If something goes wrong during the installation, the fastest way to fix something is to delete the virtual computer and start from scratch and then follow the instructions more carefully.

## LAMP

1. Request a virtual machine for yourself

   1. Go to [https://ecloud.metropolia.fi/](https://ecloud.metropolia.fi/)
   2. Log in with your credentials
   3. On the Catalog Items page, select RockyLinux
      - Choose ‘RockyLinux LAMP’ from the VM Template row. It has Apache preinstalled
   4. Select S-Small, Lease time 120
   5. ([IT Services instructions](https://wiki.metropolia.fi/pages/viewpage.action?pageId=257364221))

2. Wait 10–15 minutes for the machine to be prepared

   1. You will receive an email with the machine's IP address, password, etc.

3. If you are using Windows, newer Windows versions include PowerShell, or during the wait install a terminal/CLI. A recommended option is [Git Bash (bundled with Git)](https://git-scm.com/downloads). On Mac/Linux the terminal is available by default (on macOS it's in the Applications/Utilities folder)

4. Start a terminal and connect to the virtual machine (if you are on the school network, skip to step 2 below)

   1. If you are not on a school computer or the eduroam network, use [Metropolia's VPN connection](https://wiki.metropolia.fi/pages/viewpage.action?pageId=149652071)
   2. Then connect to the virtual machine:

      ```bash
      ssh <username>@<IP-address-from-email>
      ```

   3. Username: see the email. Password: see the email.
   4. If the terminal shows ‘...cannot change locale (UTF-8)...’, run the following command:  
      sudo localedef -i en_US -f UTF-8 en_US.UTF-8
   5. Optionally, change your password:

      ```bash
      passwd
      ```

5. Rocky Linux's package manager (used to install and update software) is called DNF. You can use it to fetch updates for installed software, which is recommended regularly. The update command is:  
   sudo dnf upgrade --refresh

6. When you no longer need the virtual machine, decommission it in ecloud.metropolia.fi under lifecycle -> retire this vm
   1. Virtual machines are not backed up, so do this only after you have received a grade for the assignment/course

## Install and configure Python3

1. Install python3 and pip3 (Python package manager)

   ```bash
   sudo dnf install -y python3 python3-pip
   ```

2. Check that python3 is installed:

   ```bash
   python3 --version
   ```

3. Configure Apache httpd server as a reverse proxy to your Python/Flask application (e.g., running on 127.0.0.1:5000):

   1. Allow Apache to make outbound connections

      ```bash
      sudo /usr/sbin/setsebool -P httpd_can_network_connect 1
      ```

   2. Create certificates for HTTPS:

      - `sudo dnf install mod_ssl openssl`
      - `openssl genrsa -out ca.key 2048`
      - `openssl req -new -key ca.key -out ca.csr`
      - `openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt`
      - `sudo cp ca.crt /etc/pki/tls/certs`
      - `sudo cp ca.key /etc/pki/tls/private/ca.key`
      - `sudo cp ca.csr /etc/pki/tls/certs/ca.csr`

   3. Restart Apache:

      ```bash
      sudo systemctl restart httpd.service
      ```

   4. Test that Apache works by going to <https://10.x.x.x> in your browser. Because the certificate is self-signed, your browser will warn you; click Advanced and proceed to the site (or import/trust the certificate). Replace 10.x.x.x with your VM's IP address.

4. Apache HTTPS settings:

   1. Open Apache's SSL configuration file:

      ```bash
      sudo nano /etc/httpd/conf.d/https-flask.conf
      ```

   2. Content: (note: change ServerName to your host, see ecloud.metropolia.fi -> Resources -> Hostname):

   ```conf
   <VirtualHost *:443>
      ServerName username-number.educloud.metropolia.fi
      SSLEngine on
      SSLCertificateFile /etc/pki/tls/certs/ca.crt
      SSLCertificateKeyFile /etc/pki/tls/private/ca.key

      # Reverse proxy to Flask app (adjust path/port as needed)
      ProxyPreserveHost On
      ProxyPass /appname/ http://127.0.0.1:5000/
      ProxyPassReverse /appname/ http://127.0.0.1:5000/

      # Forward headers to help Flask generate correct URLs behind proxy
      RequestHeader set X-Forwarded-Proto "https"
      RequestHeader set X-Forwarded-Prefix "/appname"
   </VirtualHost>
   ```

   Optional hardening/notes:

   - Ensure required modules are available: `mod_ssl`, `mod_proxy`, `mod_proxy_http`, `mod_headers` (usually installed by default).
   - If using firewalld, open HTTPS: `sudo firewall-cmd --permanent --add-service=https && sudo firewall-cmd --reload`.
   - Restrict key file permissions: `sudo chmod 600 /etc/pki/tls/private/ca.key && sudo chown root:root /etc/pki/tls/private/ca.key`.

   1. Save and close (nano: ctrl+o, Enter, ctrl+x)
   2. Restart Apache:

      ```bash
      sudo systemctl restart httpd.service
      ```

5. Test your own app by cloning your git repository and installing dependencies

   1. Install git first:

      ```bash
      sudo dnf install -y git
      ```

   2. **Optional**: If you want to be able to push from virtual machine to your github repository, you need to generate ssh keys and add the public key to your github account (see links below)

      - [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
      - [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

   3. Clone your repository (replace `<your-repo-url>` with your repository URL):

      ```bash
      git clone <your-repo-url> myapp
      cd myapp
      ```

   4. Install dependencies (assuming you have a requirements.txt file in your repository):

      ```bash
      pip3 install -r requirements.txt
      ```

   5. Run your application (development server example; for production consider Gunicorn + systemd):

      ```bash
      python3 app.py &
      ```

   6. Now you should be able to access your application at:

      ```text
      https://<your-server-ip>/appname/
      ```
