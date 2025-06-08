### ✨ Ansible High-Availability WordPress Deployment ✨

This Infrastructure as Code (IaC) project provides a collection of Ansible playbooks and roles to provision the entire deployment of a high-availability, secure WordPress stack. The architecture features an Nginx load balancer distributing traffic between two application servers, a dedicated MariaDB database, and security hardening using iptables and fail2ban. The entire process is automated to ensure repeatable and consistent deployments.

-----

### 🎯 About The Project

This project was created to demonstrate a real-world application of Infrastructure as Code principles using Ansible. The goal is to automate the setup of a scalable and secure web application environment from scratch. The entire infrastructure is defined in code, making it version-controllable, repeatable, and easy to manage.

**Key Architectural Features:**

  * **🚀 High-Availability:** A Nginx load balancer (`labfive.com`) distributes traffic across two backend application servers to prevent a single point of failure and handle increased load.
  * **🏗️ Separation of Concerns:** The infrastructure is logically tiered into a load balancer, application servers (`labtwo.com`, `labthree.com`), and a dedicated database server (`labfour.com`).
  * **🛡️ Security by Default:** Includes roles to configure `iptables` for firewall management and `fail2ban` to protect against brute-force attacks.
  * **🔑 Automated SSL:** Generates and configures a self-signed SSL certificate on the load balancer for secure HTTPS traffic, with SSL Termination handled at the load balancer.

### 🛠️ Built With

This project is built with a standard and powerful set of open-source technologies:

  * [Ansible](https://www.ansible.com/)
  * [Nginx](https://www.nginx.com/)
  * [MariaDB](https://mariadb.org/)
  * [WordPress](https://wordpress.org/)
  * [PHP-FPM](https://www.php.net/manual/en/install.fpm.php)
  * [Fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page)
  * [iptables](https://www.netfilter.org/)

-----

### 🚀 Deployment Options

This guide provides two main deployment options:

1.  **Production Deployment:** Uses Certbot to obtain a valid SSL certificate from Let's Encrypt. Requires a public domain and server.
2.  **Local/Lab Deployment:** Uses self-signed SSL certificates for local testing environments like VirtualBox.

-----

### 🌍 Option 1: Production Deployment with Let's Encrypt (Certbot)

<br>

This method is intended for production servers that are accessible from the public internet. It uses Certbot to automate the acquisition and renewal of free, trusted SSL certificates from Let's Encrypt, enabling full HTTPS security.

<br>

#### ✅ Prerequisites for Production Deployment

  * A registered public domain name (e.g., `yourdomain.com`).
  * Public DNS `A` records for `yourdomain.com` and `www.yourdomain.com` pointing to the public IP address of your load balancer server.
  * Ports **80** and **443** must be open and forwarded from any network firewalls to your load balancer server.
  * Ansible installed on a control node.
  * Target servers prepared with SSH access and Python 3.

#### 🔧 Ansible Configuration for Certbot

The `loadbalancer` role is configured to handle the entire Certbot process using a two-stage approach to pass the Let's Encrypt HTTP-01 challenge.

1.  **Inventory Setup:**

      * In `inventory/group_vars/lb.yml`, ensure your variables are set correctly:
        ```yaml
        LB_DOMAIN: yourdomain.com
        LB_NGINX_INSTANCES:
          - domain: yourdomain.com
            email: your-email@example.com
        ```

2.  **Two-Stage Nginx Configuration:**

      * **Initial Config (`lb-initial-http.conf.j2`):** The playbook first applies a minimal Nginx configuration that only listens on port 80 and includes a `location` block for `/.well-known/acme-challenge/`. This allows the Certbot challenge to be served successfully.
      * **Certbot Execution:** The `Obtain SSL certificate...` task then runs, using the `--nginx` plugin to validate domain ownership and obtain the certificates. A `--deploy-hook` is used to automatically reload Nginx upon successful acquisition or renewal.
      * **Final Config (`lb-final-ssl.conf.j2`):** After the certificate is obtained, the playbook applies the final, complete Nginx configuration. This file includes the `listen 443 ssl` block, references the Let's Encrypt certificate paths, and sets up a permanent redirect from HTTP to HTTPS.

#### ▶️ Production Deployment Execution

1.  **Configure your inventory** with your public domain and server IPs.
2.  **Run the main playbook:**
    ```sh
    ansible-playbook playbooks/all.yml
    ```
    The playbook will handle all steps automatically. After it completes, your site will be accessible at `https://yourdomain.com` with a valid, browser-trusted SSL certificate.

-----

### 🏠 Option 2: Local/Lab Deployment with Self-Signed SSL (VirtualBox)

<br>

This method is ideal for local development, testing, or lab environments where a public domain or internet accessibility is not available. It uses Ansible's crypto modules to generate a self-signed SSL certificate.

<br>

## Getting Started

To get a local copy up and running, follow these simple steps.

#### ✅ Prerequisites for Local Deployment

  * **Ansible** installed on your control node.
  * **Virtualization Software** like VirtualBox.
  * Four running virtual machines (e.g., Ubuntu Server) with SSH access and Python 3.

#### 🔧 Local Deployment Steps

1.  **Clone the repository:**

    ```sh
    git clone <your-repository-url>
    cd <your-repository-name>
    ```

2.  **Install Ansible Collections:**
    This method uses the `community.crypto` collection. Install it if you haven't already:

    ```sh
    ansible-galaxy collection install community.crypto
    ```

3.  **Configure the Inventory:**

      * Open `inventory/main.yml` and update the hostnames and `ansible_host` IP addresses to match your virtual machines (e.g., `192.168.165.12`, `192.168.165.13`, etc.).

4.  **Run the Main Playbook:**

      * The tasks in the `loadbalancer` role will automatically generate a private key and a self-signed certificate for your domain (`lab.com` by default).
      * Execute the main playbook:
        ```sh
        ansible-playbook playbooks/all.yml
        ```


        This command will provision and configure all four servers according to their roles.

#### ▶️ Usage for Local Deployment

After the playbook completes:

1.  **Update Your Local Hosts File:**
    To access the site from your host machine's browser, add the following line to your local hosts file (`/etc/hosts` on Linux/macOS or `C:\Windows\System32\drivers\etc\hosts` on Windows):

    ```
    192.168.165.15  lab.com www.lab.com
    ```

    *(Remember to replace `192.168.165.15` with the IP of your load balancer VM)*.

2.  **Access the Site and Accept Security Warning:**

      * Open your web browser and navigate to `https://lab.com`.
      * Because the project uses a self-signed SSL certificate, **your browser will show a security warning**. This is expected. You must accept the warning (usually by clicking "Advanced" -\> "Proceed to lab.com (unsafe)") to access the site.

-----

### 📂 Project Structure

The project follows standard Ansible best practices for structure and organization:

  * **`inventory/`**: Contains the host inventory and all related variables (`group_vars` and `host_vars`).
  * **`playbooks/`**: Holds the main playbooks that define which roles are applied to which hosts.
  * **`roles/`**: Contains all the Ansible roles, where each role is responsible for a specific component of the infrastructure (e.g., `nginx`, `mariadb`, `wordpress`).

-----

### 📞 Contact

Soroush Imanian - [Soroushimanian@gmail.com](mailto:Soroushimanian@gmail.com) - [LinkedIn](https://www.linkedin.com/in/soroush-imanian)

Project Link: [https://github.com/SoroushImanian/AutoPress](https://github.com/SoroushImanian/AutoPress)