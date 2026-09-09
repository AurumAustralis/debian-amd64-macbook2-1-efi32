<div align="center">
  <img src="macbook21.png" alt="Apple MacBook2,1" width="650">
</div>

<p align="center">
  <strong>Español</strong> · <a href="README.md">English</a>
</p>

---

# Debian 64 bits en MacBook2,1 con EFI de 32 bits usando Ventoy

Guía práctica para instalar **Debian GNU/Linux de 64 bits (`amd64`)** en una **Apple MacBook 2.1** de 2006/2007, equipada con un procesador Intel Core 2 Duo de 64 bits pero con firmware **EFI de 32 bits**.

El procedimiento utiliza **Ventoy** para crear un pendrive capaz de arrancar en modo **IA32 UEFI** y cargar la imagen normal de Debian `amd64`.

> [!IMPORTANT]
> Esta guía está basada en una instalación realizada y probada sobre una **MacBook2,1**. Ventoy considera experimental su soporte IA32 UEFI, por lo que el comportamiento puede variar en otros equipos Apple antiguos.

---

## El problema

La MacBook2,1 tiene una combinación poco habitual:

| Componente | Arquitectura |
|---|---|
| Procesador Intel Core 2 Duo | 64 bits |
| Sistema operativo que puede ejecutar | 64 bits (`amd64`) |
| Firmware EFI de la Mac | 32 bits (`IA32`) |

Por este motivo, un pendrive preparado de la forma habitual puede no aparecer en el administrador de arranque de la Mac o puede fallar antes de iniciar el instalador.

Ventoy incorpora soporte para **IA32 UEFI** y permite utilizar una ISO Debian `amd64` sin tener que construir manualmente un cargador GRUB EFI de 32 bits.

---

## Hardware probado

- Apple MacBook2,1.
- Procesador Intel Core 2 Duo.
- EFI de 32 bits.
- Pendrive USB 2.0.
- Debian `amd64`.
- Ventoy.
- Escritorio Xfce.

> En la prueba realizada, **Xfce funcionó correctamente**. Con **LXQt** se produjo un bucle en la pantalla de inicio de sesión. Esto se documenta como resultado de esta instalación concreta y no como un fallo general de LXQt.

---

## Qué se necesita

1. Una MacBook2,1.
2. Un pendrive USB. Se recomienda **USB 2.0** para estos equipos antiguos.
3. Otra computadora con:
   - Linux, o
   - Windows.
4. Ventoy.
5. Una imagen oficial de Debian para arquitectura `amd64`.

### Descargas

- Ventoy: https://www.ventoy.net/en/download.html
- Debian: https://www.debian.org/download

Para una instalación por red se puede utilizar la imagen:

```text
debian-13.x.x-amd64-netinst.iso
```

La versión exacta cambiará con las nuevas publicaciones de Debian.

> [!NOTE]
> Para este procedimiento se recomienda la ISO normal **`amd64`**. No es necesario utilizar la imagen `debian-mac-...-amd64-netinst.iso`.

---

# Método 1: preparar el pendrive desde Linux

## 1. Descargar Ventoy

Descargar la versión de Ventoy para Linux y descomprimirla.

Por ejemplo:

```bash
tar -xzf ventoy-x.x.xx-linux.tar.gz
cd ventoy-x.x.xx
```

---

## 2. Identificar el pendrive

Conectar el pendrive y ejecutar:

```bash
lsblk -d -e 7,11 -o NAME,SIZE,MODEL,TRAN
```

Ejemplo:

```text
NAME   SIZE MODEL             TRAN
sda    477G SSD
sdb   14.9G USB Flash Drive   usb
```

En este ejemplo el pendrive es:

```text
/dev/sdb
```

> [!CAUTION]
> Verificar cuidadosamente cuál es el dispositivo USB. Elegir el disco equivocado puede destruir los datos de otro disco.

---

## 3. Instalar Ventoy en el pendrive

Desde el directorio donde se descomprimió Ventoy:

```bash
sudo ./Ventoy2Disk.sh -i /dev/sdX
```

Reemplazar `/dev/sdX` por el dispositivo correspondiente.

Ejemplo:

```bash
sudo ./Ventoy2Disk.sh -i /dev/sdb
```

Ventoy solicitará confirmación antes de modificar el pendrive.

> [!WARNING]
> La instalación normal de Ventoy formatea el dispositivo y elimina su contenido.

