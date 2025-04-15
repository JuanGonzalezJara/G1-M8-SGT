# PASO A PASO

## Resumen

Este reporte señala los pasos realizados para l ejecucion del desafio y los problemas presentados

## Pasos

1. **Gestión del Repositorio Git**  
     
   #### Se inicializa un repositorio local y se realiza un commit con un archivo `README.md`
   ```
    git init
    git branch -m main
    git add .
    git commit -m "1st commit - base project"
   ```
   #### Se crea un proyecto en Github y se conecta remotamente con la instancia local
   ```
    git remote add origin https://github.com/JuanGonzalezJara/G1-M8-SGT.git
    git commit -m "1st commit - base project"
   ```
   #### Se agrega archivo .gitignore para evitar que la carpeta node_modules se suba al repo.

   

2. **Configuración de la API con Docker**  
     
   #### Se crear un archivo `Dockerfile` que permita construir una imagen para ejecutar la API.
    ```
    FROM node:16            /Usa la imagen oficial de Node.js v.16 como base.
    WORKDIR /usr/src/app    /Establece el directorio de trabajo dentro del contenedor.
    COPY package.json ./    /Copia el archivo package.json al directorio de trabajo.
    RUN npm install         /Instala las dependencias definidas en package.json.
    COPY . .                /Copia el resto de los archivos del proyecto al contenedor.
    EXPOSE 3000             /Expone el puerto 3000 para que pueda accederse a la app desde fuera del contenedor.
    CMD ["npm", "start"]    /Ejecuta el comando para iniciar la aplicación cuando el contenedor arranca.
    ```
   

3. **Pipeline de Jenkins**  
     
   #### Se configura Jenkins para clonar el repositorio desde GitHub.
   ```
    stage('Checkout') {
            steps {
                echo 'Checking out code..'
                git branch: 'main', url: 'https://github.com/JuanGonzalezJara/G1-M8-SGT.git'
            }
        }
   ```
   #### Se define un Pipeline que realize:
    ##### - Instale dependencias
    ##### - Ejecute las pruebas automatizadas
    ##### - Construya una imagen Docker a partir del `Dockerfile`
    ```
    stage('Build') {
            steps {
                echo 'Building the app..'
                bat 'npm install' // Instala dependencias
          }
        }

        stage('Test') {
            steps {
            echo 'Running tests..'
            bat 'npm test' // Inicia los test con jest
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying the app..'
                script {
                    docker.build('G1-M8-SGT:latest') // Crea la imagen de Docker
                }
            }
        }
   ```

## Problemas Encontrados

- No se exporta el modulo app en app.js, se resuelve agregando sentencia al final del js.
```
module.exports = app;
```
- "Checkout requires a valid repository URL", se resuelve agregando correctamente repositorio en jenkinsfile
```
git branch: 'main', url: 'https://github.com/JuanGonzalezJara/G1-M8-SGT.git'
```
- "jest" no se reconoce como un comando interno o externo, se resuelve agregando las dependencias en package.json
```
npm i jest supertest -D
```
- "Jest did not exit one second after the test run has completed." Esto es debido a que queda el servidor corriendo al finalizar las pruebas, lo que no permite cerrar el stage, se resuleve cambiando la manera en como iniciamos el servidor, quitando el app.listen de app.js y dejando la llamada al servidor de manera independiente
```
* Crear archivo server.js
const app = require('./app');
const PORT = 3000;
app.listen(PORT, () => console.log(`API is running on port ${PORT}`));

* Modificar package.json
"start": "node server.js",

*Eliminar app.listen de app.js
```

## Resultados

- Resultados de la ejecucion del Pipeline
![foto1](agregar url img)
- Resultados de la creacion y ejecucion de la imagen en Docker Desktop
![foto2](agregar url img)
- Resultados de los endpoints funcionales
![foto3](agregar url img)

