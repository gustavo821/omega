# Omega - decentralized predictions protocol

## ⚠️ Warning

Any content produced by Blockworks, or developer resources that Blockworks provides, are for educational and inspiration purposes only. Blockworks does not encourage, induce or sanction the deployment of any such applications in violation of applicable laws or regulations.

## Contribute
Significant contributions to the source code may be compensated with a grant from the Blockworks Foundation.

## To Do
* UI performance - various features take too long to load (i.e. issue set in RedeemView sometimes takes 10 seconds)
* List the YES and NO tokens on serum dex and provide liquidity
* Show historical prices
* Make a button to easily provide liquidity in one step
* Make a button to correct mispricing (e.g. YES price + NO price > 1)
* Show current position and profits
* Move Redeem page contents to Exchange page
* Hook up a USDC on ramp (e.g. Transak)

### setup testing
```
# Get solana tools
VERSION=v1.4.17
sh -c "$(curl -sSfL https://release.solana.com/$VERSION/install)"
git clone https://github.com/solana-labs/solana.git
cd solana
git pull
git checkout $VERSION
cargo install spl-token-cli
sudo apt install -y libssl-dev libudev-dev zlib1g-dev llvm clang

# Run solana local cluster in its own terminal
cd ~/solana
rm -rf config
NDEBUG=1 ./run.sh

# switch terminal, set up testing
CLUSTER=localnet
CLUSTER_URL="http://localhost:8899"
solana config set --url $CLUSTER_URL
solana-keygen new

cd ~/omega/program
cargo build-bpf

OMEGA_PROGRAM_ID="$(solana deploy target/deploy/omega.so | jq .programId -r)"
cd ../cli
KEYPAIR=~/.config/solana/id.json
MY_ADDR="$(solana address)"
QUOTE_MINT="$(spl-token create-token | head -n 1 | cut -d' ' -f3)"
USER_QUOTE_WALLET="$(spl-token create-account $QUOTE_MINT | head -n 1 | cut -d' ' -f3)"
spl-token mint $QUOTE_MINT 100 $USER_QUOTE_WALLET
CONTRACT_NAME=TRUMPFEB
OUTCOME_NAMES="YES NO"
DETAILS="Resolution: Donald Trump is the President of the United States at 2021-02-01 00:00:00 UTC. Each YES token will be redeemable for 1 USDC if the resolution is true and 0 otherwise. Similarly, each NO token will be redeemable for 1 USDC if the resolution is false. The oracle will resolve this contract before 2021-02-08 00:00:00 UTC in the same way as the TRUMPFEB token at ftx.com."
CONTRACT_KEYS_PATH="../ui/src/contract_keys.json"
cargo run -- $CLUSTER init-omega-contract --payer $KEYPAIR --omega-program-id $OMEGA_PROGRAM_ID --oracle $MY_ADDR \
    --quote-mint $QUOTE_MINT --num-outcomes 2 --outcome-names $OUTCOME_NAMES --contract-name $CONTRACT_NAME \
    --details "$DETAILS" --exp-time "2021-02-01 00:00:00" --contract-keys-path $CONTRACT_KEYS_PATH

```

### use sollet mnemonic
```
MNEMONIC="word0 word1 word2"
PASSPHRASE="pass"
cargo run sollet-to-local --keypair-path ~/.config/solana/id.json --sollet-mnemonic $MNEMONIC --passphrase $PASSPHRASE
```

### resolve
```
WINNER=NO
cargo run resolve --oracle-keypair $KEYPAIR --payer $KEYPAIR --winner $WINNER --contract-keys-path $CONTRACT_KEYS_PATH
```