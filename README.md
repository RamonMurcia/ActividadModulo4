# ActividadModulo4
actividad para la integración continua (CI, continuous integration) y para el despliegue continuo (CD, continuous deployment).

Componentes:  
Ramón Miguel Murcia Campillo
Javier Santamaría Beaucourt
Pedro Kabine

Desarrollado por Ramón:

    1. Crear en github desktop el repo ActividadModulo

    2. Clonar el repo ttps://github.com/joseagcv/pontia-mlops-tutorial, en mi equipo le puse el alias 4.1-Devops.

    3. Copiar los archivos del repo copiado al nuevo repo creado.

    4. Agregar las carpetas /data y /models al .gitignore

    5. Copiamos el requirements de la Carpeta Ci en nuestro nuevo proyecto.

    6. Modificamos el main.py y modificamos el valor de MLFLOW_URI , quedando:
        MLFLOW_URI = os. getenv("MLFLOW_URI", "http://Localhost:5001/")
        - Nota; recuerdo que en la clase tambien se modifico el EXPERIMENT_NAME, de momento como no dice nada en las instrucciones no lo modifico, si es necesario lo cambio mas adelante y lo dejo anotado.
        
    7. Arranco un servidor local de mlflow:
        docker run -d --name mlflow -p 5001:5000 ghcr.io/mlflow/mlflow:v3.7.0 mlflow server --host 0.0.0.0 --port 5000 
        - Nota; en las instrucciones usa como puertos -p 5000:5000, yo he puesto el local como 5001, para aprender a diferenciarlos y no confundirme 
            Me dio error al lanzarlo por no tener abierto docker desktop, lo abro
                ramonmurciacampillo@RamonMurcia ActividadModulo4 % docker run -d --name mlflow -p 5001:5000 ghcr.io/mlflow/mlflow:v3.7.0 mlflow server --host 0.0.0.0 --port 5000
                    Cannot connect to the Docker daemon at unix:///Users/ramonmurciacampillo/.docker/run/docker.sock. Is the docker daemon running?
                    zsh: command not found: server
                ramonmurciacampillo@RamonMurcia ActividadModulo4 % docker run -d --name mlflow -p 5001:5000 ghcr.io/mlflow/mlflow:v3.7.0 mlflow 
                server --host 0.0.0.0 --port 5000
                    Unable to find image 'ghcr.io/mlflow/mlflow:v3.7.0' locally
                    v3.7.0: Pulling from mlflow/mlflow
                    3669af84f2ec: Pull complete 
                    5e584f8f28c3: Pull complete 
                    ccaf924377f9: Pull complete 
                    954774345a61: Pull complete 
                    7360b233c860: Pull complete 
                    0ed2d686b548: Download complete 
                    Digest: sha256:4590656b2856508ba1eb6885cb9f4d2e747b164ed0d999fa542e4573843c4c8a
                    Status: Downloaded newer image for ghcr.io/mlflow/mlflow:v3.7.0
                    b4e9858adc548c9b3da59980d2847bea54043ebad952becbfe89d733907e610e
                    zsh: command not found: server

    8. Probar el codigo en local, para ello ejecuto los siguiente comandos:
            ramonmurciacampillo@RamonMurcia ActividadModulo4 %  python3.12 -m venv venv
            ramonmurciacampillo@RamonMurcia ActividadModulo4 %  source venv/bin/activate
            ((venv) ) ramonmurciacampillo@RamonMurcia ActividadModulo4 % pip install -r requirements.txt
        - modificado tambien EXPERIMENT_NAME quedando:
            EXPERIMENT_NAME = os.getenv("EXPERIMENT_NAME", "grupo07-adult-income")
        - pruebo si se ve la app en http://127.0.0.1:5001
        - La app esta levantada

    9. Anado los secrets y varaiables de entorno en staging en github.com
        - Secrets:
            - AZURE_TENANT_ID
            - AZURE_CLIENT_ID
            - AZURE_CLIENT_SECRET
        - variables de entorno:
            - MLFLOW_URL: azureml://spaincentral.api.azureml.ms/mlflow/v1.0/subscriptions/b17a3b65-c9df-48c7-9b32-8cd81f620076/resourceGroups/rg-albertog-devops/providers/Microsoft.MachineLearningServices/workspaces/devops-mlw
            - EXPERIMENT_NAME: grupo07-adult-income

    10. Creo una carpeta en la raiz del repositorio con el nombre “.github/workflows”.

    11. Copio el build.yml de la carpeta CI de los recursos
        - Error en action (Set up Python).

    12. cambio la version de python a 3.12
        - Error en action (Train ans save model).

    13. Reparo el pip install, step Install dependencies
        - Error en action (Train ans save model).

    14. Descargo los datos con un task dentro de la pipeline, creo el step Download and extract dataset
        - name: Download and extract dataset
            shell: bash
            run: |
            set -euo pipefail

            mkdir -p ./data/raw

            curl -fL \
                "https://archive.ics.uci.edu/static/public/20/census+income.zip" \
                -o /tmp/census-income.zip

            unzip -o /tmp/census-income.zip -d ./data/raw

            rm /tmp/census-income.zip

            echo "Dataset extracted to ./data/raw"
            find ./data/raw -maxdepth 2 -type f  
        - Error en action (Train ans save model).

    15. Añado al build el environmet
            environment: Staging
    -  Error en action (Train ans save model). Estaba todo el ratro dando error de ruta de main.py
            Run python main.py
            python: can't open file '/home/runner/work/ActividadModulo4/ActividadModulo4/main.py': [Errno 2] No such file or directory

    16. Corrijo la ruta del main.py en el step Train ans Save model, añadiendo delante src/
        - name: Train and save model
            run: |
            python src/main.py
        - Error en action (Train ans save model), pero esta vez ha estado mucho mas timepo y el log es mucho mayor, dando este error:
            mlflow.exceptions.MlflowException: API request to http://Localhost:5001/api/2.0/mlflow/experiments/get-by-name failed with exception HTTPConnectionPool(host='localhost', port=5001): Max retries exceeded with url: /api/2.0/mlflow/experiments/get-by-name?experiment_name=grupo07-adult-income (Caused by NewConnectionError("HTTPConnection(host='localhost', port=5001): Failed to establish a new connection: [Errno 111] Connection refused"))

    17. Añado las variables de entorno y secretos en el step Train ans Save model
            env:
            AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
            AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
            AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
            EXPERIMENT_NAME: ${{ vars.EXPERIMENT_NAME }}
            MLFLOW_URI: ${{ vars.MLFLOW_URL }}
        - Termina el build sin errores

    18.  Copio el integration.yml de la carpeta CI de los recursos.

    19. Aparece en action de github, no se ejecuta porque solo se ejecuta cuando es un pull request, como indica al principio del yml
        on:
        pull_request:

    20. Cambio la versión de Python a 3.12 en integration.yml y creo la rama feature/integration para lanzar el workflow mediante un Pull Request.

    21. En github comparo el pull request y autorizo la pr, el action de integration se lanza
        - Error en action (Run tests and save exit code),
            Run mkdir -p test-results
            /home/runner/work/_temp/72eb35d7-6acc-4a48-b831-8bcee4177050.sh: line 2: pytest: command not found
            Error: Process completed with exit code 127.

    22. Reparo el pip install, step Install dependencies 
        - Error en action (Run tests and save exit code)
            /opt/hostedtoolcache/Python/3.12.15/x64/lib/python3.12/site-packages/coverage/inorout.py:560: CoverageWarning: Module model was never imported. (module-not-imported); see https://coverage.readthedocs.io/en/7.17.1/messages.html#warning-module-not-imported
            self.warn(f"Module {pkg} was never imported.", slug="module-not-imported")
            /opt/hostedtoolcache/Python/3.12.15/x64/lib/python3.12/site-packages/coverage/control.py:967: CoverageWarning: No data was collected. (no-data-collected); see https://coverage.readthedocs.io/en/7.17.1/messages.html#warning-no-data-collected
            self._warn("No data was collected.", slug="no-data-collected")
            /opt/hostedtoolcache/Python/3.12.15/x64/lib/python3.12/site-packages/pytest_cov/plugin.py:311: CovReportWarning: Failed to generate report: No data to report.
            warnings.warn(CovReportWarning(message))

    23. Añado los steps como en el la pipeline de build, el step que descarga el modelo y el step que descarga y extrae el dataset
        - Error en action (Download Model), me faltaba añadir el environmet

    24. Añado el environmet staging
        - Error en action (Download Model), daba 403 porque scripts/download_model.py tenía una URL antigua hardcodeada y no leía las variables de entorno MLFLOW_URI ni EXPERIMENT_NAME.

    25. Modifico el scripts/download_model.py añado las variables de entorno MLFLOW_URI y EXPERIMENT_NAME y  uso os.getenv().

    26. Agrego un Ruleset sobre la rama main: 
        -  “Restrict deletions” 
        -  “Block force pushed” 
        -  “Require status checks to pass”. Sobre nuestra pipeline. 
        -  “Require branches to be up to date before merging” 

