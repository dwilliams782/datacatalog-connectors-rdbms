# mysql2datacatalog

Library for ingesting MySQL metadata into Google Cloud Data Catalog.

**Disclaimer: This is not an officially supported Google product.**

## 1. Environment setup

### 1.1. Get the code

````bash
git clone https://.../mysql2datacatalog.git
cd mysql2datacatalog
````

### 1.2. Auth credentials

##### 1.2.1. Create a service account and grant it below roles

- Data Catalog Admin

##### 1.2.2. Download a JSON key and save it as
- `<YOUR-CREDENTIALS_FILES_FOLDER>/mysql2dc-credentials.json`

> Please notice this folder and file will be required in next steps.

### 1.3. Virtualenv

Using *virtualenv* is optional, but strongly recommended unless you use Docker or a PEX file.

##### 1.3.1. Install Python 3.6+

##### 1.3.2. Create and activate a *virtualenv*

```bash
pip install --upgrade virtualenv
python3 -m virtualenv --python python3 env
source ./env/bin/activate
```

##### 1.3.3. Install the dependencies

```bash
pip install ./lib/datacatalog_connectors_commons-1.0.0-py2.py3-none-any.whl
pip install ./lib/rdbms2datacatalog-1.0.0-py2.py3-none-any.whl
pip install --editable .
```

##### 1.3.4. Set environment variables

Replace below values according to your environment:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=data_catalog_credentials_file

export MYSQL2DC_DATACATALOG_PROJECT_ID=google_cloud_project_id
export MYSQL2DC_DATACATALOG_LOCATION_ID=google_cloud_location_id
export MYSQL2DC_MYSQL_SERVER=mysql_server
export MYSQL2DC_MYSQL_USERNAME=mysql_username
export MYSQL2DC_MYSQL_PASSWORD=mysql_password
export MYSQL2DC_MYSQL_DATABASE=mysql_database
export MYSQL2DC_RAW_METADATA_CSV=mysql_raw_csv (If supplied ignores the MYSQL server credentials)

```

### 1.4. Docker

See instructions below.

## 2. Sample application entry point

### 2.1. Run main.py

- Virtualenv

```bash
mysql2datacatalog --datacatalog-project-id=$MYSQL2DC_DATACATALOG_PROJECT_ID --datacatalog-location-id=$MYSQL2DC_DATACATALOG_LOCATION_ID --mysql-host=$MYSQL2DC_MYSQL_SERVER --mysql-user=$MYSQL2DC_MYSQL_USERNAME --mysql-pass=$MYSQL2DC_MYSQL_PASSWORD --mysql-database=$MYSQL2DC_MYSQL_DATABASE  --raw-metadata-csv=$MYSQL2DC_RAW_METADATA_CSV      
```

- Or using Docker

```bash
docker build -t mysql2datacatalog .
docker run --rm --tty -v YOUR-CREDENTIALS_FILES_FOLDER:/data mysql2datacatalog --datacatalog-project-id=$MYSQL2DC_DATACATALOG_PROJECT_ID --datacatalog-location-id=$MYSQL2DC_DATACATALOG_LOCATION_ID --mysql-host=$MYSQL2DC_MYSQL_SERVER --mysql-user=$MYSQL2DC_MYSQL_USERNAME --mysql-pass=$MYSQL2DC_MYSQL_PASSWORD --mysql-database=$MYSQL2DC_MYSQL_DATABASE  --raw-metadata-csv=$MYSQL2DC_RAW_METADATA_CSV       
```

##### 3. Generate wheel file

```bash
python setup.py bdist_wheel
```

## 4 Scripts inside tools

### 4.1. Run clean up

```bash
# List of projects split by comma. Can be a single value without comma
export MYSQL2DC_DATACATALOG_PROJECT_IDS=my-project-1,my-project-2
```

```bash
# Run the clean up
python tools/cleanup_datacatalog.py --datacatalog-project-ids=$MYSQL2DC_DATACATALOG_PROJECT_IDS 

```

## 5. Developer environment

### 5.1. Install and run Yapf formatter

```bash
pip install --upgrade yapf

# Auto update files
yapf --in-place --recursive src tests

# Show diff
yapf --diff --recursive src tests

# Set up pre-commit hook
# From the root of your git project.
curl -o pre-commit.sh https://raw.githubusercontent.com/google/yapf/master/plugins/pre-commit.sh
chmod a+x pre-commit.sh
mv pre-commit.sh .git/hooks/pre-commit
```

### 5.2. Install and run Flake8 linter

```bash
pip install --upgrade flake8
flake8 src tests
```


### 5.3. Run Tests

```bash
pip install ./lib/datacatalog_connectors_commons_test-1.0.0-py2.py3-none-any.whl
python setup.py test
```

## 6. Metrics

[Metrics README.md](docs/README.md)

## 7. Troubleshooting

In the case a connector execution hits Data Catalog quota limit, an error will be raised and logged with the following detailement, depending on the performed operation READ/WRITE/SEARCH: 
```
status = StatusCode.RESOURCE_EXHAUSTED
details = "Quota exceeded for quota metric 'Read requests' and limit 'Read requests per minute' of service 'datacatalog.googleapis.com' for consumer 'project_number:1111111111111'."
debug_error_string = 
"{"created":"@1587396969.506556000", "description":"Error received from peer ipv4:172.217.29.42:443","file":"src/core/lib/surface/call.cc","file_line":1056,"grpc_message":"Quota exceeded for quota metric 'Read requests' and limit 'Read requests per minute' of service 'datacatalog.googleapis.com' for consumer 'project_number:1111111111111'.","grpc_status":8}"
```
For more info about Data Catalog quota, go to: [Data Catalog quota docs](https://cloud.google.com/data-catalog/docs/resources/quotas).