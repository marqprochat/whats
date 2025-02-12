## Deploy Ubuntu 22.x

[](https://github.com/unkbot/whaticket-free#deploy-ubuntu-22x)

```shell
  sudo apt-get install -y libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils git
```

Instalar o pacote build-essential:

```shell
sudo apt-get install build-essential
```

```shell
sudo apt update && sudo apt upgrade
```

Instale o node (16.x) e confirme se o comando do node -v e npm -v está disponível:

```shell
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

Instale o docker e adicione seu usuário ao grupo do docker:

```shell
curl -fsSL https://get.docker.com -o get-docker.sh

sudo sh get-docker.sh

sudo usermod -aG docker ${USER}

sudo apt-get install docker-compose
```

Instalar o Postgres Docker

```shell
docker run -e TZ="America/Sao_Paulo" --name postgresql -e POSTGRES_USER=riccol -e POSTGRES_PASSWORD=2025riccol -p 5432:5432 -d --restart=always -v /data:/var/lib/postgresql/data -d postgres
```

Instalar o Redis Docker

```shell
docker run -e TZ="America/Sao_Paulo" --name redis-unkbot -p 6379:6379 -d --restart=always redis:latest redis-server --appendonly yes --requirepass "riccol"
```

Clonar este repositório:

```shell
cd ~
git clone https://github.com/w3nder/whaticket-free.git
```

Crie um arquivo .env de backend e preencha com as informações correta:

```shell
cp whaticket-free/backend/.env.example whaticket-free/backend/.env
nano whaticket-free/backend/.env
```

```shell
NODE_ENV=
BACKEND_URL=http://localhost
FRONTEND_URL=http://localhost:3000
PROXY_PORT=8081
PORT=8081

DB_DIALECT=postgres
DB_HOST=localhost
DB_USER=riccol
DB_PASS=2025riccol
DB_NAME=whats

JWT_SECRET=asdsad
JWT_REFRESH_SECRET=asdasd

REDIS_URI=redis://:riccol@127.0.0.1:6379
REDIS_OPT_LIMITER_MAX=1
REDIS_OPT_LIMITER_DURATION=3000

```

Executa o npm install , cria o build cria as tabela e insere os registro padrão

```shell
cd whaticket-free/backend
npm install
npm run build
npm run db:migrate
npm run db:seed
```

Vá para a pasta frontend e instale as dependências:

```shell
cd ../frontend
cp .env.example .env
nano .env
```

```shell
REACT_APP_BACKEND_URL=https://URL_DO_BACKEND(NAO E URL DO FRONTEND)
REACT_APP_HOURS_CLOSE_TICKETS_AUTO = 24
```

```shell
npm install
npm run build
```

Instale o pm2 **com sudo** e inicie o backend com ele:

```shell
sudo npm install -g pm2

cd ../backend
pm2 start dist/server.js --name unkbot-backend
cd ../frontend
pm2 start server.js --name unkbot-frontend

```

Iniciar pm2 após a reinicialização:

```shell
pm2 startup ubuntu -u `YOUR_USERNAME`
```

Copie a última saída de linha do comando anterior e execute-o, é algo como:

```shell
sudo env PATH=\$PATH:/usr/bin pm2 startup ubuntu -u YOUR_USERNAME --hp /home/YOUR_USERNAM
```

Instale o nginx:

```shell
sudo apt install nginx
```

Remova o site padrão do nginx:

```shell
sudo rm /etc/nginx/sites-enabled/default
```

Crie o site para o Backend

```shell
sudo nano /etc/nginx/sites-available/unkbot-backend
```

```shell
server {
  server_name api.mydomain.com;

  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

Crie o site para o frontend

```shell
sudo nano /etc/nginx/sites-available/unkbot-frontend
```

```shell
server {
  server_name app.mydomain.com;

  location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
```

Crie os links simbólicos para habilitar os sites:

```shell
sudo ln -s /etc/nginx/sites-available/unkbot-backend /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/unkbot-frontend /etc/nginx/sites-enabled
```

Vamos alterar a configuração do nginx para aceitar 20MB de corpo nas requisições:

```shell
sudo nano /etc/nginx/nginx.conf
...

http {
  ...
  client_max_body_size 20M;  # HANDLE BIGGER UPLOADS
}

```

Teste a configuração e reinicie o nginx:

```shell
sudo nginx -t
sudo service nginx restart
```

Agora, ative o SSL (https) nos seus sites para utilizar todas as funcionalidades da aplicação como notificações e envio de mensagens áudio. Uma forma fácil de o fazer é utilizar Certbot:

Instale o certbor com snapd:

```shell
sudo snap install --classic certbot
```

Habilite SSL com nginx:

```shell
sudo certbot --nginx
```
