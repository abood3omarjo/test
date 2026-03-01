# ULA Core

## Development Environment Setup

This project can be run using **Laravel Sail (Docker)** or **Local Setup (Without Docker)**.

---

# Option 1: Laravel Sail (Docker)

## 1. Clone & Install

```shell
git clone git@bitbucket.org:jo-academy/core.git
```

```shell
cd core
```

```shell
cp .env.example .env
```

```shell
composer install
```

```shell
alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'
```

## 2. Build & Run

```shell
sail build
sail up -d
sail artisan ide-helper:meta
sail artisan ide-helper:generate
```

```shell
sail artisan key:generate
```

```shell
sail npm install
```

## 3. Hosts File
```shell
echo "0.0.0.0 ula-core.site disk.ula-core.site mail.ula-core.site storage.ula-core.site" | sudo tee -a /etc/hosts
```

```shell
sail npm run dev
```

## MinIO (Sail Only)
1. Exec into your MinIO container
```shell
docker exec -it UlaCore-MinIO /bin/sh
```

2. Configure mc alias for your MinIO instance
```shell
mc alias set local http://localhost:9000 sail password
```
Replace localhost:9000 with your MinIO endpoint, and minioadmin minioadmin with your actual access/secret keys.

3. Make the bucket public
```shell
mc anonymous set download local/local
```

4. Verify the policy
```shell
mc anonymous list local/local
```

Go to https://disk.ula-core.site/ abd login with the following credentials:
user:sail
password:password

create these buckets:

- local
- test
- media
- media-test

```shell
docker cp UlaCore-Caddy:/config/caddy/pki/authorities/local/root.crt ~/.ssh/caddy-root-ula-core.crt
```

Mac users

```shell
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain ~/.ssh/caddy-root-ula-core.crt
```

Ubuntu Users

```shell
sudo cp ~/.ssh/caddy-root-ula-core.crt /usr/local/share/ca-certificates/caddy-root-ula-core.crt
sudo update-ca-certificates
```

add cert exception for chrome
`chrome://certificate-manager/localcerts/usercerts`

## Git setup

```shell
git config core.fileMode false
```

## Before any commit

```shell
sail artisan ide-helper:model
```

```shell
sail pint
```

```shell
sail npm run lint
```

```shell
sail npm run types
```

```shell
sail npm run format
```

## Option 2: Local Setup (Without Docker)
(Requires PHP 8.4+ and PostgreSQL)

## 1. Clone & Install

```shell
git clone git@bitbucket.org:jo-academy/core.git
cd core
cp .env.example .env
```

```shell
composer install --ignore-platform-req=ext-mongodb
```

## 2. PostgreSQL Setup

```shell
sudo service postgresql start
sudo -u postgres psql
CREATE USER sail WITH PASSWORD 'password';
```

```shell
CREATE DATABASE ula_core;
CREATE DATABASE assessments;
```

```shell
GRANT ALL PRIVILEGES ON DATABASE ula_core TO sail;
GRANT ALL PRIVILEGES ON DATABASE assessments TO sail;
```

```shell
\q
```

```shell
\c ula_core
ALTER SCHEMA public OWNER TO sail;
GRANT ALL ON SCHEMA public TO sail;
```

```shell
\c assessments
ALTER SCHEMA public OWNER TO sail;
GRANT ALL ON SCHEMA public TO sail;
```

```shell
\q
```

## 3. Update pg_hba.conf

```shell
sudo nano /etc/postgresql/16/main/pg_hba.conf
```

## Change to:

```shell
host    all    all    127.0.0.1/32    trust
```

## Restart:
```shell
sudo systemctl restart postgresql
```

## 4. Update .env (Local)
```shell
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=ula_core
DB_USERNAME=sail
DB_PASSWORD=password
```

```shell
ASSESSMENTS_DB_CONNECTION=pgsql
ASSESSMENTS_DB_HOST=127.0.0.1
ASSESSMENTS_DB_PORT=5432
ASSESSMENTS_DB_DATABASE=assessments
ASSESSMENTS_DB_USERNAME=sail
ASSESSMENTS_DB_PASSWORD=password
```

```shell
CACHE_STORE=file
SESSION_DRIVER=file
TELESCOPE_DB_CONNECTION=pgsql
```

## change APP_SERVICE 
```shell
APP_SERVICE=ula-code.site
```

## TO
```shell
APP_SERVICE=ula-core.site
```

## 5. Run Locally
```shell
rm bootstrap/cache/config.php
```

```shell
php artisan config:clear
php artisan key:generate
php artisan migrate
php artisan ide-helper:meta
php artisan ide-helper:generate
```

```shell
php artisan serve
```
