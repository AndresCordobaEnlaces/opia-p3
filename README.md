# Aplicación Streamlit para el precio de casas en California

Esta aplicación estima el precio medio de las viviendas de una zona de California. El usuario solo introduce el ingreso medio y el número medio de habitaciones; el resto de variables del modelo se rellenan con valores por defecto (la mediana aproximada del dataset). Está formada por dos servicios: la API FastAPI (`app.py`), que carga el modelo y devuelve la predicción, y la interfaz Streamlit (`streamlit_app.py`), que recoge los datos, comprueba que están las 8 variables y llama a la API. Se conectan por HTTP: Streamlit envía un POST con un JSON a `/predict` y la API contesta con el valor predicho, que la interfaz convierte a dólares antes de mostrarlo.

Autores: Andrés Córdoba, Mykola Peredrii

## Servicios desplegados en Render

| Servicio | Qué es | URL |
|----------|--------|-----|
| API | FastAPI con el modelo (`app.py`) | <https://cordoba-p3-api.onrender.com> |
| Interfaz | Streamlit (`streamlit_app.py`), que llama a la API | <https://cordoba-p3-app.onrender.com> |

## Cómo ejecutarla en local

Hace falta tener `modelo_california.pkl` en la misma carpeta que `app.py`.

```bash
pip install -r requirements.txt
```

En una terminal se arranca la API:

```bash
uvicorn app:app --reload
```

En otra terminal se arranca la interfaz:

```bash
streamlit run streamlit_app.py
```

La API queda en http://localhost:8000 (con la documentación en `/docs`) y la interfaz en http://localhost:8501. En la barra lateral de la interfaz, la URL de la API es `http://localhost:8000`.

## Despliegue en Render

Se crean dos Web Services, uno para cada fichero.

**API**
- Build Command: `pip install -r requirements.txt`
- Start Command: `uvicorn app:app --host 0.0.0.0 --port $PORT`

**Interfaz**
- Build Command: `pip install -r requirements.txt`
- Start Command: `streamlit run streamlit_app.py --server.port $PORT --server.address 0.0.0.0 --server.headless true`

La URL de la API se cambia en `streamlit_app.py`, en la variable `API_URL` (hay que comentar la línea de localhost y descomentar la de Render con el nombre de nuestro servicio). También se puede cambiar en la barra lateral de la interfaz sin tocar el código.

## Documentación de la API

URL base: <https://cordoba-p3-api.onrender.com>

### `POST /predict`

Recibe un JSON con las 8 variables de una zona censal y devuelve el precio medio estimado de sus viviendas, en cientos de miles de dólares.

| Campo | Descripción |
|-------|-------------|
| `MedInc` | Ingreso medio de la zona (decenas de miles de $) |
| `HouseAge` | Antigüedad media de las viviendas (años) |
| `AveRooms` | Número medio de habitaciones por vivienda |
| `AveBedrms` | Número medio de dormitorios por vivienda |
| `Population` | Población de la zona |
| `AveOccup` | Número medio de ocupantes por vivienda |
| `Latitude` | Latitud |
| `Longitude` | Longitud |

**Ejemplo de petición**

```json
{
  "MedInc": 3.5,
  "HouseAge": 29.0,
  "AveRooms": 5.2,
  "AveBedrms": 1.05,
  "Population": 1166.0,
  "AveOccup": 2.8,
  "Latitude": 34.26,
  "Longitude": -118.49
}
```

**Ejemplo de respuesta correcta (200)**

```json
{
  "MedHouseVal": 1.5126018234536092
}
```

Este valor equivale a unos $151,260 (se multiplica por 100.000).

**Errores**

| Código | Cuándo ocurre | Qué muestra la aplicación |
|--------|---------------|---------------------------|
| 422 | Falta un campo o tiene un tipo incorrecto | Un mensaje con el código y el detalle que devuelve la API: "La API ha devuelto un error (código 422): ..." |
| 500 | Error interno de la API (el cuerpo es texto, no JSON) | El mismo mensaje con el código 500 y el texto de la respuesta |
| Sin respuesta | La API no está disponible o tarda demasiado | "No se ha podido contactar con la API..." con consejos para solucionarlo |

Además, la aplicación comprueba antes de enviar que están las 8 variables. Si falta alguna, no envía la petición y muestra cuál falta.
