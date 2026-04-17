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
