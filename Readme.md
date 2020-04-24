# Metaflow Setup 

Setup Entails : 

1. Metadata service setup
    1. Service Setup with Python
    2. NGINX Auth over the service. 
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

4. Run Command to Deploy.
    ```
    ansible-playbook -i hosts main.yml
    ```
5. [Setup SSL NGINX Certs using Cerbot](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx) and route domain use Route53. SSL Setup uses [LetsEncrypt CA](https://letsencrypt.org/getting-started/). This will even setup domain on NGINX.

6. [Add Authentation/Users to route using `htpasswd`](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-http-basic-authentication/). 

7. Add Redirect to metadata service and `htpasswd` location to NGINX site-enabled at `/etc/nginx/sites-enabled/default`. For template `/etc/nginx/sites-enabled/default` check [nginx.conf](nginx.conf)
    ```conf
            location / {

                    auth_basic "Admin Users Only Area";
                    # This will hold users added by `htpasswd`
                    auth_basic_user_file /etc/nginx/users/.htpasswd;
                    
                    # METAFLOW REDIRECT
                    proxy_pass http://127.0.0.1:8080/;
                
                    # Redefine the header fields that NGINX sends to the upstream server
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                
                    # Define the maximum file size on file uploads
                    client_max_body_size 5M;
            }
    ```

8. Final Metaflow config Looks like : 
    ```json
    {   
        
        "METADATA_SERVICE_URL":"https://patentwerx.isdatatower.com/",
        "METADATA_SERVICE_HEADERS":"{\"Authorization\": \"Basic <BASE_64_OF(usename:password)>\"}",
        "METAFLOW_DATASTORE_SYSROOT_S3":"s3://metaflow-machine-learning-staging/metaflow_store",
        "METAFLOW_DATATOOLS_SYSROOT_S3":"s3://metaflow-machine-learning-staging/metaflow_store/data",
        "METAFLOW_ECS_S3_ACCESS_IAM_ROLE":"<ROLE_ARN>",
        "METAFLOW_DEFAULT_DATASTORE": "s3",
        "METAFLOW_DEFAULT_METADATA": "service",
        "METAFLOW_BATCH_JOB_QUEUE":"<BATCH_JOBQ_ARN>"
    }
    ```
9. For Batch specification check metaflow website. 



