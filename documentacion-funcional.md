# Documentación funcional — Laboratorio Ciber

## 1. Introducción

> El proyecto representa la infraestructura básica de un ciber, compuesto por servidores Linux y un cliente Windows. La solución incluye una aplicación web para gestión de clientes y carga de horas, una base de datos, un servidor de archivos accesible desde Windows, un servidor FTP para backups y un servidor DHCP para asignación de red.

## 2. Objetivo del sistema

> El objetivo es implementar una infraestructura funcional, segura y segmentada, que permita administrar usuarios/clientes del ciber, almacenar información en una base de datos, compartir archivos en red, realizar backups automáticos y centralizar servicios básicos de red.

>El presente laboratorio simula la infraestructura base de un ciber, haciendo foco en los servicios administrativos y de red necesarios para su funcionamiento: aplicación web de gestión, base de datos, servidor de archivos, servicio FTP, backups automáticos, acceso SSH administrativo y conectividad entre servidores.

>La ejecución de videojuegos o aplicaciones gráficas en las estaciones cliente queda fuera del alcance del proyecto, ya que el objetivo principal es validar la integración de servicios de infraestructura y no el rendimiento gráfico de los equipos cliente. La PC Windows se utiliza como equipo de prueba para validar conectividad, resolución de nombres y acceso al recurso compartido del servidor de archivos.

## 3. Alcance funcional

**Incluye:**

* Aplicación web accesible por HTTP.
* Login de usuarios.
* ABM básico de clientes.
* Carga de horas para clientes del ciber.
* Base de datos persistente.
* Backups automáticos de web y base de datos.
* Retención de los últimos 5 backups.
* Envío del backup al servidor FTP.
* Servidor de archivos accesible desde Windows.
* Acceso SSH administrativo por puerto 22035.

**No incluye:**

* Facturación real.
* Integración con medios de pago.
* Alta disponibilidad.
* Certificados HTTPS.
* Monitoreo avanzado.
* Administración desde Internet pública.
* Ejecución de videojuegos en las PCs cliente.
* Evaluación de rendimiento gráfico.
* Instalación de software comercial para juegos.

## 4. Roles funcionales

### Administrador

Tiene acceso a la infraestructura, puede administrar servidores, revisar backups, conectarse por SSH y validar servicios.

### Empleado del ciber

Usa la aplicación web para gestionar clientes y cargar horas.

### Cliente Windows

Representa una PC del ciber o una PC administrativa desde donde se puede mapear la unidad de red compartida.

## 5. Servicios implementados


| Servidor      | Nombre        | Función                                              |
| ------------- | ------------- | ---------------------------------------------------- |
| Base de datos | `ciber-db`    | Almacena clientes, usuarios y horas                  |
| Web           | `ciber-web`   | Publica la aplicación web por HTTP                   |
| DHCP          | `ciber-dhcp`  | Servicio de asignación/red del laboratorio           |
| Files/FTP     | `ciber-files` | Comparte archivos por Samba y recibe backups por FTP |
| Cliente       | `ciber-win`   | Prueba acceso a recursos compartidos                 |

## 6. Funcionalidades principales

### Aplicación web

* Permite iniciar sesión.
* Permite registrar clientes.
* Permite consultar clientes existentes.
* Permite cargar horas de uso.
* Se conecta a la base de datos del servidor `ciber-db`.

### Base de datos

* Guarda la información usada por la aplicación.
* Solo debe ser accedida desde el servidor web.
* No expone servicios innecesarios hacia otros equipos.

### Backups

* Se ejecutan automáticamente mediante `cron`.
* Generan copia de la web y de la base de datos.
* El nombre del archivo contiene la fecha.
* Se conservan los últimos 5 backups.
* Luego de generarse, se suben al servidor FTP.

### File Server

* Permite alojar archivos compartidos.
* Permite mapear una unidad de red desde Windows.
* Usa autenticación de usuario Samba.

### Acceso administrativo

* Todos los servidores permiten acceso SSH.
* El puerto utilizado para SSH es `22035`.
* El servidor web solo expone HTTP `80` y SSH `22035`.

## 7. Flujo funcional del sistema


```text
Usuario/Empleado
      |
      v
Aplicación Web - ciber-web
      |
      v
Base de Datos - ciber-db

Backups automáticos:
ciber-web / ciber-db
      |
      v
FTP - ciber-files

Cliente Windows:
Windows
      |
      v
Unidad de red Samba - ciber-files
```

## 8. Casos de uso básicos


### Caso de uso 1: Iniciar sesión

**Actor:** Empleado
**Descripción:** El empleado ingresa usuario y contraseña en la aplicación web.
**Resultado esperado:** Accede al panel principal.

### Caso de uso 2: Registrar cliente

**Actor:** Empleado
**Descripción:** El empleado carga los datos de un nuevo cliente.
**Resultado esperado:** El cliente queda guardado en la base de datos.

### Caso de uso 3: Cargar horas

**Actor:** Empleado
**Descripción:** El empleado selecciona un cliente y registra horas de uso.
**Resultado esperado:** La carga queda registrada en la base de datos.

### Caso de uso 4: Acceder a archivos compartidos

**Actor:** Cliente Windows / Administrador
**Descripción:** Desde Windows se accede al recurso compartido del servidor de archivos.
**Resultado esperado:** Se puede leer y alojar archivos en la carpeta compartida.

### Caso de uso 5: Generación de backup

**Actor:** Sistema
**Descripción:** El cron ejecuta el script de backup diario.
**Resultado esperado:** Se genera un archivo con fecha, se conserva la rotación de 5 backups y se sube al FTP.

## 9. Validaciones funcionales

| Validación                 | Resultado esperado                    |
| -------------------------- | ------------------------------------- |
| Acceso web por puerto 80   | La aplicación carga correctamente     |
| Login en la aplicación     | El usuario accede al sistema          |
| Alta de cliente            | El cliente queda registrado           |
| Conexión web → DB          | La app consulta la base correctamente |
| Backup manual o por cron   | Se genera archivo con fecha           |
| Retención de backups       | Solo quedan los últimos 5             |
| Subida por FTP             | El backup aparece en `ciber-files`    |
| Acceso Samba desde Windows | Se puede mapear la unidad             |
| SSH por puerto 22035       | Se puede administrar cada servidor    |
| Puertos bloqueados en web  | Solo quedan permitidos 80 y 22035     |

## 10. Conclusión funcional

> La solución implementada cumple con los requerimientos funcionales solicitados para el laboratorio, integrando servicios web, base de datos, backup automático, transferencia FTP, servidor de archivos y acceso administrativo seguro. El diseño permite representar una infraestructura realista para un ciber, manteniendo separación de roles entre servidores y controlando los accesos según la consigna.

---

