<template>
  <div>
    <span v-html="msg"></span>
    <p class="greeting">{{ greeting }}</p>
    <button v-on:click="readAccount">读取当前帐号</button>
    <button v-on:click="deployContract">部署合约</button>
    <button v-on:click="readContract">读取合约</button>
  </div>
</template>

<script>
import Web3 from 'web3'
import Contract from 'web3-eth-contract'
import MSHKABIContractABI from '../../static/ABI/MSHK.json' // 引入 Truffle 编译后的合约文件
export default {
  name: 'MSHK.TOP',
  data () {
    return {
      ContractABI: MSHKABIContractABI, // 调用 智能合约的 JSON 接口
      msg: '',
      greeting: 'Hello World! Welcome to mshk.top',
      GanacheUrl: 'localhost:8545',
      CurrentAccount: null,
      NewContract: null
    }
  },
  methods: {
    readAccount () {
      var obj = this
      // 读取帐号列表
      this.web3.eth.getAccounts().then(function (accounts) {
        obj.CurrentAccount = accounts[0]
        obj.msg = '当前读取到的帐号为：' + obj.CurrentAccount // 读取 Ganache 创建的第一个帐号
      })
    },
    readContract () { // 读取合约
      var obj = this
      // 使用 JSON 接口、合约地址，创建一个新的合约实例，其所有方法和事件都在其json 接口对象中定义。
      var contract = new Contract(this.ContractABI.abi, obj.NewContract.options.address)
      // 调用合约中的 hello 方法，并赋值到 this.msg 中，输出到页面
      contract.methods.hello().call().then(s => {
        this.msg = s
      })
    },
    deployContract () { // 部署合约
      var obj = this
      if (obj.CurrentAccount == null) {
        obj.msg = '请先读取帐号'
        return
      }
      var outMsg = []
      var contract = new Contract(this.ContractABI.abi, this.ContractAddr)
      contract.options.data = this.ContractABI.deployedBytecode
      contract.deploy({
        data: obj.ContractABI.bytecode // 合约的字节码
      })
        .send({
          from: obj.CurrentAccount, // 交易的发送地址
          gas: 1500000, // 交易提供的最大 Gas
          gasPrice: '30000000000000' // 用于此交易的以 wei 为单位的 gas 价格
        })
        .on('error', function (error) { // 如果在发送过程中发生错误，则触发。
          obj.msg = 'error:' + error
        })
        .on('transactionHash', function (transactionHash) { // 当交易哈希可用时触发。
          console.log(transactionHash)
        })
        .on('receipt', function (receipt) { // 当交易收据可用时触发。来自合约的收据将没有属性，而是具有事件名称作为键和事件作为属性的属性。
          outMsg.push('Transaction:' + receipt.transactionHash) // 交易哈希
          outMsg.push('Contract created:' + receipt.contractAddress) // 合约创建的地址
          outMsg.push('Gas usage:' + receipt.gasUsed) // 使用的Gas
          outMsg.push('Block number:' + receipt.blockNumber) // 区块
          obj.msg = outMsg.join('<br/ >')
        }).then(function (newContractInstance) {
          obj.NewContract = newContractInstance // instance with the new contract address
        })
    }
  },
  mounted () {
    if (typeof this.web3 !== 'undefined') {
      // web3.currentProvider当前提供者
      this.web3 = new Web3(this.web3.currentProvider)
    } else {
      // set the provider you want from Web3.providers
      this.web3 = new Web3('http://' + this.GanacheUrl)
    }
    // 合约本地 Ganache 的RPC接口
    Contract.setProvider('ws://' + this.GanacheUrl)
  }
}
</script>
<style>
.greeting {
  color: red;
  font-weight: bold;
}
</style>
