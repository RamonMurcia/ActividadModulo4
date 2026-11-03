# ActividadModulo4
actividad para la integración continua (CI, continuous integration) y para el despliegue continuo (CD, continuous deployment).

Componentes:  
Ramón Miguel Murcia Campillo

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
    Error en action (Set up Python).
12. cambio la version de python a 3.12
    Error en action (Train ans save model).
13. Reparo el pip install, step Install dependencies
    Error en action (Train ans save model).
14. Descargo los datos con un task dentro de la pipeline, creo el step Download and extract dataset
    Error en action (Train ans save model).
15. Añado al build la linea; environment: Staging
    Error en action (Train ans save model). Estaba todo el ratro dando error de ruta de main.py
        Run python main.py
        python: can't open file '/home/runner/work/ActividadModulo4/ActividadModulo4/main.py': [Errno 2] No such file or directory
16. Corrijo la ruta del main.py en el step Train ans Save model, añadiendo delante src/
    Error en action (Train ans save model), pero esta vez ha estado mucho mas timepo y el log es mucho mayor, dando este error:
        mlflow.exceptions.MlflowException: API request to http://Localhost:5001/api/2.0/mlflow/experiments/get-by-name failed with exception HTTPConnectionPool(host='localhost', port=5001): Max retries exceeded with url: /api/2.0/mlflow/experiments/get-by-name?experiment_name=grupo07-adult-income (Caused by NewConnectionError("HTTPConnection(host='localhost', port=5001): Failed to establish a new connection: [Errno 111] Connection refused"))
17. Añado las variables de entorno y secretos en el step Train ans Save model
    