El comando anterior utiliza el esquema de particiones **MBR**, que es el predeterminado de Ventoy y es el utilizado en este procedimiento.

---

## 4. Copiar la ISO de Debian

Una vez instalado Ventoy, desconectar y volver a conectar el pendrive si fuera necesario.

Aparecerá una partición llamada normalmente:

```text
Ventoy
```

Copiar directamente dentro de ella la ISO de Debian:

```text
debian-13.x.x-amd64-netinst.iso
```

No hay que extraer la ISO ni grabarla con `dd`.

Ventoy se encargará de detectar el archivo ISO y mostrarlo en su menú de arranque.

---

# Método 2: preparar el pendrive desde Windows

Este método es más sencillo porque Ventoy dispone de una interfaz gráfica.

## 1. Descargar Ventoy para Windows

Descargar el archivo:

```text
ventoy-x.x.xx-windows.zip
```

Descomprimirlo en una carpeta.

---

## 2. Ejecutar Ventoy

Dentro de la carpeta ejecutar:

```text
Ventoy2Disk.exe
```

Es recomendable abrirlo con **Ejecutar como administrador**.

---

## 3. Seleccionar el pendrive

En **Device**, elegir cuidadosamente el pendrive USB que se utilizará para instalar Debian.

Mantener el esquema de particiones en **MBR**.

Pulsar:

```text
Install
```

Ventoy mostrará advertencias indicando que se borrará el contenido del pendrive.

Confirmar únicamente después de verificar que se seleccionó el dispositivo correcto.

---

## 4. Copiar Debian al pendrive

Cuando Ventoy termine, Windows mostrará una nueva unidad llamada normalmente:

```text
Ventoy
```

Abrirla desde el Explorador de archivos y copiar dentro la ISO:

```text
debian-13.x.x-amd64-netinst.iso
```

Eso es todo. No hace falta utilizar Rufus, `dd`, Etcher ni extraer el contenido de la ISO.

---

# Arrancar la MacBook2,1

## 1. Conectar el pendrive

Con la MacBook apagada, conectar el pendrive Ventoy.

---

## 2. Abrir el administrador de arranque

Encender la Mac e inmediatamente mantener presionada la tecla:

```text
Option / Alt (⌥)
```

Mantenerla presionada hasta que aparezca el administrador de arranque de Apple.

Debería aparecer el dispositivo USB como una opción de arranque.

Seleccionarlo y presionar **Enter**.

---

## 3. Comprobar que Ventoy inició en IA32

Cuando aparezca el menú de Ventoy, mirar la información que se muestra en la parte inferior de la pantalla.

En esta Mac el modo esperado es:

```text
IA32
```

Esto significa que Ventoy ha sido arrancado mediante el firmware EFI de 32 bits de la MacBook.

---

## 4. Seleccionar Debian

En el menú de Ventoy seleccionar:

```text
debian-13.x.x-amd64-netinst.iso
```

Presionar **Enter** y continuar con el instalador de Debian.

Aunque el firmware EFI sea de 32 bits, el sistema Debian que se instalará será de **64 bits (`amd64`)**, porque el procesador Core 2 Duo de la MacBook2,1 soporta x86-64.

---

# Instalación de Debian

Una vez iniciado el instalador, el procedimiento es prácticamente el mismo que en cualquier PC:

1. Seleccionar idioma.
2. Configurar teclado.
3. Configurar red.
4. Crear usuario y contraseña.
5. Particionar el disco según el tipo de instalación deseada.
6. Instalar el sistema base.
7. Instalar GRUB cuando lo solicite Debian.
8. Seleccionar el entorno de escritorio.

Esta guía se concentra en resolver el problema particular de **arrancar un instalador Debian de 64 bits desde una EFI de 32 bits**. El particionado dependerá de si Debian reemplazará completamente a macOS o se instalará junto a otro sistema.

---

# Escritorio recomendado

Para este hardware se recomienda:

```text
Xfce
```

Es un entorno de escritorio relativamente liviano y fue el utilizado correctamente durante la prueba.

Durante la instalación se puede seleccionar **Xfce** desde la selección de software de Debian.

En la prueba realizada, LXQt presentó un bucle de inicio de sesión, por lo que se utilizó Xfce.

---

# Verificar que Debian quedó instalado en 64 bits

Después de iniciar Debian, abrir una terminal y ejecutar:

```bash
dpkg --print-architecture
```

El resultado esperado es:

