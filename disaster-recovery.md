# Disaster Recovery

This documentation describes how to recover from critical network failure scenarios.


## Scenario 1: Dealing with compromised keys, or loss of network control

The following section deals with the situation where different keys become compromised, or where the validators collectively lose control of the network, or critical components comprising the application stack.


### Scenario 1.1: Ethereum key compromised

In the event the Ethereum key is compromised, this key needs to be deactivated as soon as possible. As signatures issued with the Ethereum key are not time bound (i.e., a signed message can be used to authenticate a transaction weeks later), a key also should be considered compromised if the attacker had communication access to the HSM, as it is then unclear which and how many messages the attacker has signed (unless the HSM supports sufficient logging to analyse this case).

The Ethereum key can be replaced by following the steps below.

1. Notify the validators that control the `MultisigControl` contract that your Ethereum key has been compromised and needs to be updated.
2. The `remove_signer` function must be executed on the `MultisigControl` contract to remove the compromised key. Each signatory on the `MultisigControl` contract needs to generate their own signature, and a single signature bundle will be submitted to the contract when calling `remove_signer`.
3. After removing the compromised key, the new key for the validator must be added to the `MultisigControl` contract by calling the `add_signer` function. The signature bundle is generated in the same way as for (2).
4. The Vega CLI can be used to generate signatures, please execute `vega bridge erc20 -h` for further instructions.


### Scenario 1.2: Vega hot key compromised

In the event that the Vega hot key is compromised, it can be replaced using the steps below.

