---
id: evm
title: イーサリアム仮想マシン
sidebar_label: EVM
---
## 概要

Loom DAppチェーンはイーサリアム仮想マシン(EVM)を含み、EVMバイトコードへコンパイルされるスマートコントラクトのデプロイと実行が可能となる。

### イーサリアム仮想マシン

EVMを構成するのは、データベース及びEVMバイトコードインタプリタである。

このインタプリタはEVMのバイトコードを実行し、ブロックチェーンに適した安全な決定的プログラムを作るために特別に設計されている。 EVMスマートコントラクトをコーディングするのに最も人気のある言語はSolidityであるが、EVMバイトコードにコンパイルされる言語ならどれでも、EVMインタプリタによって実行可能だ。

このデータベースは、EVMにデプロイされた各プログラムのアドレスをキーとしている。そのバリューは、プログラムのバイトコードと関連データを含む。

### DAppチェーンとEVM

現在、DAppチェーンのEVMと対話するいくつかの方法がある。

1. スマートコントラクトは、ブロックチェーンの初期起動時にデプロイできる。
2. Loomのコマンドラインツールは、スマートコントラクトのデプロイ、またはすでにデプロイ済みのコントラクト上のメソッドの呼び出しを可能にする。
3. EVMコントラクトもしくはプラグインコントラクトのどちらからでも、既にデプロイ済みの別のEVMコントラクト上のメソッドを呼び出すことができる。
4. Goだとgo-loomのEvmContractオブジェクトを使用できる。
5. TypeScriptやJavaScriptではloom-jsのEvmContractオブジェクトを使用する。
    
    EVMスマートコントラクトは、コンパイルされたバイトコード形式でDAppチェーンにデプロイされる。 このためチェーンは親言語を認識しない。 Solidityのスマートコントラクトメソッドの呼び出しパラメーターは、[Solidityのウェブサイトに記載されている](https://solidity.readthedocs.io/en/develop/abi-spec.html)アプリケーションバイナリインターフェイス (ABI) でエンコードされる。 このABIは非常に複雑になるのだが、後ほど取り上げるように、イーサリアムの実装はパラメーター生成をサポートする関数を与えなくてはならない。
    
    ## 立ち上げ時のデプロイ
    
    コンパイル済みのコードをコントラクトディレクトリ内に配置し`genesis.json` ファイルをリンクすることで、立ち上げ時のDAppチェーンにデプロイすることができる。
    
    これはジェネシス・ファイルのサンプルだ。 ```json { "contracts": [ { "vm": "EVM", "format": "truffle", "name": "SimpleStore", "location": "/path/to/loomchain/contracts/SimpleStore.json" }, { "vm": "plugin", "format": "plugin", "name": "evmexample", "location": "evmexample:1.0.0", "init": {
    
             }
         }
        
    
    ] } 

    配列の先頭に2つのコントラクトがある。 1つ目はEVMコントラクトで、2つ目はプラグインコントラクトだ。
    * `vm:`コントラクト実行に使用する仮想マシン。 現在2つのオプションがある。
      1. `plugin`   ユーザーがコントラクトを作成。
      2. `EVM`      コントラクトは、DAppチェーンEVM上で実行される。
    * `format` コントラクトディレクトリ内にあるスマートコントラクトのインプットファイルの性質。
      1. `plugin`   ユーザープラグイン。`go-loom`で作成可能。
      2. `truffle`  truffleのコンパイラを使用してコンパイルされたSolidityプログラム。
      3. `solidity` solcを使用してコンパイルされたSolidityプログラム。
      4. `hex`      Raw Hex。Solidityプログラムのインスタンスには`solc -o`のオプションを使用してコンパイルされる。
      .
    * `name` これはLoomもしくはEVMから割り当てられたコントラクトアドレスを取得するのに使用される。
    * `location`コントラクトディレクトリ内に配置されたバージョン化されたバイナリファイル名。
     truffleとsolidityには完全なpathを与えなければならない可能性がある。
    
    そのためこの例では、Loom DAppチェーンはSolidityコントラクト・SimpleStoreのtruffleコンパイルからバイトコードを受け取ることとなる。 そうしてこれをチェーンのEVMにデプロイする。 Loomのログ情報でその確認とコントラクトアドレスが利用可能となる。
    
    ## コマンドラインよりデプロイ及び実行する
    
    Loomのコマンドラインツールには、チェーンのEVMと対話するための3つのコマンドがある。
    * `deploy`チェーンのEVM上に、EVMバイトコードでスマートコントラクトをデプロイする。
    * `call` 既にデプロイ済みのEVMスマートコントラクトにある状態変更メソッドを呼び出す。
    * `static-call` 既にデプロイ済みのEVMスマートコントラクトにある読み取り専用メソッドを呼び出す。
    
    
    ### Deploy
    Use `./loom deploy` to deploy a contract, that can be compiled to EVM 
    bytecode, onto a DAppChains EVM. 
    ```text
    Deploy a contract 
    
    Usage:
      loom deploy [flags]
    
      -a, --address string    address file
      -b, --bytecode string   bytecode file
          --chain string      chain ID (default "default")
      -h, --help              help for deploy
      -k, --key string        private key file
      -n, --name string       contract name
      -r, --read string       URI for quering app state (default "http://localhost:46658/query")
      -w, --write string      URI for sending txs (default "http://localhost:46658/rpc")
    

-a 及び -k のフラグは、公開鍵および秘密鍵のアドレスファイルでユーザーを特定するのに使用される。

-bはコントラクトのEVMのローバイトコードがあるファイルを提供する。 これは`solc --bin -o. 
 MySolProgram.sol`といったSolidityコンパイラを使って生成可能だ。

-n はコントラクトの名前を入力できるようにする。これはコントラクトアドレスよりも、よりユーザーフレンドリーなハンドルとして機能する。

例: 

    text
     ./loom deploy -a ./data/pub -k ./data/pri -b ./data/bytecode.bin  -w \
      http://localhost:46657 -r http://localhost:9999

  
If everything works you should see something like: ```text New contract deployed with address: default:0x71A53d11A3b77e369463804FEE9B17ba7E24d98B Runtime bytecode: [96 96 96 64 82 ... 84 226 214 187 0 41] Transcation receipt: [10 178 198 52 108 ... 141 155 79 250 97 129 104 243]

    出力されたコントラクトアドレスは、callコマンドでコントラクトのメソッドを呼び出すのに使用できる。
    The uinique [transaction hash](https://loomx.io/developers/docs/en/evm.html#transaction-receipt)
    can be used to retrive a receipt of the deployment transaction. 
    
    ### call
    
    ```text
    Call a method on a contract that can mutate the state
    
    Usage:
      loom call [flags]
    
    Flags:
      -a, --address string         address file
          --chain string           chain ID (default "default")
      -c, --contract-addr string   contract address
      -n, --contract-name string   contract name
      -h, --help                   help for call
      -i, --input string           file with input data
      -k, --key string             private key file
      -r, --read string            URI for quering app state (default "http://localhost:46658/query")
      -w, --write string           URI for sending txs (default "http://localhost:46658/rpc")
    

-a 及び -k のフラグは、公開鍵および秘密鍵のアドレスファイルでユーザーを特定するのに使用される。

-c はコントラクトアドレスを要求する。これは前述の`\loom deploy`へのコール結果、もしくは立ち上げ時のログから検索したものとなりうる。

-n はコントラクトデプロイ時に入力された名前やラベルであり、アドレスを代替するものとして使用可能だ。

-i は入力文字列だ。Solidityのコントラクトでは、これは[Solidity ABI documentation](https://solidity.readthedocs.io/en/develop/abi-spec.html)で説明されているようにABIにエンコーディングされる。

例 ```text call -a ./data/pub -k ./data/pri -i ./cmd/loom/data/inputSet.bin \ -c 0xbD770416A3345f91E4b34576Cb804a576Fa48eB1 \ -w http://localhost:46657 -r http://localhost:9999

    これが完了すると、 [トランザクションのハッシュ値](https://loomx.io/developers/docs/en/evm.html#transaction-receipt)が返却されるが、これは各トランザクションコールに対し唯一であり同じものはない。 It can be used to return a receipt of the transaction.
    
    ### static-call
    Call a read only method on a contract. Returns the method return value.
    ```text
    Usage:
      loom static-call [flags]
    
    Flags:
          --chain string           chain ID (default "default")
      -c, --contract-addr string   contract address
      -n, --contract-name string   contract name
      -h, --help                   help for static-call
      -i, --input string           file with input data
      -r, --read string            URI for quering app state (default "http://localhost:46658/query")
      -w, --write string           URI for sending txs (default "http://localhost:46658/rpc")
      -a, --address string         address file
          --chain string           chain ID (default "default")
      -k, --key string             private key file
    

-a 及び -k のフラグは、公開鍵および秘密鍵のアドレスファイルでユーザーを特定するのに使用される。

-c はコントラクトアドレスを要求する。これは前述の`\loom deploy`へのコール結果、もしくは立ち上げ時のログから検索したものとなりうる。

-n はコントラクトデプロイ時に入力された名前やラベルであり、アドレスを代替するものとして使用可能だ。

-i は入力文字列だ。 Solidityのコントラクトでは、これは[Solidity ABI documentation](https://solidity.readthedocs.io/en/develop/abi-spec.html)で説明されているようにABIにエンコーディングされる。 Example

The address fields -a and -k are optional. ```text static-call -a ./data/pub -k ./data/pri -i ./cmd/loom/data/inputGet.bin \ -c 0xbD770416A3345f91E4b34576Cb804a576Fa48eB1 \ -w http://localhost:46657 -r http://localhost:9999

    <br />## ユーザープラグインから
    
    DAppチェーンのEVMにデプロイされたコントラクトは、ユーザーが作成したプラグインから呼び出すことができる。 go-loomのevmexampleの例は、これを実現する方法の例となる。 
    
    続きをやる前に、関連する様々なモジュールについて考えてみよう。
    
    * ユーザーアプリケーション。 これはエンド ユーザーのアプリケーションで、DAppチェーン上でのトランザクションを引き起こす。 
    
    * DAppチェーン。 ユーザーアプリケーションからトランザクションを受信し、適切なコントラクトへと転送し実行する。 またブロックチェーンに結果をコミットする。
    
    * スマートコントラクト。 ユーザーによって書かれ、DAppチェーン上にデプロイされる。 
    これには2つの主なタイプがある。
        1. プラグイン。 RPCでサポートされていれば、どんな言語で書くことも可能だ; 
        go-loom はGoで書かれたコントラクトの使用を簡単にし、またloom-jsは 
        javascript向けのものである。 このプラグインはDAppチェーンがgRPCを使って
        呼び出せるようなものへコンパイルされる。
        2. EVMスマートコントラクト SolidityのプログラムやEVMバイトコードにコンパイルされる
         何か他のコードはDAppチェーンでEVMを使って実行することができる。
    
    プラグインは、EVM上にデプロイ済みのものも含めて他のコントラクトを実行することができる。これはgRPCを使ってDAppチェーンへコールバックするすることで行われる。 逆はしかし真ではない。だが
    
    ### ユーザーのコード
    
    ユーザーは2つのコードを提供する。 それはスマートコントラクトと、DAppチェーンを使用するエンドアプリケーションだ。
    
    以下では、Goがエンドアプリケーションに使用されていること、さらにスマートコントラクトがプラグイン用のGo、もしくはEVM用のSolidity、このどちらかで書かれていることと仮定していく。 javaScript向けのソリューションは、loom-js-quickstart.mdを参照のこと。
    
    ### 最小限のプラグイン
    
    まず、Go-loomでのコントラクト定義について見ていこう。
    ```go
    type Contract interface {
        Meta() (plugin.Meta, error)
    }
    

and plugin.Meta is defined from a protobuf definition

```go
type ContractMeta struct {
    Name    string `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
    Version string `protobuf:"bytes,2,opt,name=version,proto3" json:"version,omitempty"`
}
```

So all a contract needs is to implement the Meta function. However to be usable as a plugin in a DAppChain there are a few other bits. Here is a minimal example.

```go
package main

import (
    "github.com/loomnetwork/go-loom/plugin"
    "github.com/loomnetwork/go-loom/plugin/contractpb"
)

type HelloWorld struct {
}

func (c *HelloWorld) Meta() (plugin.Meta, error) {
    return plugin.Meta{
        Name:    "HelloWorld",
        Version: "1.0.0",
    }, nil
}

var Contract plugin.Contract = contractpb.MakePluginContract(&HelloWorld{})

func main() {
    plugin.Serve(Contract)
}
```

Here are some points of interest. 1. First the contract has to be package main. 2. Define our contract called HelloWorld as a struct. 3. Implement the `Meta()` function, returning the contracts name and version number. 4. The variable `Contract` needs to be defined. The function `contract
.MakePluginContract` converts our simple outline into an object that a DAppChain can communicate with. 5. The main routine can then sets the contract up as a working server.

Of course our contract has no functionality so can't do anything. The next step is to add some. The MakePluginContract function can then use reflection to learn any new methods we give to our contract.

### 関数の追加

```go
func (c *HelloWorld) Hello(ctx contract.StaticContext, req *types.HelloRequest) (*types.HelloResponse, error) {
    return &types.HelloResponse{
        Out: "Hello World!",
    }, nil
}
```

So a simple function that just returns a fixed message. A couple of key points. * Either a `contract.StaticContext` or `contract.Context` should be the first parameter. It provides various methods that allow you to access the resources on the DAppChain. For example view or modify the the state database, or call other plugins. * The user input in the second parameter and the first return value take the form of [protobuf messages](https://developers.google.com/protocol-buffers/); HelloRequest and HelloResponse in this example. These protobuf message structs need to auto generated from a language neutral .proto file. See below. * The input and output protobuf message parameters need to coordinated with the calling application. As the protobuf message data structures are generated from language independent .proto files it does not matter if the calling application and the smart contract are written in different languages.

So this would be an example of a suitable types.proto file for our Hello function.

```proto
syntax = "proto3";

message HelloRequest {
    string in = 1;
}

message HelloResponse {
    string out = 1;
}

```

A types.pb.go file that we can use can be built using the protoc-gen-gogo plugin for proto, with a command like 

    bash
     protoc --gogo_out=. --plugin=protoc-gen-gogo  types.proto

### スマートコントラクトの呼び出し

The following code fragment shows how to call the Hello function of our Hello World example in Go using functions from Go-loom.

```go
    rpcClient := client.NewDAppChainRPCClient(chainId, "http://localhost:1234", "http://localhost:2345")
    contract := client.NewContract(rpcClient, contractAddr, "HelloWorld")
    request := &types.HelloRequest{}
    response := &types.HelloResponse{}
    _, err = contract.StaticCall("Hello", request, signer, response)
    fmt.Println(response.Out)
```

1. クライアントを作成し、そのURLでDAppチェーンへのアクセスできるようにする。
2. 名前とアドレスからスマートコントラクトのハンドルを取得する。
3. ワイヤータイプのHelloRequestとHelloResponseは、呼び出し中のコントラクトメソッドの入力および出力パラメーターと一致しなくてはならない。 
4. `Hello`メソッドを呼び出す。Helloメソッドは静的コンテキストを持っているので、StaticCallを使っている。

## Solidityコントラクトの呼び出し

Now we have had a quick review of implementing plugins we can look at accessing smart contracts deployed on the DAppChain's EVM from a plugin.

First we we assume we have deployed this simple solidity contract on the DAppChain's EVM.

```solidity
pragma solidity ^0.4.18;
contract SimpleStore {
  function set(uint _value) public {
    value = _value;
  }

  function get() public constant returns (uint) {
    return value;
  }

  uint value;
}
```

We will look at a simple plugin that wraps this solidity contract. So our plugin will have two functions SetValue and GetValue that will just pass data between the SimpleStore contract and the transaction initiator. As it wraps this SimpleStore we will call it EvmExample.

Here is the outline as for the EvmExample contract, with stubs added for the SetValue and GetValue methods.

```go
package main

import (
    "github.com/loomnetwork/go-loom/plugin"
    "github.com/loomnetwork/go-loom/plugin/contractpb"
    "github.com/loomnetwork/go-loom/examples/plugins/evmexample/types"
)

type EvmExample struct {
}

func (c *EvmExample) Meta() (plugin.Meta, error) {
    return plugin.Meta{
        Name:    "EvmExample",
        Version: "1.0.0",
    }, nil
}

func (c *EvmExample) SetValue(ctx contractpb.Context, value *types.WrapValue) error {
    return nil
}

func (c *EvmExample) GetValue(ctx contractpb.Context, req *types.Dummy) (*types.WrapValue, error) {
        return nil, nil
}

var Contract = contractpb.MakePluginContract(&EvmExample{})

func main() {
    plugin.Serve(Contract)
}
```

The .proto file for generating the message declarations looks like

```proto
syntax = "proto3";

message Dummy {
}

message WrapValue {
    int64 value = 1;
}
```

Lets look at the SetValue function first. The function to call to run a smart contract on the EVM is ```go contractpb.CallEVM(ctx Context, addr loom.Address, input []byte, output *[]byte) error

    ここではコンテキストが単に渡されているが、outputを設定するにはダミーとすることが可能だ。 またSolidityコントラクトとインプットを渡すことが必要だ。
    
    このContextはレジストリを含み、コントラクトのアドレスをその名前から取得することを可能にする。
    ```go
    ssAddr, err := ctx.Resolve("SimpleStore")
    

The input is passed straight though to the EVM and needs to be encoded as laid out in the [Solidity ABI documentation](https://solidity.readthedocs.io/en/develop/abi-spec.html).

### パラメーターのABIエンコーディング

So for our input we need to encode it to something like

```text
60fe47b100000000000000000000000000000000000000000000000000000000000003db
```

Don't panic, go-ethereum can help us out.

When you compile Solidity you not only get the bytecode that runs on the EVM, but you get a ABI. The ABI is a json object that describes the contracts interface. Here is the ABI for our SimpleStore

```json
[
    {
      "constant": false,
      "inputs": [
        {
          "name": "_value",
          "type": "uint256"
        }
      ],
      "name": "set",
      "outputs": [],
      "payable": false,
      "stateMutability": "nonpayable",
      "type": "function"
    },
    {
      "constant": true,
      "inputs": [],
      "name": "get",
      "outputs": [
        {
          "name": "",
          "type": "uint256"
        }
      ],
      "payable": false,
      "stateMutability": "view",
      "type": "function"
    }
  ]
```

We can use "github.com/ethereum/go-ethereum/accounts/abi" and this ABI string to encode our input. The key function is [abi.JSON](https://godoc.org/github.com/obscuren/go-ethereum/accounts/abi#JSON)

```go
    abiSimpleStore, err := abi.JSON(strings.NewReader(SimpleStoreABI))
    input, err := abiSimpleStore.Pack("set", big.NewInt(value.Value))
```

Here we have the SimpleContract ABI in the `SimpleStoreABI` variable. We could either read it in from a file, or hard code into the source.

The Pack method takes the function signature and a list of the arguments and returns the encoded input.

### まとめ

Now we know how to get the input, and contact address we can give an example of our SetValue method. Error checking removed for clarity.

```go
func (c *EvmExample) SetValue(ctx contractpb.Context, value *types.WrapValue) error {
    ssAddr, err := ctx.Resolve("SimpleStore")
    abiSS, err := abi.JSON(strings.NewReader(SimpleStoreABI))
    input, err := abiSS.Pack("set", big.NewInt(value.Value))

    evmOut := []byte{}
    err = contractpb.CallEVM(ctx, ssAddr,R input, &evmOut)
    return err
}
```

This function could be called in Go using Go-loom with.

```go
    rpcClient := client.NewDAppChainRPCClient(chainId, writeUri, readUri)
    contract := client.NewContract(rpcClient, contractAddr, "EvmExample")
    payload := &types.WrapValue{
        Value: int64(value),
    }
    _, err = contract.Call("SetValue", payload, signer, nil)

```

The GetValue function now works in a similar fashion. We now have to unwrap the output from the solidity contract and return it in a WrapValue message . `StaticCallEvm` is used as `get` is a view or constant function.

```go
import (
    "github.com/ethereum/go-ethereum/accounts/abi"
    "github.com/ethereum/go-ethereum/common"
    "github.com/loomnetwork/go-loom/examples/plugins/evmexample/types"
    "github.com/loomnetwork/go-loom/plugin"
    "github.com/loomnetwork/go-loom/plugin/contractpb"  
    "math/big"
    "strings"
    "strconv"
)

func (c *EvmExample) GetValue(ctx contractpb.Context, req *types.Dummy) (*types.WrapValue, error) {
    ssAddr, err := ctx.Resolve("SimpleStore")
    if err != nil {
        return nil, err
    }
    abiSS, err := abi.JSON(strings.NewReader(SimpleStoreABI))
    if err != nil {
        return nil, err
    }
    input, err := abiSS.Pack("get")
    if err != nil {
        return nil, err
    }
    evmOut := []byte{}
    err = contractpb.StaticCallEVM(ctx, ssAddr, input, &evmOut)
    if err != nil {
        return nil, err
    }
    value, err := strconv.ParseInt(common.Bytes2Hex(evmOut), 16, 64)
    if err != nil {
        return nil, err
    }
    return &types.WrapValue{
        Value: value,
    }, nil
}
```

## EvmContract

go-loom and loom-js provide help for communicating with a running DAppChain using a RPC client.

### go-loom

This works in much the same way as described for [go-loom Contract](https://loomx.io/developers/docs/en/go-loom-clients.html#connecting-to-a-dappchain)

#### DAppチェーン上のSolidityコントラクトへの接続

So to connect to an existing solidity smart contact running on a DAppChain EVM we can use

```go
package main

import (
  "github.com/loomnetwork/go-loom/auth"
  "github.com/loomnetwork/go-loom/client"
  "github.com/loomnetwork/go-loom/vm"
)

// getContractは新しい`Contract`インスタンスを作成し、これを使ってDAppチェーンのEVM上にあるスマートコントラクトとの対話が行える。
func getEvmContract(contractName string) (*client.EvmContract, error) {
  rpcClient := client.NewDAppChainRPCClient(
    "default",
    "ws://127.0.0.1:46657/websocket",
    "ws://127.0.0.1:9999/queryws",
  )
  contractAddr, err := rpcClient.Resolve(contractName)
  if err != nil {
    return nil, err
  }
  return client.NewEvmContract(rpcClient, contractAddr), nil
}
```

#### DAppチェーンへのSolidityコントラクトのデプロイ

We can also deploy a new smart contract to a running DAppChain EVM. For this we need the contracts bytecode.

A solidity contract can be converted to byte code using the solidity compiler `solc --bin -o . mySolidityProgram.sol`

`hex.DecodeString` can be used to convert a hex string to a []byte array. We can then use the client.DeployContract to deploy our contract. and return an EVMContract handle. The second return parameter is a [transaction hash](https://loomx.io/developers/docs/en/evm.html#transaction-receipt) that can be used to retrive a reciept of the transaction using the TxHash Query.

```go
import (
  "encoding/hex"
  "github.com/loomnetwork/go-loom/auth"
  "github.com/loomnetwork/go-loom/client"
  "github.com/loomnetwork/go-loom/vm"
)

func deployEvmContract(name string, byteHex string, signer auth.Signer) 
(handle *EvmContract, txReciept []byte, err error) {
    // remove the 0x at the beging of a hex string
    byteCode, err := hex.DecodeString(string(byteHex[2:]))
    if err != nil {
        return err
    }
    rpcClient := client.NewDAppChainRPCClient(common.ChainID, common.WriteURI, common.ReadURI)
    return client.DeployContract(rpcClient, signer, byteCode, name)
}
```

#### Retrieving Solidity contract's code

You can retrieve the runtime bytecode for a deployed solidity contract using the DAppChains QueryInterface method GetCode.

```go
// GetCode returns the runtime byte-code of a contract running on a DAppChain's EVM.
// Gives an error for non-EVM contracts.
// contract - address of the contract in the form of a string. (Use loom.Address.String() to convert)
// return []byte - runtime bytecode of the contract.
func (c *DAppChainRPCClient) GetCode(contract string) ([]byte, error) 
```

The runtime code is the inital contract's binary with the code for starting and construting the contract removed as its no longer needed.

#### Writing to a Solidity contract on a DAppChain

Writing and reading to a smart contract deployed on a DAppChain's EVM works in a similar way to [writing](https://loomx.io/developers/docs/en/go-loom-clients.html#writing-data-to-a-dappchain) and [reading](https://loomx.io/developers/docs/en/go-loom-clients.html#reading-data-from-a-dappchain) to non-EVM plugins. The main difference is that the function signature and input parameters need to be converted to bytecode using [ABI encoding](https://solidity.readthedocs.io/en/develop/abi-spec.html). You can use the go-ethereum [abi.JSON](https://godoc.org/github.com/obscuren/go-ethereum/accounts/abi#JSON) function to encode input using your contracts ABI which you can get from `solc --abi -o. MySolidiityProgram.sol`

EvmContract's Call method is used for methods that mutate the DAppChain's state. ```go input ( "github.com/loomnetwork/go-loom/auth" "github.com/loomnetwork/go-loom/client" "github.com/loomnetwork/go-loom/vm "github.com/ethereum/go-ethereum/accounts/abi"  
)

func store(contract *client.EvmContract, key, abi string, value int) ([]byte, error) { abiSS, err := abi.JSON(strings.NewReader(SimpleStoreABI)) if err != nil { return []byte{}, err } input, err := abiSS.Pack("set", big.NewInt(value.Value)) if err != nil { return []byte[], err ] return contract.Call(input, key) } ``` The Call method returns a [transaction hash](https://loomx.io/developers/docs/en/evm.html#transaction-hash) You can use the transaction hash retrieve more information about the contract using the `GetEvmTxReceipt` method. This returns a [transcation recieipt, vm.EvmTxReceipt](https://loomx.io/developers/docs/en/evm.html#transaction-receipt) object.

```go
 input (
   "github.com/loomnetwork/go-loom/auth"
   "github.com/loomnetwork/go-loom/client"
   "github.com/loomnetwork/go-loom/vm
   "github.com/ethereum/go-ethereum/accounts/abi"   
 )

 ...
    txHash, err := store(ecmContract, key, abi, 23)
    if err != nil {
        return err
    }
    rpcClient := client.NewDAppChainRPCClient(common.ChainID, common.WriteURI, common.ReadURI)
    var receipt vm.EvmTxReceipt
    receipt, err = rpcClinet.GetTxReceipt(txHash)
 ...

```

#### Reading from a Solidity contract on a DAppCahin

To get information from an EVM smart contract you need to call a view method using the EvmContract's staticCall. This returns the result in an ABI encoded []byte. As for other EVM methods the function signature and input arguments are [ABI encoded](https://solidity.readthedocs.io/en/develop/abi-spec.html). The caller field in StaticCall is optional, and using an empty loom.Address is fine.

```go
 input (
   "github.com/loomnetwork/go-loom/auth"
   "github.com/loomnetwork/go-loom/client"
   "github.com/loomnetwork/go-loom/vm
   "github.com/ethereum/go-ethereum/accounts/abi"   
 )

 func get(contract *client.EvmContract, abi string, value int) ([]byte, error) {
    abiSS, err := abi.JSON(strings.NewReader(SimpleStoreABI))
    if err != nil {
        return []byte{}, err
    }
    input, err := abiSS.Pack("set", big.NewInt(value.Value))
    if err != nil {
        return []byte[], err
    ]
    return contract.StaticCall(input, loom.RootAddress("MyChainId")) 
 }
 ```

### loom-js

In JavaScript and TypeScript you can Call methods contracts deployed on the EVM 
of a DAppChain in a similar way as for non-EVM plugins, outlined in the 
[loom-js quickstart](https://loomx.io/developers/docs/en/loom-js-quickstart.html#connecting-to-a-dappchain)

#### Connecting to a Solidity contract on a DAppChain

We use the EvmContract class instead of the Contract class. So the loom-js
 quick-start getEvmContract could looks like:
```js
const {
  NonceTxMiddleware, SignedTxMiddleware, Client,
  EvmContract, Address, LocalAddress, CryptoUtils
} = require('loom-js')

const { MapEntry } = require('./helloworld_pb')

/**
 * Creates a new `EvmContract` instance that can be used to interact with a 
 smart contract running on a DAppChain's EVM.
 * @param privateKey Private key that will be used to sign transactions sent to the contract.
 * @param publicKey Public key that corresponds to the private key.
 * @returns `EvmContract` instance.
 */
async function getContract(privateKey, publicKey) {
  const client = new Client(
    'default',
    'ws://127.0.0.1:46657/websocket',
    'ws://127.0.0.1:9999/queryws'
  )
  // required middleware
  client.txMiddleware = [
    new NonceTxMiddleware(publicKey, client),
    new SignedTxMiddleware(privateKey)
  ]
  const contractAddr = await client.getContractAddres('MySolidityContract')
  const callerAddr = new Address(client.chainId, LocalAddress.fromPublicKey(publicKey))
  return new EvmContract({
    contractAddr,
    callerAddr,
    client
  })
}
```

#### Writing to a Solidity contract on a DAppChain

Calling an EVM smart contract's method that mutates the state works the same as [writiing data to a DAppChain](https://loomx.io/developers/docs/en/loom-js-quickstart.html#writing-data-to-a-dappchain) The main difference in the case of an EvmContract is that the input takes the format of an [ABI encoded](https://solidity.readthedocs.io/en/develop/abi-spec.html) array.

```go
    let txHash = await evmContract.callAsync(abiEncodedInput)
```

The return value is a [transaction hash](https://loomx.io/developers/docs/en/evm.html#transaction-hash) You can use the transaction hsh retrive more information about the contract using the `GetEvmTxReceipt` method. This returns a [transaction receipt, EvmTxReceipt](https://loomx.io/developers/docs/en/evm.html#transaction-receipt) object

```text
    let receipt = await client.getTxReceiptAsync(rtv)
```

#### Reading from a Solidity contract on a DAppCahin

To get information from an EVM smart contract you need to call a view method using the EvmContract's staticCall. This returns the result in an ABI encoded []byte. As for other EVM methods the function signature and input arguments are [ABI encoded](https://solidity.readthedocs.io/en/develop/abi-spec.html).

```go
    let txResult = await evmContract.staticCallAsync(abiEncodedInput)
```

## トランザクションのハッシュ値

Writing to a DAppChain using a `Call` transactions that can modify the state returns a transaction hash. This is a unique hash of the transaction details. No two contracts should return the same hash. It can be used to retrieve details of the transaction.

### トランザクションのレシート

Details of each EVM call transaction are stored on the loomchain and can be accessed using the transaction hash.

The loom chain `QueryService` has the method `TxReceipt(txHash []byte) 
([]byte, error)` which returns the receipt in a protobuf form. go-loom and loom-js provide an API for this query.

go-loom:`func (c *DAppChainRPCClient) GetEvmTxReceipt(txHash []byte) (vm
.EvmTxReceipt, error)`

loom-js: `async getTxReceiptAsync(txHash: Uint8Array): Promise<EvmTxReceipt | null>`

Details of the transaction receipt objects follow. 

| Field             | Contents                                       |
| ----------------- |:---------------------------------------------- |
| TransactionIndex  | transaction number this block                  |
| BlockHash         | Hash of the last block                         |
| BlockNumber       | Block height                                   |
| CumulativeGasUsed | Currently not used                             |
| GasUsed           | Currently not used                             |
| ContractAddress   | Address of the contract called                 |
| Logs              | Events, encoded as an array of Event protobufs |
| LogsBloom         | Not used                                       |
| Status            | 1 = success or 0 = failier                     |