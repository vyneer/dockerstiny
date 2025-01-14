# dockerstiny

A collection of Dockerfiles and config files that enable you to easily run Destiny.gg (website and chat) locally on your computer for development purposes. Tested on macOS, Linux, and Windows (via Linux on WSL2).

## Requirements
* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/)
* [npm](https://www.npmjs.com/)
* [mkcert](https://github.com/FiloSottile/mkcert)

## Instructions
1. Download/install the requirements above. If using Linux on WSL2, mkcert must be installed on Windows. See the additional Windows mkcert instructions below.
2. Clone this repo.
```
git clone https://github.com/11k/dockerstiny.git
```

3. Navigate into the project folder.
```
cd dockerstiny
```

4. Use `mkcert` to create and install a locally-trusted certificate authority.
```
mkcert -install
```

5. Generate a certificate and private key.
```
mkcert -cert-file docker/nginx-certs/dgg.pem -key-file docker/nginx-certs/dgg-key.pem localhost 127.0.0.1 host.docker.internal
```

6. Copy the CA certificate created by mkcert into `ca-certs`.
```
cp "$(mkcert -CAROOT)/rootCA.pem" docker/ca-certs/
```

7. Clone [`destinygg/website-private`](https://github.com/destinygg/website-private), [`destinygg/chat-private`](https://github.com/destinygg/chat-private), [`destinygg/chat-gui`](https://github.com/destinygg/chat-gui), and [`destinygg/live-ws`](https://github.com/destinygg/live-ws) into `dockerstiny`.
```
git clone git@github.com:destinygg/website-private.git website
git clone git@github.com:destinygg/chat-private.git chat
git clone git@github.com:destinygg/chat-gui.git
git clone git@github.com:destinygg/live-ws.git
```

8. Copy the included website and chat config files to the appropriate locations.
```
cp docker/website-config/config.local.php website/config/
cp docker/website-config/.env website/.env
cp docker/chat-config/settings.cfg chat/
cp docker/live-ws-config/.env live-ws/.env
```

9. Copy the database initialization scripts into the `mysql-scripts` directory. Note the numeric prefixes. This ensures the scripts are executed in the correct order.
```
cp website/config/destiny.gg.sql docker/mysql-scripts/01.destiny.gg.sql
cp website/config/destiny.gg.data.sql docker/mysql-scripts/02.destiny.gg.data.sql
```

10. Install all Node.js dependencies for the chat frontend.
```
cd chat-gui
npm ci
```

11. Install the website's dependencies.
```
cd ../website
npm ci
```

12. Link the local copy of `chat-gui` rather than using the version installed from the npm registry.
```
npm link ../chat-gui
```

13. Generate the site's static assets.
```
npm run build
```

14. Head back into the project folder and build/run. This command will take some time.
```
cd ..
docker-compose --profile dev up
```

15. Access the site via `https://localhost:8080`.
16. Go to `https://localhost:8080/impersonate?username=admin` to log in as the admin.

## Windows mkcert Instructions
1. Download the latest release of mkcert from the project's [releases page on GitHub](https://github.com/FiloSottile/mkcert/releases). The version you need ends with "-windows-amd64.exe".
2. Open Command Prompt.
3. Navigate into the directory that contains the mkcert executable you just downloaded, likely `Downloads`.
```
cd %HOMEPATH%\Downloads
```

4. Create and install a locally-trusted certificate authority. Your executable may have a slightly different name.
```
mkcert-v1.4.3-windows-amd64.exe -install
```

5. Generate a certificate and private key.
```
mkcert-v1.4.3-windows-amd64.exe -cert-file dgg.pem -key-file dgg-key.pem localhost 127.0.0.1 host.docker.internal
```

6. Copy the generated files to the appropriate location. This can be done from within WSL2 by utilizing `wslvar` and `wslpath`.
```
cp $(wslpath "$(wslvar HOMEDRIVE)$(wslvar HOMEPATH)")/Downloads/dgg* docker/nginx-certs
```

8. Do the same with the CA certificate.
```
cp $(wslpath "$(mkcert-v1.4.3-windows-amd64.exe -CAROOT)\rootCA.pem") docker/ca-certs/
```
