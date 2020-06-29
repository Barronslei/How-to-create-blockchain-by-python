# How-to-create-blockchain-by-python

import hashlib
import time
from flask import Flask
from uuid import uuid4

#首先定义区块
class Block:

    def __init__(self, index, proof_no, prev_hash, data, timestamp=None):
        self.index = index
        self.proof_no = proof_no
        self.prev_hash = prev_hash
        self.data = data
        self.timestamp = timestamp or time.time()

#计算区块哈希
    def calculate_hash(self):
        block_of_string = "{}{}{}{}{}".format(self.index, self.proof_no,
                                              self.prev_hash, self.data,
                                              self.timestamp)

        return hashlib.sha256(block_of_string.encode()).hexdigest()

#返回区块内容
    def __repr__(self):
        return "{} - {} - {} - {} - {}".format(self.index, self.proof_no,
                                               self.prev_hash, self.data,
                                               self.timestamp)

#
class BlockChain:

    def __init__(self):
        self.chain = []
        self.current_data = []
        self.nodes = set()
        self.construct_genesis()
#创世区块
    def construct_genesis(self):
        self.construct_block(proof_no=0, prev_hash=0x04436e9146643e8c2c8cca9e071c793aa0c6d31)
#构造新的区块
    def construct_block(self, proof_no, prev_hash):
        block = Block(
            index=len(self.chain),
            proof_no=proof_no,
            prev_hash=prev_hash,
            data=self.current_data,
            timestamp=None or time.time())
        self.current_data = []

        self.chain.append(block)
        return block

#添加新的交易
    def new_data(self, sender, recipient, quantity):
        self.current_data.append({
            'sender': sender,
            'recipient': recipient,
            'quantity': quantity
        })
        return True

#工作量证明
    def proof_of_work(self,last_proof):
        proof_no = 0
        while BlockChain.verifying_proof(proof_no, last_proof) is False:
            proof_no += 1
            print(proof_no)

        return proof_no

    def verifying_proof(last_proof, proof):
        # verifying the proof: does hash(last_proof, proof) contain 4 leading zeroes?

        guess = f'{last_proof}{proof}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"

#返回最新区块
    def latest_block(self):
        return self.chain[-1]




#页面编辑接口
app = Flask(__name__)

# 生成节点ID
node_identifier = str(uuid4()).replace('-', '')

# 实例化
blockchain = BlockChain()

#挖矿
@app.route('/mine', methods=['GET'])

def mine():
    return "We'll mine a new Block"

#添加新交易
@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    return "We'll add a new transaction"

#生成新区块
@app.route('/chain', methods=['GET'])
def full_chain():
    response = {
        'chain': blockchain.chain,
        'length': len(blockchain.chain),
    }
    return jsonify(response), 200
    

if __name__=='__main__':
    app.run(host='0.0.0.0',port=500)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
