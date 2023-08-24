### WSL Installation​​​​​​​ for Python and Nodejs Dev

-   Install WSL from the Suncor Company Portal and **reboot** machine
-   Open terminal, search **`terminal`** from task bar Search box, then open **`terminal`**, you can run all the commands under **`terminal`**
-   List all the available Linux distributions

    ```shell
    wsl -l -o
    ```

    ```shell
    The following is a list of valid distributions that can be installed.
    Install using 'wsl.exe --install <Distro>'.

    NAME                                   FRIENDLY NAME
    Ubuntu                                 Ubuntu
    Debian                                 Debian GNU/Linux
    kali-linux                             Kali Linux Rolling
    Ubuntu-18.04                           Ubuntu 18.04 LTS
    Ubuntu-20.04                           Ubuntu 20.04 LTS
    Ubuntu-22.04                           Ubuntu 22.04 LTS
    OracleLinux_7_9                        Oracle Linux 7.9
    OracleLinux_8_7                        Oracle Linux 8.7
    OracleLinux_9_1                        Oracle Linux 9.1
    openSUSE-Leap-15.5                     openSUSE Leap 15.5
    SUSE-Linux-Enterprise-Server-15-SP4    SUSE Linux Enterprise Server 15 SP4
    SUSE-Linux-Enterprise-15-SP5           SUSE Linux Enterprise 15 SP5
    openSUSE-Tumbleweed                    openSUSE Tumbleweed
    ```

-   Install an Ubuntu image under terminal, run `wsl --install <Distro>`. Complete the setup of the image
    ```shell
    # for example, install Ubuntu-22.04, you can install other Linux distribution
    wsl --install Ubuntu-22.04
    ```
-   Shutdown and restart WSL
    ```shell
    wsl --shutdown
    ```
-   Check your linux distribution, example, Ubuntu 22.04.2 LTS
    ```bash
    lsb_release -a
    No LSB modules are available.
    Distributor ID: Ubuntu
    Description:    Ubuntu 20.04.6 LTS
    Release:        20.04
    Codename:       focal
    ```
-   Ensure the elelvated priviledges in the current session
    ```bash
    sudo ls
    [sudo] password for ding:
    ```
-   Setup network for WSL, open Ubuntu under terminal
    ```bash
    sudo rm /etc/resolv.conf && \
    sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf' && \
    sudo bash -c 'echo "[network]" > /etc/wsl.conf' && \
    sudo bash -c 'echo "generateResolvConf = false" >> /etc/wsl.conf' && \
    sudo chattr +i /etc/resolv.conf
    ```
-   Check the network
    ```bash
    ping www.google.com
    ```
    if the network is working, you should get the response like the followings:
    ```bash
    PING www.google.com (142.250.190.100) 56(84) bytes of data.
    64 bytes from ord37s35-in-f4.1e100.net (142.250.190.100): icmp_seq=1 ttl=114 time=33.0 ms
    64 bytes from ord37s35-in-f4.1e100.net (142.250.190.100): icmp_seq=2 ttl=114 time=32.8 ms
    ```
-   Install **ZScaler Root Certificate** on WSL
    Done to experience less pain with secure connections from WSL, with ZScaler connected. Do this part before you do any of the further steps with **pip**, **npm**, **docker**, as it's relevant to ensure those tools can also connect to the outside world when you're connected through ZScaler. If you get the message(**`SSL certificate problem: unable to get local issuer certificate`**), ZScaler is connected and you don't have the appropriate certificates set up on your machine. - Open terminal under Powershell

    ```bash
    certmgr.msc
    ```

    A window will pop up. Under menu leftside, click **`Trusted Root Certification Authorities`**, then click **`Certificates`**, scroll down to **`Zscaler Root CA`**, click **`Aciton`** on the top menu of the window, hover **`All tasks`**, click **`Export`**. Export the **`ZScaler Root CA from`** there: Windows Certificate Manage export Choose **'Base-64 encoded X.509 (.CER)'** as the export format, and use the resulting file as your .cer file, example **zscaler.cer**. - Copy the Zscaler certificate from windows directory to linux distribution certificates directory:

    ```bash
    sudo cp /mnt/c/Users/ding/Documents/zscaler.cer /usr/local/share/ca-certificates/zscaler.crt
    sudo update-ca-certificates
    ```

    -   You should see **`1 added, 0 removed; done.`** in the ensuing output. The above creates a symlink /etc/ssl/certs/zscaler.pem that directs to the above .crt file. A master of all certificates on the system is stored as /etc/ssl/certs/ca-certificates.crt, and the zscaler.pem contents are in there. As mentioned, configuring the **REQUESTS_CA_BUNDLE** environment variable is necessary for the python requests library (and the many libraries that rely on it) to function properly over https. The **NODE_EXTRA_CA_CERTS** variable that is set relates to nodejs development; it is set here as good practice per the guidance from ZScaler's site, above. - Run the following command to add environment variables to your WSL **`~/.bashrc`** file

        ```bash
        echo '# SSL Certificates & ZScaler' >> ~/.bashrc \
        && echo 'export CERT_PATH=/etc/ssl/certs/zscaler.pem' >> ~/.bashrc \
        && echo 'export CERT_DIR=/etc/ssl/certs/' >> ~/.bashrc \
        && echo 'export SSL_CERT_FILE=${CERT_PATH}' >> ~/.bashrc \
        && echo 'export SSL_CERT_DIR=${CERT_DIR}' >> ~/.bashrc \
        && echo 'export REQUESTS_CA_BUNDLE=${CERT_PATH}' >> ~/.bashrc \
        && echo 'export NODE_EXTRA_CA_CERTS=${CERT_PATH}' >> ~/.bashrc \
        && source ~/.bashrc
        ```

    -   Restart the terminal

