---
cover: ../.gitbook/assets/CABAL (1).png
coverY: 0
---

# Create a simple transaction

We can use `cardano-cli` to create transactions.

```bash
cardano-cli transaction
```

### Create a transaction with `build-raw`

we will need the protocol parameters, so that we can later calculate the transaction fee

{% code overflow="wrap" %}
```bash
cardano-cli query protocol-parameters --testnet-magic 2 --out-file pparams.json
```
{% endcode %}

We need to know the transaction hash and index we will send funds from

```bash
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2
```

&#x20;Let's send 5,000 ada to our `paymentwithstake.addr`

```bash
cardano-cli transaction build-raw --babbage-era \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--tx-out $(cat paymentwithstake.addr)+5000000000 \
--tx-out $(cat payment.addr)+5000000000 \
--fee 0 \
--protocol-params-file pparams.json \
--out-file tx.draft
```

```bash
cardano-cli transaction calculate-min-fee --tx-body-file tx.draft \
--testnet-magic 2 \
--protocol-params-file pparams.json \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 
```

This will tell us the transaction fee that we need to pay, for example

```
171353 Lovelace
```

Now we need to rebuild the transaction body adding the fees and recalculating the change that we will send to `payment.addr`&#x20;

```
 echo $((10000000000 - 5000000000 - 171353))
 code
 > 4999828647
```

{% code overflow="wrap" %}
```
cardano-cli transaction build-raw --babbage-era \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--tx-out $(cat paymentwithstake.addr)+5000000000 \
--tx-out $(cat payment.addr)+4999828647 \
--fee 171353 \
--protocol-params-file pparams.json \
--out-file tx.raw
```
{% endcode %}

Now we just need to sign and submit the transaction. Of course, we sign it with the `payment.skey` that we generated in the first place.&#x20;

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 2 \
--out-file tx.signed
```

```bash
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed 
```

### Create a transaction with `build`

Now, let's send the rest of the funds in `payment.addr` to `paymentwithstake.addr` This time we will use the build command which will automatically take care of the fees, simplifying the process

{% code overflow="wrap" %}
```
cardano-cli transaction build --babbage-era \
--testnet-magic 2 \
--tx-in $(cardano-cli query utxo --address $(cat payment.addr) --testnet-magic 2 --out-file  /dev/stdout | jq -r 'keys[0]') \
--change-address $(cat paymentwithstake.addr) \
--out-file tx.raw
```
{% endcode %}

Sign and submit as before

```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--testnet-magic 2 \
--out-file tx.signed
```

```
cardano-cli transaction submit \
--testnet-magic 2 \
--tx-file tx.signed 
```

