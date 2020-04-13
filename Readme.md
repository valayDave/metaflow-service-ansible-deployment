# Metaflow Setup 

Setup Entails : 

1. Metadata service setup
    1. NGINX Auth over the service. (TODO)
    2. Service Setup with Python
2. Metadata Postgres setup


## Setup Details 
1. Setup Secrets in Secrets/var.yml
    ```yaml

    MF_METADATA_DB_USER: username
    MF_METADATA_DB_PSWD: supersecurepassword
    MF_METADATA_DB_NAME: metaflow
    postgresql_databases:
        - name: "{{MF_METADATA_DB_NAME}}"
    postgresql_users:
        - name: "{{MF_METADATA_DB_USER}}"
        password: "{{MF_METADATA_DB_PSWD}}"
    ```
2. Run `apt-get update` on remote server
3. Setup Hosts configurations 
    ```
    [deploy-machine]
    m-1 ansible_host=<MACHINE_IP> ansible_port=22

    [deploy-machine:vars]
    ansible_user=<MACHINE_USER> 
    ansible_ssh_private_key_file=<PATH_TO_KEY_FILE>
    ansible_python_interpreter=/usr/bin/python3
    ```

3. Run Command to Deploy.
    ```
    ansible-playbook -i hosts main.yml
    ```

