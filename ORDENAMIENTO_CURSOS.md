# Ordenamiento de Cursos por Nombre (Alfabético)

## Cambios Realizados

Se ha implementado una nueva funcionalidad para ordenar los cursos alfabéticamente por su nombre de visualización (display name). Esto es útil para ordenar cursos por nivel, como: **A1+, A2, A2+, B1, B1+**.

### Archivos Modificados:

1. **`common/djangoapps/student/views/management.py`**
   - Se agregó la función `sort_by_display_name()` que ordena los cursos alfabéticamente usando ordenamiento natural (maneja correctamente números y letras)
   - Se modificó la función `index()` para incluir la nueva opción de ordenamiento

2. **`lms/envs/common.py`**
   - Se agregó la configuración `ENABLE_COURSE_SORTING_BY_DISPLAY_NAME`

## Cómo Habilitar el Ordenamiento Alfabético

### Opción 1: Usando Configuración de Django (Recomendado para Producción)

Agrega lo siguiente en tu archivo de configuración de producción o en las variables de entorno:

```python
FEATURES['ENABLE_COURSE_SORTING_BY_DISPLAY_NAME'] = True
```

### Opción 2: Modificar directamente `lms/envs/common.py` (Para desarrollo/testing)

Cambia el valor en la línea 516 de `False` a `True`:

```python
'ENABLE_COURSE_SORTING_BY_DISPLAY_NAME': True,  # Cambiar de False a True
```

### Opción 3: Usando Configuración del Sitio (Site Configuration)

Si usas Site Configuration de Open edX, puedes agregar esto en la configuración JSON de tu sitio:

```json
{
  "ENABLE_COURSE_SORTING_BY_DISPLAY_NAME": true
}
```

## Prioridad de Ordenamiento

El sistema verifica las opciones de ordenamiento en el siguiente orden:

1. **Alfabético** (`ENABLE_COURSE_SORTING_BY_DISPLAY_NAME`) - ⭐ NUEVO
2. Por fecha de inicio (`ENABLE_COURSE_SORTING_BY_START_DATE`)
3. Por fecha de anuncio (por defecto)

Si `ENABLE_COURSE_SORTING_BY_DISPLAY_NAME` está habilitado, tendrá prioridad sobre los otros métodos.

## Cómo Funciona el Ordenamiento

La función `sort_by_display_name()` usa un algoritmo de "ordenamiento natural" que:

- Separa las letras de los números en el nombre del curso
- Ordena las letras alfabéticamente (sin importar mayúsculas/minúsculas)
- Ordena los números numéricamente (no como texto)

**Ejemplo:**
- A1+ viene antes que A2
- A2 viene antes que A2+
- A2+ viene antes que B1
- Resultado: A1+, A2, A2+, B1, B1+

## Reiniciar el Servidor

Después de hacer los cambios, reinicia el servidor LMS:

```bash
# Para desarrollo con paver
paver lms restart

# Para Docker
docker-compose restart lms

# Para producción con systemd
sudo systemctl restart edxapp_lms
```

## Limpiar Caché

Si los cambios no se reflejan inmediatamente, puede ser necesario limpiar la caché:

```python
# En el shell de Django
./manage.py lms shell
from django.core.cache import cache
cache.clear()
```

## Verificación

Para verificar que el ordenamiento está funcionando:

1. Ve a la página principal (homepage) en tu navegador
2. Los cursos deberían aparecer ordenados alfabéticamente: A1+, A2, A2+, B1, B1+
3. Si no ves el cambio, verifica:
   - Que el servidor se haya reiniciado
   - Que la configuración esté activada
   - Que la caché esté limpiada

## Desactivar el Ordenamiento Alfabético

Para volver al ordenamiento anterior, simplemente:

1. Cambia `ENABLE_COURSE_SORTING_BY_DISPLAY_NAME` a `False`
2. Reinicia el servidor

El sistema volverá a usar el ordenamiento por fecha de inicio o fecha de anuncio según la otra configuración.
