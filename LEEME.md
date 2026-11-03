# UDB Academic · Versión gratuita

El proyecto principal está en `C:\Users\ander\OneDrive\Desktop\ProyectoCatedraAPR`. Abrí esa carpeta completa en Android Studio y sincronizá Gradle.

## Qué utiliza la aplicación

- Firebase Authentication para las cuentas institucionales.
- Cloud Firestore en el plan **Spark gratuito** para perfiles, materias, horarios, actividades, texto de los apuntes y noticias.
- Archivos internos del teléfono para los PDF e imágenes adjuntos.
- Alarmas y notificaciones locales de Android para los recordatorios.

No se vinculó facturación ni se activó Blaze. La aplicación no integra Firebase Storage, Cloud Messaging ni Cloud Functions. La configuración de despliegue incluye únicamente Firestore. Spark tiene límites de uso: al alcanzarlos, las operaciones correspondientes pueden detenerse hasta que se renueve la cuota; no se contrata capacidad de pago automáticamente.

## Versiones conservadas

| Componente | Configuración |
|---|---|
| Complemento Android | 9.1.1 |
| Gradle | 9.3.1 |
| Kotlin | Integrado en el complemento Android |
| Android mínimo / destino / compilación | 24 / 36 / 37.1 |
| Java del código / ejecución de Gradle | 11 / JDK 21 de Android Studio |
| Firebase BoM | 34.4.0, compatible con el Kotlin del proyecto |

Se conservaron las bibliotecas Android originales. El complemento muestra una advertencia porque fue probado hasta SDK 37.0 y el proyecto original usa 37.1; esa selección se conservó.

## Primer uso

1. Creá una cuenta con un correo real `@alumno.udb.edu.sv` o `@udb.edu.sv` y una contraseña de al menos ocho caracteres.
2. Abrí el correo de verificación y activá la cuenta antes de ingresar. La cuenta de Google que administra Firebase es independiente de las cuentas estudiantiles de la app.
3. Completá tu perfil y agregá las materias en **Horario**.
4. Registrá clases de lunes a viernes, indicando aula y horas. La aplicación valida que no se crucen.
5. En **Actividades**, registrá tareas, laboratorios, parciales y proyectos, con fecha de entrega, porcentaje y nota opcional. Podés editar, eliminar y cambiar su estado.
6. En **Apuntes**, guardá notas por materia y adjuntá archivos del teléfono.
7. Permití las notificaciones para recibir recordatorios. En **Perfil → Probar recordatorio ahora** podés comprobarlos al instante.

**Ver demostración con datos de ejemplo** permite recorrer las pantallas sin una cuenta. Los datos son ficticios, están separados de Firebase y se descartan al salir de la demostración. También permite probar adjuntos y notificaciones locales.

## Adjuntos guardados en el teléfono

Los archivos se copian al almacenamiento privado de la aplicación y quedan separados por cuenta y por apunte. Sus nombres y rutas locales no se envían a Firestore.

Cada apunte admite hasta cinco archivos PDF, JPEG, PNG o WebP, de 10 MB como máximo por archivo. Las copias se realizan en segundo plano para mantener la interfaz disponible. El límite real se comprueba durante la copia, aunque el proveedor no indique el tamaño del archivo.

**Guardar copia de…** abre el selector de Android para exportar el archivo, por ejemplo a Descargas o a una carpeta que elijas. No se solicita acceso general a todos los archivos del dispositivo.

**Los adjuntos no se sincronizan con otros teléfonos.** El texto de los apuntes sí se sincroniza con Firebase. Si desinstalás la app o borrás sus datos, Android elimina los archivos privados; exportá una copia de los documentos que quieras conservar. Cerrar sesión conserva los archivos de la cuenta para el próximo ingreso en el mismo teléfono.

## Notificaciones locales y calendario

El teléfono programa recordatorios aproximadamente veinticuatro horas antes de las entregas pendientes que se hayan sincronizado en ese dispositivo. Se actualizan al modificar actividades y se restauran después de reiniciar el teléfono. Cerrar sesión cancela los recordatorios de la cuenta.

Android puede retrasar las alarmas por ahorro de batería. No se solicita permiso para alarmas exactas. Si una actividad se registra cuando ya faltan menos de veinticuatro horas, no puede generar un aviso en una hora que ya pasó. El botón de prueba muestra una notificación de ejemplo inmediata, claramente identificada como simulación local.

**Agregar al calendario**, dentro de una clase, abre el calendario del teléfono con una repetición semanal. Revisá la repetición y su fecha final antes de confirmar. Es una exportación, no una sincronización automática en ambos sentidos.

