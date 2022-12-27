# Quickstart

Code and walkthrough for generating a Deadman's switch for Eth2 PoS validators. 

> Companion article: https://medium.com/@chimera_defi/how-to-setup-a-dead-mans-switch-for-ethereum-validators-2d31ef42ef6b


# 1. Encrypt data for DMS

## Pre-reqs
Get staking deposit cli 
```
git clone https://github.com/ethereum/staking-deposit-cli.git
cd staking-deposit-cli/
pip3 install -r requirements.txt
python3 setup.py install
cd ..
```  

1. Generate validator deposit data and keystores if needed.  
Compress the validator keystores and deposit data that was generated.   
The password you supply to the deposit cli to lock keystores is the `deposit_password`.  The CLI will generate the `mnemonic`. The `validator_password` is used to compress the generated data.  

```
mkdir gen
mkdir encr

cd staking-deposit-cli/
./deposit.sh new-mnemonic --num_validators=5 --mnemonic_language=english --chain=mainnet --folder=../gen
cd ..

zip -P keystorepass -e ./encr/validators.zip -r ./gen/*
```  

2. Collect the mnemonic and supplied validator keystore passwords in the raw secrets file.  Take a look at the provided sample.   

3. Encrypt the secrets file  
```
cp sample.secrets.raw.json secrets.raw.json
export secretsFilePass="privateKeyFilePass"

openssl enc -aes-256-cbc -a -salt -in secrets.raw.json -out ./encr/secrets.enc -pass pass:$secretsFilePass
```

4. Split the passwd into shards
```
# install the tool if needed
# > brew install vitkabele/tap/sss-cli

echo $secretsFilePass | secret-share-split -n 5 -t 2 > ./encr/shares.txt
```
All encrypted files and data you need should now be in `./encr/`  

5. Send the shards in `shares.txt` to each of the Deadman switch nominees. 
6. Queue up a deadmans switch on Gmail or another provider with the encrypted data in `secrets.enc` and `validators.zip` attached.  

# 2. Decrypting the data

## Pre-reqs:
1. You recieved a shard via keybase or other means. e.g.  
```
04a776938fbce611cf90b6a91997652f74a3d3789046b6b251f76e310eb29c56e018c703b07a6dd12650205288141cbc14c045bd6a6820bcd41802020a6812a54c4464fe
```  

2. You get a DMS email with the `secrets.enc` and `validators.zip` files then communicate with other holders to retrieve another password shard from them. 
All holders will get the same email allowing verification.  

3. Confirmed the creator of the DMS is dead or unresponsive. 

## Software pre-reqs:
```
brew install vitkabele/tap/sss-cli
```

## Steps:
1. Combine min shards to retrieve password. In our case 2.  
```
head -n 2 encr/shares.txt | secret-share-combine
```  

2. Use the recovered pass to decrypt the secretsFile  
```
openssl enc -d -aes-256-cbc -a -in encr/secrets.enc -out secrets.raw.out.json
```

3. Use the keystore zip pass in the recovered `secrets.raw.out.json` file to unlock `validators.zip`
```
unzip encr/validators.zip
```

4. Use the validator keystores to call exit on the staked eth. Use the mnemonic to accumulate the ETH for disbursement. etc.. 

