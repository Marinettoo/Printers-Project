# Proyecto Servidor de Correo

# "Old school email configuration"
## Características del Proyecto
Este proyecto está compuesto por una infraestructura de correo electrónico completa dividida en dos máquinas virtuales. Una servidora de nombres (DNS) para la resolución de dominios, y otra que actúa como Servidor de Correo (MTA/MDA) usando Postfix y Dovecot para el envío y recepción de emails.

## Requisitos
* [Vagrant](https://developer.hashicorp.com/vagrant/install)
* [Oracle Virtualbox](https://www.virtualbox.org/wiki/Downloads)
* [Mozilla Thunderbird](https://www.thunderbird.net/es-ES/)

---

**[Enunciados del Proyecto](https://github.com/Marinettoo/Printers-Project)**

---

## Servidor DNS (Infraestructura)
[Configuración de Red]

En este servidor, la configuración permite la resolución de nombres del dominio `example.test`, esencial para que los correos sepan a dónde dirigirse.

* 🖥️ **Máquina:** `dns.example.test`
* 🌐 **IP:** `192.168.57.10`

### ¿Cómo desplegarlo?
* Abriremos la terminal e iremos a la carpeta del proyecto.
* Usaremos el comando `vagrant up` para arrancar las máquinas virtuales (DNS y Servidor).
    * Se instalarán automáticamente las dependencias y configuraciones mediante el script de aprovisionamiento.
* Una vez hayan arrancado, configuraremos nuestro ordenador anfitrión (Linux/Windows) para que use este DNS:
    * Editaremos `/etc/resolv.conf` añadiendo `nameserver 192.168.57.10` al principio.

### ¿Cómo comprobarlo?
* **Usando la terminal de linux (Ping)**
    * Usaremos el comando `ping -c 3 srv.example.test`.
    * Si la resolución es correcta, deberíamos ver respuesta desde la IP `192.168.57.20`.

### ¿Qué funciona?

| Acción | Resultado Esperado |
| :--- | :--- |
| **Resolución DNS** | El comando `host srv.example.test` devuelve `192.168.57.20`. |
| **Registros MX** | El dominio sabe que el correo lo gestiona `srv.example.test`. |
| **Conectividad** | Las máquinas se ven entre sí y desde el anfitrión. |

---

## Servidor de Correo (Postfix + Dovecot)
[Configuración del Servicio]

En este servidor, la configuración implementa los protocolos SMTP (envío) e IMAP/POP3 (recepción), permitiendo a los usuarios intercambiar mensajes y almacenarlos en formato Maildir.

* 🖥️ **Máquina:** `srv.example.test`
* 🌐 **IP:** `192.168.57.20`

### Usuarios Configurados

| Usuario | Contraseña | Dirección de Correo |
| :--- | :---: | :--- |
| **Usuario 1** | `asir` | `usuario1@example.test` |
| **Usuario 2** | `asir` | `usuario2@example.test` |

### ¿Cómo conectarse?

Usaremos un cliente de correo de escritorio para las pruebas.

1. **Usando Mozilla Thunderbird**
    * Abriremos Thunderbird e iremos a **Configuración de cuenta > Añadir cuenta de correo**.
    * Rellenaremos los datos de usuario y pulsaremos en "Configurar manualmente".
    * Configuraremos la conexión así:


* **Protocolo IMAP:** Puerto `143` | Servidor: `srv.example.test` | SSL: `Ninguna` / `STARTTLS`.
* **Protocolo SMTP:** Puerto `25` | Servidor: `srv.example.test` | SSL: `Ninguna` / `STARTTLS`.
* **Autenticación:** `Contraseña normal`.
* Al conectar, saltará un **Aviso de Seguridad**:
    * Esto es normal porque no usamos certificados oficiales firmados por una CA pública.
    * Marcaremos "Confirmar excepción de seguridad" y aceptaremos.

**Si todo ha ido bien, veremos la bandeja de entrada vacía lista para recibir correos.**

### ¿Qué funciona?

| Acción | Resultado Esperado |
| :--- | :--- |
| **Envío (SMTP)** | Podemos enviar un correo desde Usuario 1 a Usuario 2. |
| **Recepción (IMAP)** | El Usuario 2 recibe el correo en su bandeja de entrada. |
| **Almacenamiento** | Los correos se guardan en el servidor en `~/Maildir`. |
| **Autenticación** | Solo se puede acceder con la contraseña correcta (`asir`). |

---

## 📜 Guía Técnica de Instalación (Comandos y Configuraciones)

A continuación se detallan los comandos reales utilizados dentro de las máquinas virtuales para lograr la configuración.

### 1. Instalación del Servidor DNS (`dns`)
Accedemos a la máquina con `vagrant ssh dns` y nos convertimos en root con `sudo su`.

**Comandos de instalación:**
```bash
apt update && apt install bind9 -y

```

**Edición de Archivos:**

* `/etc/bind/named.conf.local`: Definimos la zona maestra `example.test`.
* `/etc/bind/db.example.test`: Creamos el archivo de zona con los registros A, CNAME y MX.

**Reinicio del servicio:**

```bash
systemctl restart bind9

```

### 2. Instalación del Servidor de Correo (`srv`)

Accedemos a la máquina con `vagrant ssh srv` y nos convertimos en root.

**Instalación de paquetes:**
Ejecutamos el comando para instalar Postfix y Dovecot:

```bash
apt update && apt install postfix dovecot-imapd dovecot-pop3d -y

```

**🖥️ Pantallas de Configuración (Debconf):**
Durante la instalación de Postfix, aparecieron las siguientes ventanas de configuración:

1. **Tipo de configuración de correo:** Elegimos `Sitio de Internet` ("Internet Site") para habilitar SMTP estándar.
2. **Nombre del sistema de correo:** Escribimos `example.test`.

**Configuración de Postfix (`main.cf`):**
Editamos la configuración principal con el comando:

```bash
nano /etc/postfix/main.cf

```

Modificamos y añadimos las siguientes líneas clave:

```conf
home_mailbox = Maildir/

mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.57.0/24

```

**Reinicio de Postfix:**

```bash
systemctl restart postfix

```

**Configuración de Dovecot:**
Editamos los archivos en `/etc/dovecot/conf.d/` para indicar la ubicación del correo y permitir la autenticación:

```conf
mail_location = maildir:~/Maildir

disable_plaintext_auth = no

```

---

### Tarea Adicional: Pruebas de Envío

Hemos realizado una prueba de fuego enviando un correo real entre las dos cuentas configuradas.

**¿Cómo comprobar que funciona?**

1. **Envío de correo:** Desde la cuenta de `usuario1`, redactamos un correo nuevo para `usuario2@example.test`.
2. **Recepción del correo:** Entramos en la bandeja de entrada de `usuario2` y pulsamos "Recibir mensajes".

| Acción | Resultado Esperado |
| --- | --- |
| **Bandeja de Entrada** | Aparece el correo nuevo con el asunto "Prueba". |
| **Cabeceras** | Se verifica que el remitente es `usuario1@example.test`. |

```

```
