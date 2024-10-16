# JupyterLab setup

## Setup JupyterLab environment compute instance

- Create an Oracle Linux 9 compute instance on OCI
- Connect to it as user `opc` and run the following

```
sudo dnf update
sudo dnf install -y jdk-22-headful

python --version
python -m ensurepip --upgrade
python -m pip install --user jupyterlab
python -m pip list | grep jupyterlab

sudo dnf install oracle-instantclient-release-23ai-el9
sudo dnf install oracle-instantclient-basic
sudo dnf install oracle-instantclient-sqlplus
sudo dnf list installed | grep instantclient
```


## Configure JupyterLab

- Generate JupyterLab configuration file and write down its path

```
jupyter-lab --generate-config
```

- Generate JupyterLab password hash in a JSON file

```
jupyter-lab password
```

- In the JupyterLab configuration file, replace the variable `c.NotebookApp.password` as follows with the generated password hash from the JSON file

```
c.NotebookApp.password = u'<string_from_the_json_file>'
```


## Start and access JupyterLab

- Start JupyterLab

```
jupyter-lab --no-browser ; 
```

- Create a tunnel on port 8888 to access JupyterLab via browser
```
ssh -i /your/ssh/key/file.key -L 8888:localhost:8888 opc@<jupyter-lab_server>
```

- Navigate to `http://localhost:8888/`



# Oracle client keystore setup

- In file `/usr/lib/oracle/23/client64/lib/network/admin/tnsnames.ora`, create necessary TNS entries to connect to the target CDB and PDB. For example
```
application_cdb_admin
application_pdb_admin
application_pdb_appuser
application_pdb_firefighter
```

- Using `mkstore` or `orapki` with a full Database software installation / full Oracle client installation, create the keystore and add the necessary credentials
```
mkstore -wrl /path/to/keystore -create

mkstore -wrl /path/to/keystore -createCredential application_cdb_admin SYS
mkstore -wrl /path/to/keystore -createCredential application_pdb_admin SYS
mkstore -wrl /path/to/keystore -createCredential application_pdb_appuser APPUSER
mkstore -wrl /path/to/keystore -createCredential application_pdb_firefighter FIREFIGHTER

mkstore -wrl /path/to/keystore -listCredential
```

- Transfer the keystore directory `/path/to/keystore` to your JupyterLab compute instance, in any location

- Create the file `/usr/lib/oracle/23/client64/lib/network/admin/sqlnet.ora` to point to the newly created keystore
```
WALLET_LOCATION = (SOURCE = (METHOD = FILE) (METHOD_DATA = (DIRECTORY = /path/to/keystore/on/jupyterlab)))
SQLNET.WALLET_OVERRIDE = TRUE 
```
