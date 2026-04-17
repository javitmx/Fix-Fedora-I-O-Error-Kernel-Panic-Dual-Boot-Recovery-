# Fix:Fedora I/O Error Kernel Panic Dual Boot Recovery
Este Write-up documenta la resolución de un error crítico de entrada/salida (I/O Error) en un sistema Fedora Linux con Dual Boot, que derivó en un Kernel Panic, y la implementación de un sistema de recuperación ante desastres mediante Btrfs Snapshots.

---
## 🚨 El Problema: "Input/output error (os error 5)"
El sistema comenzó a congelarse aleatoriamente. Al ejecutar comandos básicos como ls, el sistema respondía con:
``Input/output error 5)`` 

1.**NCQ (Native Command Queuing):** Incompatibilidad del kernel con el controlador del disco duro SSD SATA (Fanxiang S101Q), causando bloqueos en la cola de comandos.

2.**Fast Startup de Windows:** El inicio rápido de Windows bloqueaba el hardware, impidiendo que Fedora tomara control del bus SATA.

3.**Corrupción de USB de rescate:** El primer intento de reparación falló debido a una memoria USB dañada físicamente (Error de E/S en diskpart).

---

## 🛠️ Fase 1: Estabilización del Hardware
1. **Parche de estabilidad del Kernel**
Para evitar que el SSD SATA se bloquee, desactivamos el NCQ.
- **Archivo:** `/etc/default/grub`
- Línea a modificar: `GRUB_CMDLINE_LINUX`
- Parámetro añadido: `libata.force=noncq`
  
```cmd
# Ejemplo de la línea final:

GRUB_CMDLINE_LINUX="rhgb quiet libata.force=noncq"

# Actualizar la configuración del GRUB (UEFI):

sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
2. **Reseteo de Controladores (Teclado/Energía)**
Si el hardware (teclado/touchpad) deja de responder tras múltiples Kernel Panics:
- Apagar la laptop, desconectar cargador y mantener presionado el botón de **Encendido por 60 segundos.** Esto drena la energía residual y resetea el controlador embebido (EC).
---
## 🛡️ Fase 2: Blindaje del Sistema (Btrfs Snapshots)
Para no depender de una USB de rescate en el futuro, instalamos un sistema de "puntos de restauración" accesibles desde el menú de arranque.

## 1. Instalación de Herramientas
```bash
sudo dnf install snapper btrfs-assistant
```
2. Configuración de Snapper
Configuramos el sistema para que realice capturas de la partición raíz (/):

```Bash
# Crear configuración para root
sudo snapper -c root create-config /

# Dar permisos al usuario actual
sudo chmod a+rx /.snapshots
sudo chown :$(whoami) /.snapshots
```

3.**Integracion con el menu GRUB** 
para poder arrancar un snapshot si el sistema no inicia, instalamos `grub-btrfs` desde COPR (en Fedora 43+):

```bash
# Habilitar repositorio de la comunidad
sudo dnf copr enable zeno/grub-btrfs
sudo dnf install grub-btrfs

# Solución de compatibilidad de rutas para Fedora:
sudo ln -s /boot/grub2 /boot/grub
sudo ln -s /usr/bin/grub2-script-check /usr/bin/grub-script-check

# Activar el demonio de monitoreo
sudo systemctl enable --now grub-btrfsd

# Actualizar el menú de arranque
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```
## 🔄 Fase 3: Procedimiento de Recuperación
**Opción A: Rollback mediante Consola (Si el sistema arranca)**
```bash
# Listar puntos de restauración
sudo snapper list

# Volver al snapshot ID X
sudo snapper rollback <ID>
reboot
```
**Opción B: Recuperación ante Desastres (Si el sistema NO arranca)**
1. Reiniciar la PC.
2. En el menú de GRUB, seleccionar **"Fedora Snapshots".**
3. Elegir la versión funcional por fecha.
4. Una vez en el escritorio, abrir **Btrfs Assistant** -> pestaña **Snapper** -> **Browse/Restore** y restaurar permanentemente.
---
## 💡 Lecciones Aprendidas
- **Hardware:** Verifica siempre el estado SMART del disco (CrystalDiskInfo en Windows o `smartctl` en Linux).
- **Herramientas:** Una USB de mala calidad puede reportar errores de software falsos. Usa Fedora Media Writer para verificar la integridad de la imagen.
- **Configuración:** En sistemas Dual Boot, el **Fast Startup de Windows debe estar desactivado** para evitar colisiones de hardware.
---
## 📂 Comandos útiles de diagnóstico
- `dmesg | grep -i error:` Ver errores del kernel en tiempo real.

- `lsblk -f:` Identificar particiones y UUIDs.

- `sudo btrfs check /dev/sdX:` Verificar integridad de archivos Btrfs (sin reparar).
