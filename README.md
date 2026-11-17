# FutbolWebApp

Sprints 1 y 2 de DataFlex en las prácticas de Data Control (DAW, CESUR).

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
- `feature/navegacion-*`: navegaciones del Sprint 2.

## Sprint 2: Navegación

Las vistas siguen la filosofía drill down (Dashboard > Select > Zoom). Cada navegación se clasifica
según la relación entre las tablas que alimentan la vista de origen y la de destino.

| Tipo | Dónde | Cómo |
|---|---|---|
| Undefined (ntNavigateForwardCustom) | Tarjetas del Dashboard hacia cada Select | `WebRegisterPath ntNavigateForwardCustom oSelect<X>` en cada tarjeta. No hay intercambio de información. |
| Undefined (ntNavigateForward) | Zoom jugador > Select posición | Botón "Ver posiciones". jugador y posicion solo se relacionan a través de `jugadorposicion`, sus DDs no tienen relación directa, así que no se aplica ninguna restricción. |
| FromMain | Select > Zoom de la misma tabla | Al pulsar la fila (`OnRowClick` de `oList`, cuando se viene del Dashboard) o el botón de detalle (`oDetailButton`). |
| FromChild | Zoom equipo > Select liga; Zoom jugador y Zoom entrenador > Select equipo | Formulario del padre con `pbPromptButton`. Al elegir una fila, el Select hace `NavigateClose` y el Zoom recibe el padre nuevo. |
| FromParent | Zoom liga > equipos de la liga | 1) Botón "Ver equipos de esta liga" que navega a Select equipo, restringido a esa liga. 2) Lista "Equipos de esta liga" dentro del propio Zoom, con un DD hijo (`oequipo_DD`) con `Constrain_File` a liga. |

### Participantes y eventos

- **InvokingView**: vista desde la que salgo. **InvokedView**: vista a la que llego.
- **InvokingObject**: objeto que lanza la navegación. **InvokedObject**: objeto de la vista destino que permite volver (`NavigateClose`).
- `OnGetNavigateForwardData` se declara en el InvokingObject, `OnNavigateForward` en la InvokedView,
  `OnGetNavigateBackData` en el InvokedObject y `OnNavigateBack` en el InvokingObject.

### FromMain: ¿qué pasa con el InvokingObject?

En Select > Zoom la vista de origen y la de destino son las mismas en los dos caminos, pero el
InvokingObject cambia:

- Si pulso la fila, el InvokingObject es la lista `oList` (lanza la navegación desde `OnRowClick`).
- Si pulso el botón de detalle, el InvokingObject es `oDetailButton` (desde su `OnClick`).
- Si pulso "Nuevo", es `oNewButton`.

Por eso `OnGetNavigateForwardData` está declarado en cada uno de esos objetos: en la lista y en el
botón de detalle marca `bReadOnly` (el Zoom se abre en modo consulta) y en "Nuevo" marca `bNewRecord`.
Si solo estuviera declarado en la vista no se ejecutaría. A la vuelta, la InvokedView del Zoom es
también el InvokedObject (`OnViewSaved` hace `NavigateClose Self`), y `OnNavigateBack` llegaría al
objeto que inició la navegación (la lista o el botón), no a la vista.

### Nota sobre ZoomLiga

`ZoomLiga.wo` no incluye `SelectEquipo.wo` con `Use` porque se formaría un ciclo
(SelectEquipo > ZoomEquipo > SelectLiga > ZoomLiga). Se declara el objeto con `Register_Object`
y el botón navega con `Send NavigateForward of oSelectEquipo Self`, que se resuelve en ejecución.
