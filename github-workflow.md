## GitHub WorkFlow

```
name: 📦 Install Node Modules

on:
  workflow_dispatch:  # Allows manual triggering from GitHub UI

jobs:
  deploy:
    name: 🛠 Deploy FE-DigitDial
    runs-on: ubuntu-latest  # GitHub-hosted runner
    strategy:
      matrix:
        node-version: [20.11.1]

    steps:
      - name: 📥 Get latest code
        uses: actions/checkout@v2

      - name: 🏷 Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}

      - name: 📦 Install NPM Modules
        run: npm install

      - name: 🔑 Install SSH Key
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          known_hosts: ${{ secrets.SSH_HOST }} # Set known_hosts here or use the step below

      - name: 🗝 Adding Known Hosts
        run: ssh-keyscan -p 22 -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      - name: 🚚 Deploy with rsync
        run: rsync -avz -e "ssh -p 22" node_modules ubuntu@${{ secrets.SSH_HOST }}:/var/www/digitdial-pbx/  # Set known_hosts here or use the step below
```

### Nextjs Project Workflow

```
name: 🚀 Deploy DigitDial Frontend

on:
  workflow_dispatch:  # Allows manual triggering from GitHub UI

jobs:
  deploy:
    name: 🛠 Deploy FE-DigitDial
    runs-on: ubuntu-latest  # GitHub-hosted runner
    strategy:
      matrix:
        node-version: [20.11.1]

    steps:
      - name: 📥 Get latest code
        uses: actions/checkout@v2

      - name: 🏷 Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}
      
      - name: 📦 Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            ${{ github.workspace }}/.next/cache
            node_modules
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-

      - name: 📦 Install NPM Modules
        run: npm install

      - name: 🏗 Build Project
        run:  npm run build

      - name: 🔑 Install SSH Key
        uses: shimataro/ssh-key-action@v2
        with:
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          known_hosts: ${{ secrets.SSH_HOST }}  # Set known_hosts here or use the step below

      - name: 🗝 Adding Known Hosts
        run: ssh-keyscan -p 22 -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      - name: 🚚 Deploy with rsync
        run: rsync -avz -e "ssh -p 22" .next package.json package-lock.json next.config.mjs  ubuntu@${{ secrets.SSH_HOST }}:/var/www/digitdial-pbx


      - name: 🔄 Restart PM2 digitdial-fe
        run: ssh ubuntu@${{ secrets.SSH_HOST }} '/home/ubuntu/.nvm/versions/node/v20.17.0/bin/pm2 restart digitdial-pbx'
```
