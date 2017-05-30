#Blockchain Certified PDF reports
## or securing document validity with blockchain immunibility

### Abstract

### What is a PDF report:
The Portable Document Format (commonly referred to as PDF) is a file format used to present documents in a manner independent of application software, hardware, and operating systems.[2] Each PDF file encapsulates a complete description of a fixed-layout flat document, including the text, fonts, graphics, and other information needed to display it.

### What is the Blockchain
A blockchain is a distributed, append-only database hardened against tampering and revision that provides a single shared source of cryptographically verifiable truth. The use cases outlined here are provided in order to make progress toward possible future standardization and interoperability of blockchains and ledgers. The use cases in this document focus on concrete scenarios that the technology defined by the group should address.

### What is Blockcast
A multi-transaction protocol for storing data in the Bitcoin blockchain.

### Blockcast Litecoin
A multi-transaction protocol for storing data in the Litecoin blockchain.

### Embedding data in the blockchain with OP_RETURN

This tutorial is designed to get a developer started with storing data in the Bitcoin blockchain using the OP_RETURN script opcode. Pre-requisites are either a 21 Bitcoin Computer or 21 plus a full node. The developer should have a basic knowledge of how Bitcoin transactions work; it is recommended to start with the 21 Bitcoin tutorials first. The scope of this tutorial is as an introduction to OP_RETURN and storing data with it.

Introduction
Embedding data in the Bitcoin blockchain is used in many Bitcoin-enabled applications. It allows data to be stored with many of the same benefits that secure bitcoin. Once data is stored in the blockchain, it is very difficult to remove or alter that data. Whoever created the transaction with the data can prove that they created it, because they hold the private key used to sign the transaction. Additionally they can prove the approximate time and date the data became part of the blockchain. This powerful feature can be used to solve very diverse problems.

One notable use of OP_RETURN transactions is proofofexistence.com , a service that allows users to embed the unique hash of any document on the blockchain.

A brief history
In the early days of Bitcoin if you wanted to store data in the blockchain, you had to hack it in. Usually this was accomplished by encoding data and disguising it as regular public addresses. This was seen as an issue because it bloated the UTXO database--the outputs had to be tracked forever because they looked like normal outputs that could be spent in the future. A consensus was reached and the issue addressed in Bitcoin Core 0.9.0, released on March 19th, 2014. From that point forward, Bitcoin Core would relay and mine transactions using the OP_RETURN opcode by default

Introducing OP_RETURN
In Bitcoin every transaction contains input(s) and output(s). Both utilize a Bitcoin language called Script. Diving deep into Script is outside the scope of this tutorial, you can learn more on the Bitcoin Wiki. What OP_RETURN does is allows an output to contain arbitrary data, simultaneously identifying that it is not a spendable output (not bitcoin being transferred for a later redemption). The opcode says "this is an invalid output" but as long as it is a valid transaction, with an appropriate fee, it will be propagated through the network and mined into a block.

Size limitation
Each OP_RETURN output is currently limited to 80 bytes by the rules set forth in Bitcoin Core. Each transaction is limited to one OP_RETURN output. If you are wanting to store plain text in ASCII format that limits you to 80 characters. This may seem small but the intention was to encourage developers to only store a hash of data on the blockchain, with the data itself stored off-chain. This compromise allowed many of the benefits of storing data on the blockchain, without the drawback of bloating it with large-sized data.

Storing a message in the blockchain with OP_RETURN
We are going to use a standard Bitcoin development environment, as well as Pycoin. Pycoin is an open source Python library for working with Bitcoin created by Richard Kiss.

Pre-Work
Check to make sure your bitcoind is running; if it's running this will print an error message (which you can ignore); if it's not running, this will start it. For more information, learn about running a full node on your 21 Bitcoin Computer.

bitcoind --daemon
Note: if you get an error about needing to set an rpcpassword, please follow the full node instructions.

After your bitcoind has been running for several minutes, you can run the following command to see whether you have the most recent blocks:

bitcoin-cli getblockchaininfo
If the number in the verificationprogress field is 0.99999 or higher, you should have the most recent blocks. If not, wait 30 minutes and try running the command again. (If you get an error message, you can try running the command again after five minutes.) Until you have the most recent blocks, you won't be able to complete this tutorial.

If you do not have Pycoin yet, you can install it using pip. Note that you need a version lower than 0.70 to follow this tutorial:

sudo pip3 install --upgrade 'pycoin<0.70'
Private key with test bitcoin
To keep this tutorial short we are going to manually use a bitcoin private key to pay the transaction fee. Managing private keys by hand is risky and a practice that shouldn't be used outside of learning purposes such as this tutorial.

Run the following command (ku is part of pycoin we installed earlier):

ku create -j | grep -i -e "wif\"" -e "btc_address\""
You may receive the following notice:

