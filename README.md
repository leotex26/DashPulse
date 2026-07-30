# DashPulse

<p align="center">
  <strong>Un tableau de bord personnel moderne développé avec Vue.js</strong>
</p>

---

## À propos du projet

**DashPulse** est une application web de tableau de bord (dashboard) construite à partir de la base open-source *Homer*, que j'ai personnalisée et enrichie avec des composants sur-mesure.

L'objectif principal est d'offrir une interface centralisée, réactive et élégante pour regrouper mes outils de développement et suivre mes métriques en temps réel.

### Fonctionnalités clés
-  **Widget GitHub réactif :** Intégration d'un composant Vue.js sur-mesure consommant l'API REST de GitHub pour afficher les métriques en direct (repos, followers, avatar).
-  **Recherche rapide :** Filtrage instantané des services enregistrés.
-  **Mode Sombre / Clair :** Gestion dynamique du thème visuel.
-  **Design Responsive :** Adapté aux écrans mobiles et desktop.
-  **Configuration simple :** Gestion des raccourcis et liens via fichier YAML.

---

## Stack Technique

- **Framework :** [Vue.js 3](https://vuejs.org/)
- **Langages :** JavaScript (ES6+), HTML5, CSS3 / Sass
- **API :** GitHub REST API (Fetch)
- **Outil de build :** Vite / Webpack

---

## Installation & Lancement en local

**Start the container with `docker run`**

```sh
# Make sure your local config directory exists
docker run -d \
  --name homer \
  -p 8080:8080 \
  --mount type=bind,source="/path/to/config/dir",target=/www/assets \
  --restart=unless-stopped \
  b4bz/homer:latest
```

> [!NOTE]  
> The container will run using a user uid and gid 1000 by default, add `--user <your-UID>:<your-GID>` to the docker command to adjust it if necessary. Make sure this match the permissions of your assets directory.

**or `docker-compose`**

```yaml
services:
  homer:
    image: b4bz/homer
    container_name: homer
    volumes:
      - /path/to/config/dir:/www/assets # Make sure your local config directory exists
    ports:
      - 8080:8080
    user: 1000:1000 # default
    environment:
      - INIT_ASSETS=1 # default, requires the config directory to be writable for the container user (see user option)
    restart: unless-stopped
```

**Environment variables:**

- **`INIT_ASSETS`** (default: `1`)
Install example configuration file & assets (favicons, ...) to help you get started.

- **`SUBFOLDER`** (default: `null`)
If you would like to host Homer in a subfolder, (ex: *<http://my-domain/homer>*), set this to the subfolder path (ex `/homer`).

- **`PORT`** (default: `8080`)
If you would like to change internal port of Homer from default `8080` to your port choice.

- **`IPV6_DISABLE`** (default: 0)
Set to `1` to disable listening on IPv6.

### Using the release tarball (prebuilt, ready to use)

Download and extract the latest release (`homer.zip`) from the [release page](https://github.com/bastienwirtz/homer/releases), rename the `assets/config.yml.dist` file to `assets/config.yml`, and put it behind a web server.

```sh
wget https://github.com/bastienwirtz/homer/releases/latest/download/homer.zip
unzip homer.zip -d homer
cd homer
cp assets/config.yml.dist assets/config.yml
pnpx http-server # or python -m http.server 8010 or any web server.
```

### Build manually

```sh

<img width="1211" height="549" alt="image" src="https://github.com/user-attachments/assets/46e01f78-7bdd-4fb1-b749-dab863090907" />

pnpm install
pnpm build
```

Then your dashboard is ready to use in the `/dist` directory.