Desarrollado por Javier:

    1. En la pipeline de integración (`integration.yml`), desactivé la condición `if: always()` del paso que publica los resultados de las pruebas. Al quedar comentada, ese paso se ejecuta solo si los anteriores terminan correctamente.

    2. Añadí la carpeta `deployment` y la nueva pipeline `deploy.yml` para construir y publicar la imagen Docker y desplegar la aplicación.

    3. Limpié el Dockerfile: las librerías dejaron de estar definidas directamente en él y pasaron al archivo `deployment/requirements.txt`, que el contenedor instala durante la construcción.

    4. Ramón creó el repositorio en Docker Hub, definió el nombre de la imagen y configuró las variables y los secretos necesarios para que pudiera ejecutarse la pipeline de despliegue.

    5. Finalmente, creé la pull request para integrar los cambios.

## ¿En qué consiste el proyecto?

ActividadModulo4 reúne el código para entrenar y evaluar un modelo de aprendizaje automático que predice el nivel de ingresos a partir de datos demográficos, junto con las automatizaciones para probarlo y desplegar una API que permite usarlo.

### Pipelines

- **Build (`.github/workflows/build.yml`)**: se ejecuta al actualizar `main`. Instala las dependencias, descarga y prepara el conjunto de datos, entrena el modelo con `src/main.py` y registra el modelo y sus artefactos en MLflow. También publica el archivo del modelo como artefacto de la ejecución.
- **Integración (`.github/workflows/integration.yml`)**: se ejecuta al abrir o actualizar una pull request. Instala las dependencias, descarga el modelo y los datos que necesitan las pruebas, ejecuta los tests y publica un resumen de resultados y cobertura como comentario en la pull request.
- **Despliegue (`.github/workflows/deploy.yml`)**: se ejecuta al actualizar `main` o manualmente. Construye la imagen con los archivos de `deployment`, la publica en Docker Hub y actualiza la aplicación en Azure Container Apps.

