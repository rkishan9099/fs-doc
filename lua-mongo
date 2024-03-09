LUA MONGO INSTALATION 

configure freeswitch 

STEP 1
./configure --enable-lua --with-lua=lua5.3
make
sudo make install

STEP 2
install mongo c driver
wget https://github.com/mongodb/mongo-c-driver/releases/download/1.9.4/mongo-c-driver-1.9.4.tar.gz
tar xzf mongo-c-driver-1.9.4.tar.gz
cd mongo-c-driver-1.9.4
./configure --disable-automatic-init-and-cleanup
make 
sudo make install

STEP 3 
install lua-mongo
git clone https://github.com/neoxic/lua-mongo.git
cd lua-mongo
sudo apt-get install liblua5.3-dev
vim CMakeLists.txt
set(USE_LUA_VERSION 5.3)
cmake .
make 
sudo make install