```text
amd64
```

También se puede comprobar el kernel:

```bash
uname -m
```

Resultado esperado:

```text
x86_64
```

Para comprobar si el sistema arrancó mediante EFI:

```bash
if [ -d /sys/firmware/efi ]; then
    echo "Sistema iniciado mediante EFI"
else
    echo "Sistema iniciado sin EFI"
fi
```

Y para comprobar los paquetes de GRUB EFI de 32 bits:

```bash
dpkg -l | grep grub-efi-ia32
```

---

# Solución de problemas

## El pendrive no aparece al mantener Option/Alt

Apagar completamente la Mac, conectar nuevamente el pendrive y repetir el arranque manteniendo:

```text
Option / Alt (⌥)
```

Si continúa sin aparecer, probar un **pendrive USB 2.0** y otro puerto USB.

También puede ser necesario restablecer la NVRAM/PRAM.

---

## Reset de NVRAM / PRAM

Apagar la Mac.

Encenderla y mantener presionadas simultáneamente:

```text
Command (⌘) + Option (⌥) + P + R
```

Después volver a intentar el arranque con el pendrive conectado.

---

## Reset de SMC

La MacBook2,1 normalmente utiliza batería removible.

### Mac con batería removible

1. Apagar la Mac.
2. Desconectar el cargador.
3. Retirar la batería.
4. Mantener presionado el botón de encendido durante aproximadamente **5 segundos**.
5. Volver a colocar la batería.
6. Conectar el cargador.
7. Encender normalmente.

Si se está aplicando esta guía a otro modelo Intel con batería no removible, el procedimiento de restablecimiento del SMC puede ser diferente y debe verificarse para ese modelo concreto.

---

## Ventoy arranca pero Debian no inicia

Comprobar primero que:

- se descargó una ISO para arquitectura `amd64`;
- la ISO se copió como archivo al pendrive Ventoy;
- Ventoy muestra `IA32` como modo de arranque;
- se utiliza una versión reciente de Ventoy;
- la ISO no está corrupta.

Puede verificarse la suma de comprobación de la ISO utilizando los archivos SHA publicados por Debian.

---

## Debian inicia pero después no arranca desde el disco interno

El instalador actual de Debian dispone de soporte para sistemas con CPU de 64 bits y firmware UEFI de 32 bits y puede instalar el GRUB EFI apropiado.

Si se produce un problema, iniciar nuevamente desde el pendrive Ventoy y utilizar el modo de rescate de Debian para revisar la partición EFI y la instalación de GRUB.

---

# Por qué funciona

La MacBook2,1 utiliza un procesador de 64 bits, pero Apple incorporó en este modelo firmware EFI de 32 bits.

El problema no está en la capacidad del procesador para ejecutar Debian de 64 bits, sino en la primera etapa del arranque.

La cadena utilizada es, conceptualmente:

```text
MacBook2,1
    │
    ├── EFI Apple de 32 bits
    │
    ▼
Ventoy IA32 UEFI
    │
    ▼
Instalador Debian amd64
    │
    ▼
Debian GNU/Linux de 64 bits
```

Ventoy actúa como puente entre el firmware IA32 de la Mac y el instalador de Debian `amd64`.

---

# Referencias

- Ventoy - Getting Started: https://www.ventoy.net/en/doc_start.html
- Ventoy - IA32 UEFI Support: https://www.ventoy.net/en/doc_ia32.html
- Debian - Descargar Debian: https://www.debian.org/download
- Debian Wiki - UEFI: https://wiki.debian.org/UEFI
- Debian Wiki - Installing Debian on MacBook2,1: https://wiki.debian.org/InstallingDebianOn/Apple/MacBook/2-1
- Apple - Combinaciones de teclas de arranque: https://support.apple.com/es-la/102603

---

# Resultado

El objetivo final es obtener:

```text
Apple MacBook2,1
CPU x86-64
EFI IA32 de 32 bits
Debian amd64 de 64 bits
Xfce
```

sin necesidad de modificar manualmente una ISO de Debian ni construir manualmente un cargador GRUB EFI de 32 bits para el pendrive de instalación.

---

## Licencia

Este procedimiento puede utilizarse, modificarse y compartirse libremente con fines educativos y técnicos.

Si encontrás mejoras o diferencias al realizar el procedimiento en otra Mac Intel con EFI de 32 bits, podés documentarlas mediante un *issue* o *pull request*.
