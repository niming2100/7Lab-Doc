## Fabric and Webassambly

### Fabric

#### Preparations(Centos)
- docker  
- docker-compose
- golang environment
- git

#### Setup

- Clone the [hyperledger/fabric-samples](https://github.com/hyperledger/fabric-samples) repository and put it in $GOPATH/src/
- When you are in the directory into which you will install the Fabric Samples and binaries, go ahead and execute the following command
    

    sudo curl -sSL http://bit.ly/2ysbOFE | bash -s 1.1.0
- <font color="red">Make sure the version tag you pulled is the same as in the command line.</font>

##### Terminal 1 - Start the network
- cd $GOPATH/src/fabric-samples/chaincode-docker-devmode,open the 'docker-compose-simple.yaml'
- Make sure the version of 'service.orderer.image' is same with that above,just like this 'image: hyperledger/fabric-orderer:x86_64-1.1.0' [How to chose version?](https://hub.docker.com/r/hyperledger/fabric-orderer/tags/).

- Execute the following command.


    sudo docker-compose -f docker-compose-simple.yaml up
    
- If you see an error like "could not read directory /etc/hyperledger/msp/signcerts: open /etc/hyperledger/msp/signcerts: permission denied", you should execute "<font color="red">sudo setenforce 0</font>" before this step


##### Terminal 2 - Build & start the chaincode

    docker exec -it chaincode bash
- You should see the following:

    
    root@cd974117055b:/opt/gopath/src/chaincode#
- Now, compile your chaincode:


    cd chaincode_example02/go
    go build -o chaincode_example02
- Now run the chaincode:


    CORE_PEER_ADDRESS=peer:7052 CORE_CHAINCODE_ID_NAME=mycc:0 ./chaincode_example02

- The chaincode is started with peer and chaincode logs indicating successful registration with the peer. Note that at this stage the chaincode is not associated with any channel. This is done in subsequent steps using the instantiate command.


##### Terminal 3 - Use the chaincode

    docker exec -it cli bash

- Now we run a simple example


    peer chaincode install -p chaincodedev/chaincode/chaincode_example02/go -n mycc -v 0
    
    peer chaincode instantiate -n mycc -v 0 -c '{"Args":["init","a","100","b","200"]}' -C myc
    
- Now issue an invoke to move 10 from a to b


    peer chaincode invoke -n mycc -c '{"Args":["invoke","a","b","10"]}' -C myc

- Finally, query b. We should see a value of 210.


    peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc
    
    
    
### Webassembly

#### Preparations(Centos)
- c++ sdk(yum install gcc,yum install gcc++)
- cmake
- python
- git


#### Setup

    $ git clone https://github.com/juj/emsdk.git
    $ cd emsdk
    $ ./emsdk install --build=Release sdk-incoming-64bit binaryen-master-64bit
    $ ./emsdk activate --build=Release sdk-incoming-64bit binaryen-master-64bit
    
- It will take a long time to install.When it finished


    source ./emsdk_env.sh --build=Release

- <font color="red">Try command emcc,if it shows command not found,execute the command below,then try again</font>


    ln -s /home/Polarbear/Workspace/emsdk/emscripten/incoming/emcc /usr/bin/emcc
    ! replace'/home/Polarbear/Workspace/' to yours

- The commands below will create a simple “hello world” program and compile it.


    $ mkdir hello
    $ cd hello
    $ cat << EOF > hello.c
    #include <stdio.h>
    int main(int argc, char ** argv) {
      printf("Hello, world!\n");
    }
    EOF
    $ emcc hello.c -s WASM=1 -o hello.html

- You will see such files in your folder


    hello.c
    hello.html
    hello.js
    hello.wasm

- You can not open 'hello.html'  directly, you need to put it in a web container or run a http-server,and open it in your browser(firefox,chrome,safari)
- Finally you will see the 'Hello world' printed to the Emscripten console in the page.