1. Terminate the affected node.
2. Use the Vega master key to generate a new hot key (refer to [generating keys](https://github.com/vegaprotocol/networks#generating-vega-keys)).
3. Submit a transaction to the Vega network, signed with your master key, authorising the new hot key.
4. Rejoin the network using a [snapshot file](https://docs.vega.xyz/testnet/node-operators/how-to/use-snapshots#restarting-a-node-using-a-network-snapshot) from network history.


### Scenario 1.3: Vega master key compromised

In the event that the Vega master key is compromised, one can assume that the validator's infrastructure has been rather deeply penetrated. Thus, a compromise of the Vega master key can be seen to imply that all other keys are compromised as well (with the exception of when a HSM has been used; however, as outlined above, even if the key is not known to the attacker it should still be treated as compromised).

1. Since we assume in this case that the validator's ethereum key is also compromised - follow steps from [Scenario 1.1: Ethereum key compromised](https://github.com/vegaprotocol/networks/blob/disaster-recovery-docs/disaster-recovery.md#scenario-11-ethereum-key-compromised)
2. Since we assume in this case the validator's your hot key is also compromised - follow steps 1 - 3 from [Scenario 1.2: Vega hot key compromised](https://github.com/vegaprotocol/networks/blob/disaster-recovery-docs/disaster-recovery.md#scenario-12-vega-hot-key-compromised)
3. Inform the other validators via the appropriate channels to immediately remove the compromised node from the network:
   1. Stop the network
   2. Update the genesis config file to remove the compromised node
   3. Non-compromised validators should restart the network from a [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots)
4. Use newly generated ethereum key to associate ERC20 Vega tokens from the compromised node to the new master key.
5. Inform all token holders that delegated to the compromised node to move their delegation.
6. The compromised validator should start a new node and join the network using a [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restarting-a-node-using-a-network-snapshot) from network history.


### Scenario 1.4: Tendermint key compromised

In the event a tendermint key gets compromised, a third party can double sign messages in that validator's name. This is not highly critical if it happens to only one validator, but that validator should replace its key as soon as reasonable to not allow this situation to get worse through multiple compromises.

**Note**: It is generally hard to detect a key compromise; this is normally discovered only through a general compromise (in which case all keys in working memory should be considered compromised), or by someone else using the key. In the specific case of the Tendermint key, a double signing could also stem from a misconfiguration or an error in switching between validator replicas; thus, a single double signing by a validator should be investigated, but does not require disaster recovery right away.

1. Inform other validators that your tendermint key has been compromised and switch off your node.
2. At network restart, add the new tendermint pubkey in the [genesis.json](https://github.com/vegaprotocol/networks#genesis-config-for-validator) and join others using a [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restarting-a-node-using-a-network-snapshot) from network history.


### Scenario 1.5: Several keys compromised in a short period of time

In the event several keys are compromised/found to be compromised in a short time, the network is under a (successful) attack from a competent and dedicated attacker (APT: Advanced persistent attacker). This means we should assume the worst case scenario, that the network is under attack and the attacker is after all keys.

1. Disconnect all servers you can that had any access to any of the three keys.
2. Start a very thorough forensic audit of *everything* to ensure you’re still safe.


### Scenario 1.6: Validator (and their keys) permanently gone, i.e. bankrupt

In the event a validator disappears, their multisig key is lost (or, worse, sold during the  bankruptcy proceedings when someone else buys the HSM. Yes, this has happened in the past). To make sure this does not escalate into the next scenario, the other validators need to remove that signer reasonably quickly (a matter of one or two days) to reset the thresholds.

1. Call `remove_signer` on the [MultisigControl Contract](https://etherscan.io/address/0x9d0707C91C67d598808834b4881348684e92E11e#writeContract) with the affected key
2. Find a reliable validator to replace this party and agree this validator using appropriate validator governance processes
3. Call `add_signer` to add a new validator on the [MultisigControl Contract](https://etherscan.io/address/0x9d0707C91C67d598808834b4881348684e92E11e#writeContract)


### Scenario 1.7: Loss of control of MultiSig contract on Ethereum

Let `t` be the maximum number of validators we can tolerate to be corrupted (i.e., the largest integer less than a third of the total number of validators).
In the event that `t+1` validators lose their ETH key/go bankrupt/disappear without trace, and the multisig contract is not adapted fast enough, we lost control of the contract. As there’s limited assets on three for the time being, Vega can be reborn with a new instance.

1. Stop the chain immediately
2. Conduct root cause to determine how this happened
3. Restart with a new instance of the contract


## Scenario 2: Incorrect Genesis file configuration

In the event that the genesis configuration is incorrect and this causes a major incident when the network has been started:


### Scenario 2.1: If no network events have taken place before the incorrect config is noticed

1. Stop the network in coordination with all validators.
2. Update the [genesis configuration](https://github.com/vegaprotocol/networks#genesis-config-for-validator)
3. Restart the network from a community agreed [snapshot file](https://docs.vega.xyz/testnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots).


### Scenario 2.2: If the impact of the incorrect configuration CAN be managed/mitigated with the network running and the parameter CAN be changed via governance

1. Create a network parameter governance proposal:  **One** validator to run the command to create proposal >> `vega wallet command --name="testing-mainnet" --pubkey="<my-public-key>" '{<insert json payload of proposal>}'`

2. Coordinate between validators to vote and enact this change:  **All** validators run command to vote on proposal >> `vega wallet command --name="testing-mainnet" --pubkey="<my-public-key>" '{"proposalSubmission": {"reference": "some-ref", "terms": {"closingTimestamp": "1234567890", "enactmentTimestamp": "1234567891", "validationTimestamp": "1234567892", "updateNetworkParameter": { "changes": { "key": "<network-parameter>", "value": "<new-value>" } } } } }'`

**NOTE:** This action will only work if the parameter/config is able to be changed via a governance proposal AND time is not critical in the parameter being updated. If this is time critical it's recommended to jump to go to [Scenario 2.4.](https://github.com/vegaprotocol/networks/blob/disaster-recovery-docs/disaster-recovery.md#scenario-24-if-the-incorrec[…]-part-of-the-snapshot-file-data).

### Scenario 2.3: If the incorrect configuration CANNOT be changed via a governance proposal and the parameter is NOT part of the snapshot file data

1. Stop the network in coordination with all validators.
2. Update the [genesis configuration](https://github.com/vegaprotocol/networks#genesis-config-for-validator)
3. Restart the network from a community agreed [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots).

**NOTE:** This action will only work if the parameter/config is NOT part of the snapshot file data

### Scenario 2.4: If the incorrect configuration CANNOT be changed via governance and the parameter IS part of the snapshot file data

1. Stop the network in coordination with all validators and inform Vega team immediately
2. Vega team to create a software release that would hardcode the value in question into the binary to resolve the issue.
3. Validators would need to ALL deploy the latest code
4. Restart the network using a community agreed [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots).

**NOTE:** Hardcoding values would be implemented at a block height so that it’s only hard coded for the time it’s needed, not in the software until the next deployment.


## Scenario 3: Less than 2/3+1 of the validators are active

### Scenario 3.1: If the network has not been running long

1. Restart the node(s) and see the chain data.
2. Each node will try to replay the chain to the latest block.

**NOTE:** As chain events grow, the data required to catch up will become too large for this to be a viable action.

### Scenario 3.2: If 3.1 is not viable due to age of the network

1. Shut down the full network with coordination between validators.
2. Restart the network using a community agreed [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots).

**NOTE:** If there is a planned outage happening soon the node(s) could be brought back at that time to avoid unplanned outages.

## Scenario 4: Critical bugs

The following section describes how to resolve critical bugs that manifest in various components within the Vega application stack.

### Scenario 4.1: Critical bug in Vega core ABCI application

In the event that a critical bug is found in the protocol software that falls under one of the following conditions that means the network MUST be stopped:
* Severe financial loss
* The network is not operational
* Someone can change balances

If this should occur the following actions should be taken

1. Shut down the full network with coordination between validators.
2. The Vega team will replay the chain to investigate the issue
3. The Vega team will create a new software release that fixes the bug
4. Validators would ALL need to deploy the latest software release (assuming sucessful governance vote to deploy)
5. Restart network using a community agreed [snapshot file](https://docs.vega.xyz/mainnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots).


### Scenario 4.2: Critical big in a smart contract used by the network

In the event that a critical bug is found in tone or more of the Smart Contracts that falls under one of the following conditions::
* Severe financial loss

If this should occur the following actions should be taken:

1. Shut down the full network with coordination between validators.
2. The Vega team will investigate the issue
3. The Vega team will create and deploy a new smart contract release that fixes the bug
4. Validators would ALL need to deploy the new smart contract address (assuming sucessful governance vote to deploy)
5. Restart network using a community agreed [snapshot file](https://docs.vega.xyz/testnet/node-operators/how-to/use-snapshots#restart-a-node-using-local-snapshots).
