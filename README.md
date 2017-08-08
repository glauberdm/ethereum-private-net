# An Ethereum private-net with [geth](https://geth.ethereum.org/downloads/) and [Mist](https://github.com/ethereum/mist/releases)
Os comandos a seguir foram realizados em Unix-like, porém podem ser executados em qualquer Sistema Operacional com os softwares necessários.

## Rede RPC local com geth

### Criando a rede com arquivo [genesis.json](http://ethdocs.org/en/latest/network/test-networks.html#the-genesis-file) e definindo diretório de dados da rede

```
../ethereum-private-net$ geth --datadir private-net init genesis.json
```
```
WARN [08-04|14:58:29] No etherbase set and no accounts found as default
INFO [08-04|14:58:29] Allocated cache and file handles         database=/Users/glauber/job/finchain/courses/dev/eth/private-net/geth/chaindata cache=16 handles=16
INFO [08-04|14:58:29] Writing custom genesis block
INFO [08-04|14:58:29] Successfully wrote genesis state         database=chaindata                                                              hash=8cb7a7…344b56
INFO [08-04|14:58:29] Allocated cache and file handles         database=/Users/glauber/job/finchain/courses/dev/eth/private-net/geth/lightchaindata cache=16 handles=16
INFO [08-04|14:58:29] Writing custom genesis block
INFO [08-04|14:58:29] Successfully wrote genesis state         database=lightchaindata                                                              hash=8cb7a7…344b56
```
> genesis.json
```
{
  "config": {
        "chainId": 10,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x3333333333333333333333333333333333333333",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000000042",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```
