## 프라이빗 이더리움 구축하기

<br>

#### 설치

```
virtualBox
vagrant
```

<br>

#### 디렉토리 이동

```
dev/eth_prac001
vagrant init
```

디렉토리에 Vagrantfile이 생성된다.

주석을 모두 지우고 아래와 같이 수정함

```
Vagrant.configure("2") do |config|
  config.vm.define "eth01" do |eth01|
    eth01.vm.box = "ubuntu/bionic64"
    eth01.vm.hostname = "eth01"
    eth01.vm.network "private_network", ip: "192.168.50.10"
    eth01.vm.provider "virtualbox" do |eth01v|
      eth01v.memory = 4096
    end
  end
  config.vm.define "eth02" do |eth02|
    eth02.vm.box = "ubuntu/bionic64"
    eth02.vm.hostname = "eth02"
    eth02.vm.network "private_network", ip: "192.168.50.11"
    eth02.vm.provider "virtualbox" do |eth02v|
      eth02v.memory = 4096
    end
  end
end
```

<br>

<br>

#### 가상 머신 구동

```
vagrant up eth01
vagrant up eth02
```

<br>

#### 가상 머신 접속

```
vagrant ssh eth01
vagarnt ssh eth02
```

<br>

#### Geth(Go-ethereum client) 설치

```
// eth01 가상머신과 eth02 가상머신에서 각각 수행

sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get install ethereum


// 설치 확인
geth version
```

<br>

eth01 가상머신에서 디렉토리 생성 이후, 프라이빗 이더리움을 위한 genesis 블록파일 생성

```
mkdir -p dev/eth_localdata
cd dev/eth_localdata

vi CustomGenesis.json
```

<br>

#### CustomGenesis.json

```json
{
  "config": {
    "chainId": 921,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
  },
  "alloc": {},
  "coinbase": "0x0000000000000000000000000000000000000000",
  "difficulty": "0x20",
  "extraData": "",
  "gasLimit": "0x47e7c5",
  "nonce": "0x0000000000000042",
  "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp": "0x00"
 }
```

<br>

#### Geth 초기화

> eth01, eth02에서 Geth 초기화

```
geth --datadir /home/vagarnt/dev/eth_localdata init /home/vagarnt/dev/eth_localdata/CustomGenesis.json
```

<br>

#### Geth 구동

> eth01과 eth02에서 각각 다른 포트로 수행한다

```
geth --networkid 921 --maxpeers 2 --datadir /home/vagarnt/dev/eth_localdata --port 30303 console

geth --networkid 921 --maxpeers 2 --datadir /home/vagarnt/dev/eth_localdata --port 30304 console
```

<br>

#### 노드 연결

eth01에서 `admin.nodeInfo.enode`를 터미널에 입력했을 때 나오는 enode 값을 복사한 뒤

eth02 터미널에서 아래와 같이 입력

```
admin.addPeer(복사내용)

제대로 입력했으면 'true'라는 결과가 나옴
```

노드 연결을 확인하자

```
admin.peers
```

포트 충돌이 일어나지 않으면 노드 정보가 출력된다.

<br>

<br>

### 이더리움 계정(EOA) 생성

---

eth01과 eth02 가상머신에서 각각 새로운 계정을 만들어보자

```
personal.newAccount("test1234")
```

터미널에 입력하면 주소가 하나 나온다.

#### 계정 확인

```
eth.accounts
```

<br>

#### 트랜잭션 생성

> 트랜잭션 생성을 위한 이더를 채굴한다

eth01 가상머신에 입력

```
miner.start(1)
```

20여개의 블록 채굴 확인 후 mining 종료

```
miner.stop()
```

채굴 보상으로 획득한 이더 잔액확인

```
eth.getBalance(계정주소)
```

<img src="https://github.com/kim6394/PrivateEthereum/blob/master/screenshot/%EC%B1%84%EA%B5%B4.PNG?raw=true">

어마어마한 부자가 되었다..

<br>

트랜잭션을 생성하기 전에 먼저 계정을 unlock 한다. (eth01과 eth02 둘 다)

```
web3.personal.unlockAccount("계정주소");

패스워드는 내가 입력한 걸 넣는다. (현재 실습에서는 test1234)
Unlock이 완료되면 true라고 나온다.
```



이제 트랜잭션을 진행한다.

```
eth.sendTransaction({from:"eth01 계정주소", to: "eth02 계정주소", value: web3.toWei(1, "ether")})
```

결과 값으로 txhash 값이 나오게 된다.

<img src="https://github.com/kim6394/PrivateEthereum/blob/master/screenshot/txhash.PNG?raw=true">

<br>

txhash 값을 가져와서 트랜잭션을 확인해보자

<br>

<img src="https://github.com/kim6394/PrivateEthereum/blob/master/screenshot/transaction%20%ED%99%95%EC%9D%B8.PNG?raw=true">

현재 블록해시값과 넘버값이 null이다.

다시 채굴을 진행하고, 6개정도 채굴이 되면 트랜잭션 값을 다시 확인해보자

<br>

<img src="https://github.com/kim6394/PrivateEthereum/blob/master/screenshot/transaction%20%EC%9E%AC%ED%99%95%EC%9D%B8.PNG?raw=true">

이제 블록값을 확인할 수 있다!

채굴된 블록값을 확인해보자

<br>

<img src="https://github.com/kim6394/PrivateEthereum/blob/master/screenshot/%EC%B1%84%EA%B5%B4%EB%90%9C%20%EB%B8%94%EB%A1%9D%ED%99%95%EC%9D%B8.PNG?raw=true">

