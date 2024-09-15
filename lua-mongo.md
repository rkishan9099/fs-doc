# *Install Lua-mongo*

#### *configure freeswitch*

```bash
  ./configure --enable-lua --with-lua=lua5.3
  make
  sudo make install

```

### *Install mongo c driver*

```bash
wget https://github.com/mongodb/mongo-c-driver/releases/download/1.9.4/mongo-c-driver-1.9.4.tar.gz

tar xzf mongo-c-driver-1.9.4.tar.gz

cd mongo-c-driver-1.9.4

./configure --disable-automatic-init-and-cleanup

make 
sudo make install
```

### *Install lua-mongo*

Install MongoDB C Driver (libmongoc)
```bash
sudo apt-get update
sudo apt-get install libmongoc-1.0-0 libmongoc-dev
```

Install Lua and LuaRocks
```bash
sudo apt-get install lua5.3 luarocks
```

 Install lua-mongo using LuaRocks
 ```bash
sudo luarocks install lua-mongo
```

Verify Installation
```bash
luarocks list
```
