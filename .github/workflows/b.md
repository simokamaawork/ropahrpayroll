name: Laravel

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  laravel-tests:
    runs-on: ubuntu-latest

    services:
      mysql:
        image: mysql:5.7
        env:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: laravel
        ports:
          - 3306:3306
        options: >-
          --health-cmd="mysqladmin ping --silent"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=3

    steps:
    - uses: actions/checkout@v4
    - name: Setup PHP
      uses: shivammathur/setup-php@v2
      with:
        php-version: '8.1'
        extensions: bcmath, ctype, fileinfo, json, mbstring, openssl, pdo, tokenizer, xml
    - name: Copy .env
      run: php -r "file_exists('.env') || copy('.env.example', '.env');"
    - name: Install Dependencies
      run: composer install --prefer-dist --no-interaction --no-scripts --no-progress
    - name: Generate Key
      run: php artisan key:generate
    - name: Set Permissions
      run: chmod -R 777 storage bootstrap/cache
    - name: Create Database
      run: |
        mysql -u root -proot -e 'CREATE DATABASE IF NOT EXISTS laravel;'
        php artisan migrate --force
    - name: Run Tests
      env:
        DB_CONNECTION: mysql
        DB_HOST: 127.0.0.1
        DB_PORT: 3306
        DB_DATABASE: laravel
        DB_USERNAME: root
        DB_PASSWORD: root
      run: php artisan test