gpg: directory `/home/twenty/.gnupg' created
gpg: new configuration file `/home/twenty/.gnupg/gpg.conf' created
gpg: WARNING: options in `/home/twenty/.gnupg/gpg.conf' are not yet active
during this run
gpg: keyring `/home/twenty/.gnupg/secring.gpg' created
gpg: keyring `/home/twenty/.gnupg/pubring.gpg' created

Not enough random bytes available.  Please do some other work to give
the OS a chance to collect more entropy! (Need 188 more bytes)
You can just sit around and wait for the required entropy to be generated, but it will take a long time. Instead, it is suggested you use a separate terminal or PuTTY session to open a second connection to your 21 Bitcoin Computer and then run a command that generates a lot of output, such as the yes command shown below. This will send all that output over the network card, which is one of the sources of entropy your computer uses.

yes
Even with yes running, it may take five or more minutes to generate enough entropy on the 21 Bitcoin Computer or a Virtual Private Server (VPS); it should take much less time on a laptop. After you've generated enough entropy and seen the results below, you may simply close the second terminal/PuTTY session.

The output should be something like this:

"btc_address": "1iU4ZHwusMNJ9WVdqGZAYPgCa8RE2S3jWD",
"wif": "Lip8FQzXqQuaqtkJCFgdrG3EwKisqF1otsrVgV4uQ2vfxfvBvW5H",
The second line wif is the private key; WIF stands for Wallet Import Format. Save both the private key and bitcoin address somewhere.

For the purpose of this tutorial you will need enough bitcoin to cover the transaction fee. We are going to use a fee of 10,000 satoshis (0.0001 BTC) for each transaction. You may want to experiment with sending more than one OP_RETURN transaction so a good amount to send would be 50,000 satoshis (0.0005 BTC). Do not send more than you are willing to lose.

Send bitcoin from your wallet to the public address you generated. If you are using 21, you already have bitcoin available and can send it using:

21 send ADDRESS 50000 satoshis
Be sure to replace ADDRESS with the address you created above.

If you don't yet have satoshis available, you can flush buffer satoshis to your on-chain wallet by running the following command:

21 flush
Creating and sending the transaction
Download the following code:

wget https://gist.githubusercontent.com/harding/d34b581d8cfbb8919812/raw/send-op-return.py
Open it in a text editor such as nano, vim, or emacs. For example:

nano send-op-return.py
Replace the ADDRESS and PRIVATE_KEY (near the top) with the ones you created in the section above.

Once you save the file, run it with your message (you may replace the message below with whatever you like):

python3 send-op-return.py 'This is a test'
The output should be something like this:

e15cb87e5030e6241a7c0cd3ac60cc3a1793ab5834a793f73f94db764070edba
If you get the error Not enough satoshis to cover the fee this is probably because the bitcoin you sent has not been mined into a block. You can check for confirmations on a blockchain explorer such as blockchain.info. The related error, missing inputs, means your bitcoind is out of sync; please see the pre-work section for how to check the verificationprogress.

Congratulations, you just sent a Bitcoin transaction that stores your message. It is currently propagating throughout all the nodes in the world and waiting for a miner to include it in a block. What was returned by the script was the Bitcoin transaction hash (also known as the Bitcoin transaction id). Save this so you can look it up in the next section

You can take the transaction id and paste into a blockchain explorer to view its details. Note that some blockchain explorers, such as blockchain.info, show an error until your transaction has been mined into a block. If you want to track the status of the transaction until it is confirmed try Blockchain.info. Once your transaction has a confirmation, you will be able to see your data on Blockchain.info (when viewing a transaction you will need to switch to advanced mode to see your OP_RETURN output at the bottom of the page).

Retrieving your message
Once your message has been stored, the easiest way to retrieve it is with the transaction id, and that is what we will do for this tutorial. For the future, depending on what kind of application you are building, other ways of retrieving messages would be by the address that sent or received them (by iterating over all the transactions and looking for OP_RETURN outputs). You could also build your application to listen to every new Bitcoin transaction and figure out if a message is in it. One service that was built like this is Coin Secrets; you should be able to see your messages on it now.

Download the following code into a file named retrieve-op-return.py:

wget https://gist.githubusercontent.com/harding/983b9aa19ff7cfb8ec80/raw/retrieve-op-return.py
Once you save the file run it with (replace TRANSACTION_ID with one you got back from the send-op-return.py script):

python3 retrieve-op-return.py TRANSACTION_ID
Conclusion
You have now learned the basics of OP_RETURN, sent a message, and retrieved it. If you are excited about the possibilities of using the Bitcoin blockchain as a decentralized data store, here are some projects worth looking into:


# License
Copyright (c) 2017 Sean Westfall 

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
