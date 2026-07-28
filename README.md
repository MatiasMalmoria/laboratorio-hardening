# Reporte Técnico de Configuración de Laboratorio
## 1. La Fundación: VirtualBox y Red Aislada

<img width="812" height="514" alt="01-red-nat" src="https://github.com/user-attachments/assets/8f2e75ce-5364-478a-a1cf-142944b506ca" />

Configuré el adaptador de red de la VM en modo **NAT**. En este modo, la máquina virtual accede a internet a través de la IP del equipo anfitrión (Host), pero no es visible ni direccionable desde la red local (Wi-Fi/LAN) en la que está conectado el Host.

**¿Por qué NAT y no Bridge (Puente)?**

- **Modo Bridge:** la VM obtiene su propia IP dentro de la red local, como si fuera un dispositivo físico más. Cualquier otro equipo de esa red podría intentar conectarse a ella y escanear sus puertos.
- **Modo NAT:** la VM queda detrás de un "NAT virtual" que gestiona VirtualBox. Sale a internet para actualizar paquetes, pero desde la red local es invisible: no tiene una IP propia alcanzable por otros dispositivos.
- Esto es crítico porque una máquina de prácticas de ciberseguridad suele tener herramientas de pentesting, vulnerabilidades intencionales o configuraciones débiles. Si estuviera en modo Bridge, sería un punto de entrada expuesto en la red doméstica u corporativa.

---

## 2. Capa Windows: Usuarios y Actualizaciones

### 2.1 Usuario estándar separado del Administrador

<img width="1293" height="847" alt="02-cuentas-usuario" src="https://github.com/user-attachments/assets/a74aee71-947c-4236-8e03-ffcf547b9e72" />

Creé una cuenta de usuario ("UsuarioSeguro") de tipo Estándar, distinta de las cuentas Administrador ("vboxuser" y "admin") que existen en el sistema. El trabajo diario dentro de la VM (abrir herramientas, navegar, ejecutar prácticas) se realiza con la cuenta Estándar.

**¿Por qué es importante?**

- Trabajar como Administrador todo el tiempo es el error más común de un principiante: cualquier programa malicioso, script o herramienta mal usada tiene automáticamente permisos totales sobre el sistema.
- Con una cuenta Estándar, si algo sale mal (malware, error de configuración, herramienta de prueba con comportamiento inesperado), el daño queda limitado porque el sistema pide credenciales de administrador para cambios críticos (control de cuentas de usuario, UAC).

### 2.2 Sistema actualizado

<img width="1102" height="913" alt="03-windows-update" src="https://github.com/user-attachments/assets/62f6a198-9f99-48c8-ae41-4ae3b6c93494" />

Ejecuté Windows Update y confirmé que el sistema no tiene actualizaciones de seguridad pendientes. Mantener el sistema parcheado reduce la superficie de ataque, ya que muchas vulnerabilidades explotadas en la práctica son fallas ya corregidas por el fabricante.

---

## 3. Capa Linux: Permisos y Gestión

### 3.1 Permisos de archivo con `ls -l` / 3.2 Búsqueda de actualizaciones de paquetes

Trabajando con el usuario `practicante` (sin privilegios de root), ejecuté los siguientes comandos en la terminal de Kali Linux:

```bash
practicante@kali:~$ ls -l practica.txt
-rw-rw-r-- 1 practicante practicante 0 Jul 27 19:31 practica.txt

practicante@kali:~$ sudo apt update
[sudo] password for practicante:
Hit:1 http://http.kali.org/kali kali-rolling InRelease
808 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

<img width="1560" height="803" alt="04-linux-permisos-apt" src="https://github.com/user-attachments/assets/be054cdb-958e-4877-b375-f6a3254a3c34" />

**Interpretación de `ls -l`:** el primer carácter (`-`) indica que es un archivo regular. Los siguientes nueve caracteres se agrupan de a tres: `rw-` para el dueño (lectura y escritura), `rw-` para el grupo (lectura y escritura) y `r--` para el resto de los usuarios (solo lectura). Le siguen el usuario y grupo propietarios (`practicante`), el tamaño en bytes y la fecha de modificación.

El comando `sudo apt update` consulta los repositorios configurados y actualiza la lista local de versiones disponibles. Se ejecuta con `sudo` —solicitando la contraseña del usuario `practicante`, como se ve en el prompt `[sudo] password for practicante:`— porque modificar el índice de paquetes del sistema requiere privilegios elevados, pero de forma puntual y controlada, no logueado como root permanentemente. Esto evita justamente el error #1 del principiante: trabajar siempre como root.

---

## 4. La Red de Seguridad: Snapshot Inicial

<img width="1917" height="1027" alt="05-snapshot" src="https://github.com/user-attachments/assets/e9fa4607-d670-44ae-9bd0-12ae6541372d" />

Con cada VM apagada, luego de aplicar todas las medidas anteriores (red NAT, usuario estándar, sistema actualizado), creé en ambas una instantánea (snapshot) llamada **"hardening inicial"** desde el Administrador de Instantáneas de VirtualBox.

**¿Por qué es indispensable?**

- Un snapshot es un punto de restauración exacto del estado de la VM. Si durante una práctica de ciberseguridad algo se rompe, se instala malware de prueba, o se modifica una configuración crítica por error, puedo volver en segundos a este estado limpio y verificado.
- Es la "red de seguridad" que permite experimentar con confianza: probar herramientas potencialmente riesgosas sabiendo que siempre existe un camino de vuelta a un estado conocido y seguro.

---

## 5. Conclusión

La combinación de aislamiento de red (NAT), separación de privilegios (usuario estándar en Windows, usuario `practicante` en Linux), sistema parcheado y un snapshot de restauración en cada máquina constituye el hardening mínimo recomendado antes de comenzar cualquier práctica de ciberseguridad. Estas cuatro capas se refuerzan entre sí: si una falla, las otras limitan el impacto.