## Cálculo de notas

La escala es de 0 a 10. Cada evaluación aporta `nota × porcentaje ÷ 100`. Las notas pendientes aportan cero; su porcentaje no se redistribuye.

Ejemplo: una nota de 10 con peso del 30 % y una nota de 6 con peso del 70 % producen 7.20. La app impide superar el 100 % por materia al guardar y avisa si faltan porcentajes.

El promedio del ciclo es la media simple de las materias con evaluaciones configuradas. No se aplican unidades valorativas porque el documento no las define. Los cruces de horario y la suma de porcentajes se validan en la aplicación; las reglas de Firebase verifican propiedad, estructura y límites individuales.

## Organización del código

| Archivo o carpeta | Contenido |
|---|---|
| `app/src/main/java/com/example/udb_academic/ActividadPrincipal.kt` | Pantallas, formularios y navegación |
| `modelo/Modelos.kt` | Modelos de datos del sistema |
| `dominio/ReglasAcademicas.kt` | Promedios y validaciones académicas |
| `datos/Autenticacion.kt` | Registro, ingreso, verificación y contraseñas |
| `datos/RepositorioAcademico.kt` | Consultas y sincronización de datos académicos |
| `datos/AdjuntosLocales.kt` | Guardado, exportación y eliminación de archivos del teléfono |
| `notificaciones/Recordatorios.kt` | Alarmas y notificaciones locales |
| `firebase/firestore.rules` | Reglas privadas de la base de datos |
| `firebase/pruebas/reglas.test.mjs` | Pruebas de seguridad de Firebase |

Los nombres propios del sistema están en español. Los nombres obligatorios de Android y sus bibliotecas se conservan.

## Firebase configurado

Proyecto: **UDB Academic**. Identificador: `udb-academic`.

Consola: https://console.firebase.google.com/project/udb-academic/overview

Paquete Android: `com.example.udb_academic`. Configuración: `app/google-services.json`.

Firestore está en la ubicación `nam5`:

```text
estudiantes/{identificadorDeFirebaseAuth}
  nombre, correo, carnet, carrera, ciclo
  materias/{id}
  horarios/{id}
  actividades/{id}
  apuntes/{id}
noticias/{id}
```

Las colecciones privadas se crean cuando cada estudiante guarda sus primeros registros. No se crearon cuentas reales de estudiantes para las pruebas. El aviso de bienvenida publicado en `noticias/bienvenida` pertenece a la aplicación y no contiene fechas institucionales inventadas.

Las reglas exigen correo institucional verificado y permiten acceder solamente a los datos del propio usuario. Las contraseñas las gestiona Firebase Authentication; no se almacenan en Firestore. La app solicita en español los correos de verificación y recuperación.

Los estudiantes leen noticias; su publicación corresponde a una persona con acceso administrativo a la consola. Para agregar una noticia, creá un documento en `noticias` con estos campos:

| Campo | Tipo | Contenido |
|---|---|---|
| `titulo` | Texto | Título de la publicación |
| `descripcion` | Texto | Contenido |
| `fecha` | Número entero | Fecha y hora en milisegundos desde 1970 |
| `enlace` | Texto | URL HTTPS o cadena vacía |
| `categoria` | Texto | Noticia, Evento, Aviso u otra categoría |

La sincronización sin conexión corresponde a Firestore. El acceso inicial, la verificación de correo y la recuperación de contraseña requieren Internet. Los archivos locales y las alarmas programadas no necesitan conexión.

## Comprobaciones

Resultado final: 5 pruebas de lógica, 12 de seguridad de Firestore y 4 de Android aprobadas. Compilación y revisión de Android completadas sin errores.

- Pruebas de lógica: dominio institucional, ponderaciones, promedio global, notas inválidas y cruces de horario.
- Pruebas de Firestore con emulador: aislamiento entre cuentas, verificación de correo, validación de documentos, bloqueo de rutas de archivos locales y protección de noticias.
- Pruebas Android: archivos locales, exportación, separación entre cuentas, límites de tamaño y cantidad, notificación de ejemplo y recorrido de pantallas.

El modo oscuro automático queda fuera de esta fase, tal como lo indica el documento. Esta versión sustituye los adjuntos en Firebase Storage y los push del servidor por funciones locales, según tu decisión de mantener el proyecto sin servicios de pago.

## Referencias

- Almacenamiento privado de Android: https://developer.android.com/training/data-storage/app-specific
- Planes de Firebase: https://firebase.google.com/docs/projects/billing/firebase-pricing-plans
