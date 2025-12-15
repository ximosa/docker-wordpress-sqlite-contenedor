# WordPress Docker Images with SQLite Support
> Always up-to-date multi-arch WordPress Docker images with built-in SQLite support.<br>

[![License: GPL v2](https://img.shields.io/badge/License-GPL_v2-blue.svg)](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)
[![Multi-Architecture Support](https://img.shields.io/badge/arch-linux%2Famd64-green)](https://github.com/yunussandikci/docker-wordpress-sqlite/pkgs/container/wordpress-sqlite)
[![Multi-Architecture Support](https://img.shields.io/badge/arch-linux%2Farm%2Fv7-green)](https://github.com/yunussandikci/docker-wordpress-sqlite/pkgs/container/wordpress-sqlite)
[![Multi-Architecture Support](https://img.shields.io/badge/arch-linux%2Farm%2Fv8-green)](https://github.com/yunussandikci/docker-wordpress-sqlite/pkgs/container/wordpress-sqlite)

TL;DR 
```
docker run -d -p 8080:80 ghcr.io/yunussandikci/wordpress-sqlite:6.8.2-php8.4-apache
```

This repository provides always up-to-date WordPress Docker images with SQLite support, making it the best choice for quick development, testing, and small-scale deployments. [Check all available docker images!](https://github.com/yunussandikci/docker-wordpress-sqlite/pkgs/container/wordpress-sqlite/versions)

<img src="https://github.com/user-attachments/assets/79f287b9-ef4d-4b12-b9a6-fbb5152d5517"  width="600"/>

## ✨ Features 

- **SQLite Integration**: WordPress is pre-configured to use SQLite as the database, thanks to the [SQLite Database Integration Plugin](https://github.com/WordPress/sqlite-database-integration). No extra setup required!
- **Automated Builds**: Docker images are automatically built and pushed to the GitHub Container Registry (`ghcr.io`) whenever new WordPress tags are released.
- **Multi-Architecture Support**: Images are built for multiple architectures ensuring compatibility across a wide range of systems.
  - `linux/amd64`, `linux/arm/v7`, `linux/arm64/v8` 
- **Minimalistic**: Only WordPress and the SQLite plugin are included—nothing extra. Lightweight and efficient!

## 🚀 Usage
To run the container, use the following command:

```bash
docker run -d -p 8080:80 ghcr.io/yunussandikci/wordpress-sqlite:6.8.2-php8.4-apache
```

This will start a WordPress instance with SQLite as the database, accessible at `http://localhost:8080`.

## 🚀 Despliegue Headless en Google Cloud (SQLite + Docker)

Este proyecto despliega una instancia de WordPress optimizada con SQLite para ser usada como un CMS Headless ultraligero y persistente. Se utiliza Google Cloud Compute Engine (la capa gratuita) para asegurar que la base de datos no se borre.

### Requisitos Previos

- Una cuenta en Google Cloud Platform (GCP).
- Tener instalado el Docker Engine y Docker Compose en tu máquina local.
- Tu código fuente (este repositorio) ya debe estar en tu máquina.

### 1. Configuración de Google Cloud (Instancia Gratuita)

Para que tus datos sean persistentes y el coste sea 0€, la instancia debe crearse con esta configuración específica:

1. Ve a la Consola de Google Cloud.
2. Navega a **Compute Engine > VM Instances**.
3. Haz clic en **CREAR INSTANCIA**.

| Configuración Clave | Valor | Notas Importantes |
| :--- | :--- | :--- |
| Región | `us-central1` (o `us-west1`/`us-east1`) | ¡CRÍTICO! Debe ser una de estas para el plan "Always Free". |
| Tipo de Máquina | `e2-micro` | ¡CRÍTICO! Único tipo de máquina gratuita. |
| Disco de Arranque | Cambiar a **Standard persistent disk** | 30GB de disco son gratuitos. No uses el "Balanced". |
| Firewall | Marcar: **Permitir tráfico HTTP** y **Permitir tráfico HTTPS** | Para abrir los puertos 80 y 443 al mundo. |

Anota la **Dirección IP Externa** que se asigna a tu nueva instancia.

### 2. Instalación de Docker y Configuración

Ahora nos conectaremos a la máquina que acabamos de crear para prepararla.

1. Haz clic en el botón **SSH** junto a tu nueva instancia en la consola de GCP (esto te abre una terminal en el navegador).

2. **Instalar Docker y Docker Compose:**
   (Si usas Debian/Ubuntu, usa estos comandos):

```bash
sudo apt-get update
sudo apt-get install -y docker.io docker-compose
sudo usermod -aG docker $USER
# ¡Importante! Cierra y vuelve a abrir la sesión SSH para que el grupo docker se aplique.
```

3. **Clonar el Repositorio:**

```bash
git clone https://github.com/ximosa/docker-wordpress-sqlite-contenedor.git

# Entra en la carpeta
cd docker-wordpress-sqlite-contenedor
```

4. **Asegurar Permisos de Archivo:**

Para evitar errores de escritura en la base de datos (SQLite), dale permisos a la carpeta de contenidos.

```bash
sudo chown -R www-data:www-data wp-content
```

### 3. Despliegue Final (Build y Run)

Vamos a construir la imagen de Docker usando el Dockerfile que tienes en esta carpeta y luego la levantaremos con la configuración de persistencia del `docker-compose.yml` (que previamente debes crear, ver la sección de Configuración de docker-compose.yml).

1. **Construir y arrancar el contenedor:**

```bash
sudo docker-compose up -d --build
```

(Esto tardará unos minutos la primera vez. La bandera `--build` asegura que el Dockerfile se ejecute correctamente).

2. **Verificar el estado:**

```bash
sudo docker ps -a
```

Asegúrate de que el contenedor (`wordpress_sqlite`) ponga `STATUS: Up`.

### 4. Acceso y Uso Headless

Abre tu navegador y ve a la IP pública de tu instancia:
`http://[TU_IP_PUBLICA]/`
(Verás la pantalla de instalación de WordPress. ¡Ya está online!)

**Configuración Headless:**

1. Una vez instalado, entra al panel de WordPress.
2. Instala y activa el plugin **WPGraphQL**.
3. Instala y activa un plugin como **Headless Mode** para que la portada de tu WordPress no sea visible y solo responda la API.

**Conexión Astro:**

La URL para consumir datos con Astro será: `http://[TU_IP_PUBLICA]/graphql` (o usa el dominio personalizado que configuraste en Cloudflare, ej: `cms.tudominio.com/graphql`).

### 🛠️ Modificar Archivos del Contenedor

Para mantener la persistencia, los archivos se gestionan de forma diferente:

| Archivo/Ubicación | Propósito | Método de Edición |
| :--- | :--- | :--- |
| `wp-content/` | Temas, Plugins, Uploads (Persistentes) | SSH + nano (en la carpeta local) o SFTP a tu VM. |
| `wp-config.php` | Core de WordPress (No es persistente) | Usar docker exec para entrar al contenedor: `sudo docker exec -it wordpress_sqlite /bin/bash` |

<a name="configuracion-de-docker-composeyml"></a>

### 💾 Configuración de docker-compose.yml

Pega esto en un archivo `docker-compose.yml` en la raíz del repositorio. Es necesario para el despliegue.

```yaml
version: '3'
services:
  wordpress:
    build: .
    container_name: wordpress_sqlite
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      # Mapea la carpeta local wp-content al contenedor (PERSISTENCIA)
      - ./wp-content:/var/www/html/wp-content
```

## 🏷️ Available Tags

The available tags correspond to the official WordPress Docker tags. You can find the list of supported tags in the [tags.txt](tags.txt) file.

## 🤝 Contributing 

We welcome contributions! If you'd like to contribute, please follow these steps:

1. Fork the repository.
2. Create a new branch for your feature or bugfix.
3. Make your changes and commit them with a clear commit message.
4. Push your changes to your fork.
5. Submit a pull request to the `main` branch of this repository.

Please ensure your code follows the existing style and includes appropriate tests if applicable.

## 🙏 Acknowledgments 

- [WordPress](https://wordpress.org/) for the amazing CMS.
- [SQLite Database Integration Plugin](https://github.com/WordPress/sqlite-database-integration) for enabling SQLite support in WordPress.

## ❤️ Support ❤️

If you encounter any issues or have questions, please open an issue in the [GitHub repository](https://github.com/yunussandikci/wordpress-sqlite/issues).

## 📄 License 

This project is licensed under the GPL-2.0 License. See the [LICENSE](LICENSE) file for details.