-   Run Python and pip to test the Python development setup

    -   Ubuntu-20.04 comes with Python 3.8.10 and Ubuntu-22.04 comes with Python 3.10.6, check the version of Python

    ```shell
    python3 --version
    ```

    -   Update apt

    ```shell
    sudo apt update
    ```

    -   Install pip(Python package management)

    ```shell
    sudo apt install python3-pip
    ```

    -   Check the verison of pip

    ```shell
    pip --version
    pip 20.0.2 from /usr/lib/python3/dist-packages/pip (python 3.8)
    ```

    -   Use pip to install Python dependencies

    ```shell
    pip install requests
    ```

-   Run Nodejs and npm to test the Nodejs development setup
    -   Update apt
    ```shell
    sudo apt update
    ```
    -   Install nvm(recommened by Nodejs)
    ```shell
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
    ```
    -   Install **nvm** script to your account.
    ```shell
    source ~/.bashrc
    ```
    -   Check the Nodejs version available, for example v18
    ```shell
    nvm list-remote | grep v18
    ```
    -   Install Nodejs v18.17.0
    ```shell
    nvm install v18.17.0
    Downloading and installing node v18.17.0...
    Downloading https://nodejs.org/dist/v18.17.0/node-v18.17.0-linux-x64.tar.xz...
    ##################################################################################################### 100.0%
    ```
    -   Check the verison of nodejs
    ```shell
    node --version
    v18.17.0
    ```
    -   Check the version of npm
    ```shell
    npm --version
    9.6.7
    ```
    -   Use npm to install Nodejs dependencies as a test
    ```shell
    mkdir tmp && cd tmp
    npm init -y
    npm install express
    cd ..
    rm -rf tmp
    ```
-   **Run tests** to check Python and Nodejs development environment are working

    -   With certificates correctly installed, these should succeed on WSL (the **https** is important where stated in these tests - don't omit it). If you see errors for **https** URLs, then try the equivalent **http** URL where you can to check connectivity is there:

    -   curl test

    ```bash
    curl https://www.google.com
    ```

    -   wget test

    ```bash
    wget -O- https://www.google.com
    ```

    -   Python **requests** test

    ```bash
    python3
    ```

    -   Run scripts under python interpreter

    ```python
    import requests
    requests.get('https://www.google.com')
    ```

    ...output should be <Response [200]> in the interpreter

    -   If the above tests passed, that means your WSL development has been setup successfully

#### Troubleshooting

The above steps should work for most of the cases, if you still have issues, please try the following steps to troubleshoot:

```bash
pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org <python-libary>
```

install pandas

```bash
pip install --trusted-host files.pythonhosted.org --trusted-host pypi.org --trusted-host pypi.python.org pandas
```

### Docker Installation Under WSL

-   In WSL, install docker:

```bash
sudo apt update
sudo apt install docker.io -y
```

-   Add your username to the docker group, to allow you to run docker

```bash
sudo usermod -aG docker $USER
newgrp docker  # to ensure your group membership is updated in the current session
```

-   Check docker is working

```bash
docker --version
```

-   Output should be

```bash
Docker version 20.10.21, build 20.10.21-0ubuntu1~22.04.3
```

-   Check status of docker engine and client

```bash
docker info
```

-   Run a docker container and docker should pull the **hello-world** image automatically

```bash
docker run hello-world
```

-   Build docker image with avoiding **`Zscaler Certificate Issue`**

    -   Copy your **zscaler.pem**(your Zscaler cert file) from **`/etc/ssl/certs/`** to your build context, aka, the current project base directory to avoid the Zscaler certificate issue during the building of docker image
    -   Dockerfile example for Python and Nodejs

    ```Dockerfile
      # Use the official Python 3.10 image as the base image
      FROM python:3.10

      # Set the working directory inside the container
      WORKDIR /app

      # Copy the CA certificate file from the host to the container
      COPY zscaler.pem /app/

      # Set the environment variable to specify the CA certificate file for python
      ENV REQUESTS_CA_BUNDLE /app/zscaler.pem

      # Set the environment variable to specify the CA certificate file for nodejs
      ENV NODE_EXTRA_CA_CERTS /app/zscaler.pem

      # Create a virtual environment avoid running as root user
      RUN python -m venv /venv

      # Add the virtual environment's binary directory to the PATH
      ENV PATH="/venv/bin:$PATH"

      # Activate the virtual environment
      RUN . /venv/bin/activate

      # Install pip upgrade
      RUN pip install --upgrade pip

      # Copy the requirements.txt file
      COPY requirements.txt /app

      # Install any required dependencies
      RUN pip install -r requirements.txt

      # Set the CMD to activate the virtual environment
      CMD ["/bin/bash", "-c", "source /venv/bin/activate"]
    ```

    -   Run docker build

    ```bash
    docker build -t image_name .
    ```