O diretório `../ethereum-private-net` passará a ter o diretório passado no parâmetro `--datadir`, no caso, `private-net`:
```
private-net/
├── geth
│   ├── chaindata
│   │   ├── 000001.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000000
│   └── lightchaindata
│       ├── 000001.log
│       ├── CURRENT
│       ├── LOCK
│       ├── LOG
│       └── MANIFEST-000000
└── keystore
```
### Executando a rede RPC local e acessando o console geth
```
geth --rpc --rpccorsdomain "*" --rpcapi eth,web3,personal --datadir private-net console 2> console.log
```
> Para saber mais sobre os parâmetros `rpc`, `rpccorsdomain`, `rpciapi` vide [ethdocs](http://ethdocs.org/en/latest/network/test-networks.html#command-line-parameters-for-private-network). Iremos falar sobre eles mais tarde.

Console geth:
```
Welcome to the Geth JavaScript console!

instance: Geth/v1.6.6-stable-10a45cb5/darwin-amd64/go1.8.3
coinbase: 0x7c5d4294f41f3ccaec6c3c1739eb4e37d498fdb2
at block: 0 (Wed, 31 Dec 1969 21:00:00 -03)
 datadir: ../ethereum-private-net/private-net
 modules: admin:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0

>
```
### Criando contas
Inicialmente a rede não possui nenhuma conta:
```
> eth.accounts
```
```
[]
```
Vamos criar duas contas para uso da rede privada.

#### Conta 1 - phrase: test01
```
> personal.newAccount('test01')
```
```
"0x7c5d4294f41f3ccaec6c3c1739eb4e37d498fdb2"
```

#### Conta 2 - phrase: test02
```
> personal.newAccount('test02')
```
```
"0xac291f42f5c68b0c535a022a912d1c23dcbbb95c"
```

Agora possuímos duas contas:
```
> eth.accounts
```
```
["0x7c5d4294f41f3ccaec6c3c1739eb4e37d498fdb2", "0xac291f42f5c68b0c535a022a912d1c23dcbbb95c"]
```
Observe que agora que o diretório `private-net/keystore` possui dois arquivos correspondente as keys das contas criadas:
```
private-net/
├── geth
│   ├── LOCK
│   ├── chaindata
│   │   ├── 000012.log
│   │   ├── 000014.ldb
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000013
│   ├── lightchaindata
│   │   ├── 000001.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000000
│   ├── nodekey
│   └── nodes
│       ├── 000010.log
│       ├── 000012.ldb
│       ├── CURRENT
│       ├── LOCK
│       ├── LOG
│       └── MANIFEST-000011
├── history
└── keystore
    ├── UTC--2017-08-04T17-18-01.981294567Z--7c5d4294f41f3ccaec6c3c1739eb4e37d498fdb2
    └── UTC--2017-08-04T17-18-19.826605056Z--ac291f42f5c68b0c535a022a912d1c23dcbbb95c
```
### Minerando
No console geth, vamos usar a função `miner.start(n)` onde `n` é a quantidade de threads. 
```
> miner.start(4)
```
```
null
```
```
eth.mining
```
```
true
```
Iniciada a mineração, podemos consultar o saldo da conta phrase *test01*. A primeira conta criada é a conta default para mineração:
```
> eth.getBalance(eth.accounts[0])
```
```
0
```
A mineração, na primeira vez, demora a aconter. Essa demora é devido a montagem do DAG (posteriormente irei escrever sobre isso). Abrindo um novo terminal, podemos observar esta comportamento `../ethereum-private-net$ tail -f console.log`:
```
INFO [08-04|16:15:01] Updated mining threads                   threads=4
INFO [08-04|16:15:01] Transaction pool price threshold updated price=18000000000
INFO [08-04|16:15:01] Starting mining operation
INFO [08-04|16:15:01] Commit new mining work                   number=1 txs=0 uncles=0 elapsed=2.122ms
INFO [08-04|16:15:07] Generating DAG in progress               epoch=0 percentage=0 elapsed=3.270s
INFO [08-04|16:15:10] Generating DAG in progress               epoch=0 percentage=1 elapsed=6.316s
INFO [08-04|16:15:13] Generating DAG in progress               epoch=0 percentage=2 elapsed=9.275s
```
Observe que há o valor `percentage=` com atualização constante.

Ao final do DAG temos o início da mineração:
```
INFO [08-04|16:21:21] Generating DAG in progress               epoch=0 percentage=98 elapsed=6m17.231s
INFO [08-04|16:21:26] Generating DAG in progress               epoch=0 percentage=99 elapsed=6m22.051s
INFO [08-04|16:21:26] Generated ethash verification cache      epoch=0 elapsed=6m22.057s
INFO [08-04|16:21:40] Generating ethash verification cache     epoch=1 percentage=27 elapsed=3.045s
INFO [08-04|16:21:43] Generating ethash verification cache     epoch=1 percentage=59 elapsed=6.261s
INFO [08-04|16:21:46] Generating ethash verification cache     epoch=1 percentage=99 elapsed=9.265s
INFO [08-04|16:21:47] Generated ethash verification cache      epoch=1 elapsed=9.365s
INFO [08-04|16:21:53] Generating DAG in progress               epoch=1 percentage=0  elapsed=6.400s
INFO [08-04|16:21:59] Generating DAG in progress               epoch=1 percentage=1  elapsed=11.807s
INFO [08-04|16:22:04] Generating DAG in progress               epoch=1 percentage=2  elapsed=17.093s
INFO [08-04|16:22:05] Successfully sealed new block            number=1 hash=5047bd…11306b
INFO [08-04|16:22:05] 🔨 mined potential block                  number=1 hash=5047bd…11306b
INFO [08-04|16:22:05] Commit new mining work                   number=2 txs=0 uncles=0 elapsed=3.544ms
INFO [08-04|16:22:11] Generating DAG in progress               epoch=1 percentage=3  elapsed=24.007s
INFO [08-04|16:22:14] Successfully sealed new block            number=2 hash=676040…3b05e1
INFO [08-04|16:22:14] 🔨 mined potential block                  number=2 hash=676040…3b05e1
INFO [08-04|16:22:14] Commit new mining work                   number=3 txs=0 uncles=0 elapsed=305.393µs
INFO [08-04|16:22:15] Successfully sealed new block            number=3 hash=3563c1…c88a1c
INFO [08-04|16:22:15] 🔨 mined potential block                  number=3 hash=3563c1…c88a1c
```
Voltando ao terminal geth podemos consultar o saldo novamente:
```
> eth.getBalance(eth.accounts[0])
```
```
225000000000000000000
```

## Mist - Uma interface amigável

### Conectando o Mist à rede RPC local

A rede local precisa estar executando, sugiro abrir um novo terminal para as ações a seguir.

#### Windows

Para conectar o Mist à rede no S.O. Windows basta abrir o Mist, ele automaticamente detecta a rede criado pelo geth.

### Linux

Considerando que o Mist foi configurado no PATH, executar:
```
../ethereum-private-net$ mist --rpc private-net/geth.ipc
```

### Mac OS X
```
/Applications/Mist.app/Contents/MacOS/Mist --rpc ~/ethereum-private-net/private-net/geth.ipc # ~ representa /Users/<seu_usuario>
```
![alt text](https://github.com/glauberdm/ethereum-private-net/blob/master/misc/mist_splah.png)

Observe no topo superior direito o texto *PRIVATE-NET*, indicando que você está conectado à rede criada no geth.

No Mac OS X é preciso clicar no botão *LAUNCH APPLICATION*. Então a home do Mist será apresentada:

![alt text](https://github.com/glauberdm/ethereum-private-net/blob/master/misc/mist_home.png)

A *MAIN ACCOUNT* é apresentada com alguns Ethers devido a mineração realizada. Se a mineração estiver executando, você observará a quantidade de Ether ser alterada.

A partir desse momento, você já pode realizar transações entre as contas e realizar deploy de contratos.