### Estructura del proyecto

```text
ActividadModulo4/
├── .github/workflows/
│   ├── build.yml            # Entrenamiento del modelo
│   ├── integration.yml      # Pruebas en las pull requests
│   └── deploy.yml           # Construcción y despliegue de la API
├── deployment/
│   ├── app/main.py          # API FastAPI que carga el modelo y ofrece /predict
│   ├── Dockerfile           # Instrucciones para construir el contenedor
│   └── requirements.txt     # Dependencias específicas de la aplicación desplegada
├── scripts/
│   └── download_model.py    # Descarga desde MLflow la última versión del modelo
├── src/
│   ├── data_loader.py       # Lectura, limpieza y preparación de los datos
│   ├── evaluate.py          # Evaluación del modelo y registro de métricas
│   ├── main.py              # Coordina el entrenamiento y el registro en MLflow
│   └── model.py             # Entrenamiento del clasificador Random Forest
├── tests/                   # Pruebas del procesamiento, entrenamiento y evaluación
├── requirements.txt         # Dependencias para desarrollo, entrenamiento y pruebas
├── pytest.ini               # Configuración de pytest
└── .gitignore               # Archivos locales, datos y modelos que no se versionan
```

En pocas palabras, `src` contiene el flujo de entrenamiento: se leen y preparan los datos, se entrena un Random Forest, se evalúa y se registra el resultado en MLflow. `tests` comprueba las piezas principales de ese flujo. La carpeta `deployment` contiene la API y su contenedor; al arrancar, la API carga desde MLflow el modelo configurado y permite solicitar predicciones mediante `/predict`. Los datos y modelos generados se mantienen fuera del control de versiones según las reglas de `.gitignore`.

### Organización del trabajo

En el proyecto participamos Javier, Pedro y Ramón. Hemos seguido las clases del profesor y completado las tareas correspondientes; sin embargo, al repartir el trabajo, no todos hemos podido contribuir de la misma manera a los cambios compartidos del repositorio (no había suficiente para distribuir). Como posible forma de dejar constancia del trabajo y del conocimiento común, podríamos añadir al profesor como colaborador en cada uno de nuestros proyectos individuales, de modo que pueda comprobar que hemos seguido las tareas y trabajado los mismos contenidos.