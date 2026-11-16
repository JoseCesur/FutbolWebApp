# FutbolWebApp

Sprint 1 de DataFlex en las prácticas de Data Control (DAW, CESUR).

Aplicación web (DataFlex 25, Web Mobile Project) sobre la base de datos MySQL `futbol`:
vistas Select y Zoom de entrenador, jugador, posición, equipo y liga, y sus tarjetas en el Dashboard.

## Puesta en marcha

1. Crear la base de datos con el script `futbol.sql` en MySQL 8.
2. Crear el usuario de servicio `futbol_app` con permisos SELECT, INSERT, UPDATE y DELETE sobre `futbol`.
3. Crear un DSN de sistema ODBC de 64 bits llamado `futbol` (MySQL ODBC Unicode Driver).
4. Copiar `Data/DFConnId.ini.ejemplo` a `Data/DFConnId.ini` y guardar la contraseña desde
   DataFlex Studio (Database > SQL Connection Manager).
5. Abrir `FutbolWebApp.sws` en DataFlex Studio 25 y ejecutar (F5).

## Ramas (gitflow)

- `main`: versión entregada.
- `develop`: rama de trabajo donde se integran las features.
- `feature/<tabla>`: vistas Select y Zoom de cada tabla; `feature/dashboard`: tarjetas del Dashboard.
