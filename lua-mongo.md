
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

```bash
git clone https://github.com/neoxic/lua-mongo.git

cd lua-mongo
sudo apt-get install liblua5.3-dev

vim CMakeLists.txt
set(USE_LUA_VERSION 5.3)

cmake .
make 
sudo make install
```
