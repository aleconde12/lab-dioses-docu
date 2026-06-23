# Diagrama de red  — Laboratorio Ciber

Red Interna: 192.168.100.0/24

~~~ bash

                ┌────────────────────┐
                │ Cliente Windows    │
                │ DHCP / Samba test  │
                └─────────┬──────────┘
                          │
                ┌─────────┴──────────┐
                │ Internal Network   │
                │ 192.168.100.0/24   │
                └─────────┬──────────┘
                          │
     ┌────────────────────┼────────────────────┐
     │                    │                    │
┌────▼─────┐        ┌─────▼─────┐        ┌─────▼──────┐
│ ciber-web│        │ ciber-db  │        │ ciber-files│
│ .20      │        │ .10       │        │ .40        │
│ HTTP 80  │        │ MariaDB   │        │ FTP/Samba  │
│ SSH22035 │        │ SSH22035  │        │ SSH22035   │
└──────────┘        └───────────┘        └────────────┘
                          │
                    ┌─────▼──────┐
                    │ ciber-dhcp │
                    │ .30        │
                    │ DHCP       │
                    │ SSH22035   │
                    └────────────┘
~~~



|Equipo|IP|Función|Puertos relevantes|
|------|--|-------|------------------|
|ciber-db|192.168.100.10|Base de datos|3306, 22035|
|ciber-web|192.168.100.20|Aplicación web|80, 22035|
|ciber-dhcp|192.168.100.30|DHCP|67/68, 22035|
|ciber-files|192.168.100.40|FTP + Samba|21, 445, 139, 22035|
|ciber-win|DHCP|Prueba de acceso|Navegador / unidad de red|