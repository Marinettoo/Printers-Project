# Proyecto Servidor de Correo
## Características del Proyecto

Este proyecto está compuesto por una infraestructura de correo electrónico completa dividida en dos máquinas virtuales. Una servidora de nombres (DNS) para la resolución de dominios, y otra que actúa como Servidor de Correo (MTA/MDA) usando Postfix y Dovecot para el envío y recepción de emails.

## Requisitos
* [Vagrant](https://developer.hashicorp.com/vagrant/install)
* [Oracle Virtualbox](https://www.virtualbox.org/wiki/Downloads)
* [Mozilla Thunderbird](https://www.thunderbird.net/es-ES/)

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

- **Usando Mozilla Thunderbird**
    * Abriremos Thunderbird e iremos a **Configuración de cuenta > Añadir cuenta de correo**.
    * Rellenaeremos los datos de usuario y pulsaremos en "Configurar manualmente".
    * Configuraremos la conexión así:


* **Protocolo IMAP:** Puerto `143` | Servidor: `srv.example.test` | SSL: `Ninguna` / `STARTTLS`.
* **Protocolo SMTP:** Puerto `25` | Servidor: `srv.example.test` | SSL: `Ninguna` / `STARTTLS`.
* **Autenticación:** `Contraseña normal`.

* Al conectar, saltará un **Aviso de Seguridad**
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

## Tarea Adicional: Pruebas de Envío
Hemos realizado una prueba de fuego enviando un correo real entre las dos cuentas configuradas.

### ¿Cómo comprobar que funciona?

**Envío de correo**
Desde la cuenta de `usuario1`, redactamos un correo nuevo para `usuario2@example.test`.

**Recepción del correo**
Entramos en la bandeja de entrada de `usuario2` y pulsamos "Recibir mensajes".

| Acción | Resultado Esperado |
| :--- | :--- |
| **Bandeja de Entrada** | Aparece el correo nuevo con el asunto "Prueba". |
| **Cabeceras** | Se verifica que el remitente es `usuario1@example.test`. |


---