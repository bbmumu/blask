How to deploy a contract on Aleo
1. Create a new Aleo wallet
•	If you already have a wallet, you can use it and skip the wallet creation step ✅
•	To create a new wallet, go to the website and click the "Generate" button. Save the output in a safe place 🔒
After saving the keys, use them to add variables to your server using the commands below
2. Request tokens for your wallet
To request tokens, you will need to send an SMS to the number +1-867-888-5688 with your wallet address in the following format:
Send 50 credits to aleo1hcgx79gqerlj4ad2y2w2ysn3pc38nav69vd2r5lc3hjycfy6xcpse0cag0

3. Download required packages and create a tmux session
sudo apt update && \
sudo apt install make clang pkg-config libssl-dev build-essential gcc xz-utils git curl vim tmux ntp jq llvm ufw -y && \
tmux new -s deploy
P.S. Creating a tmux session is required to build a binary, which takes some time. So you won't need to add variables and build a binary again if the ssh connection to your server lost. Just reconnect to the tmux session.
________________________________________
4. Add your wallet and private key as a variable.
echo Enter your Private Key: && read PK && \
echo Enter your View Key: && read VK && \
echo Enter your Address: && read ADDRESS
5. Make sure the data is correct. If not, you can do step 4 again.
echo Private Key: $PK && \
echo View Key: $VK && \
echo Address: $ADDRESS
After receiving a message from the bot about the successful replenishment of your wallet, you need to go to the faucet website and make sure that the transaction has been executed.
Use the Transaction ID as the answer when using the following command.
echo Enter your Transaction ID: && read TI
CIPHERTEXT=$(curl -s https://vm.aleo.org/api/testnet3/transaction/$TI | jq -r '.execution.transitions[0].outputs[0].value')
6. Install required software
cd $HOME
git clone https://github.com/AleoHQ/snarkOS.git --depth 1
cd snarkOS
bash ./build_ubuntu.sh
source $HOME/.bashrc
source $HOME/.cargo/env
cd $HOME
git clone https://github.com/AleoHQ/leo.git
cd leo
cargo install --path .
________________________________________
7. Deploy a contract
NAME=helloworld_"${ADDRESS:4:6}"
mkdir $HOME/leo_deploy
cd $HOME/leo_deploy
leo new $NAME
RECORD=$(snarkos developer decrypt --ciphertext $CIPHERTEXT --view-key $VK)
snarkos developer deploy "$NAME.aleo" \
--private-key "$PK" \
--query "https://vm.aleo.org/api" \
--path "$HOME/leo_deploy/$NAME/build/" \
--broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" \
--fee 4000000 \
--record "$RECORD"
After executing the command, you should see a similar result.
 
Use the received transaction hash to search for your contract on the explore
After your contract is displayed in the explorer, you can proceed to the next step.
8. Execute a contract
Use the transaction hash as the answer for the following command.
echo Enter your Deploy hash: && read DH
CIPHERTEXT=$(curl -s https://vm.aleo.org/api/testnet3/transaction/$DH | jq -r '.fee.transition.outputs[].value')
RECORD=$(snarkos developer decrypt --ciphertext $CIPHERTEXT --view-key $VK)
snarkos developer execute "$NAME.aleo" "hello" "1u32" "2u32" \
--private-key $PK \
--query "https://vm.aleo.org/api" \
--broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" \
--fee 1000000 \
--record "$RECORD"
After execution, you should see the following output
 
Use the received transaction hash to search for your contract execute on the explore
That is it!
________________________________________
8. Useful commands
Add a new tmux session
ctrl+b c
Show all sessions
ctrl+b w
Detach from tmux session
ctrl+b d
Return to a tmux session
tmux attach -t deploy


