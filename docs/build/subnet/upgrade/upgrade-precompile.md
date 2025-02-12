---
tags: [Build, Subnets]
description: You can customize Subnet-EVM based Subnets after deployment by enabling and disabling precompiles.
sidebar_label: Subnet Precompile Config
pagination_label: Upgrade Your Subnet-EVM Precompile Configuration 
sidebar_position: 3
---

# How to Upgrade Your Subnet-EVM Precompile Configuration 

## Upgrading a Subnet

You can customize Subnet-EVM based Subnets after deployment by enabling and disabling precompiles.
To do this, create a `upgrade.json` file and place it in the appropriate directory. 

This document describes how to perform such network upgrades. It's specific for Subnet-EVM upgrades.


The document [Upgrade a Subnet](/build/subnet/upgrade/considerations-subnet-upgrade.md) 
describes all the background information required regarding Subnet upgrades.

:::warning

It's very important that you have read and understood the previously linked document. 
Failing to do so can potentially grind your network to a halt.


:::

This tutorial assumes that you have already 
[installed](/tooling/cli-guides/install-avalanche-cli.md) Avalanche-CLI.
It assumes you have already created and deployed a Subnet called `testSubnet`. 


## Generate the Upgrade File

The 
[Precompiles](/build/subnet/upgrade/customize-a-subnet.md#network-upgrades-enabledisable-precompiles)
documentation describes what files the network upgrade requires, and where to place them.

To generate a valid `upgrade.json` file, run:

```shell
avalanche subnet upgrade generate testSubnet
```

If you didn't create `testSubnet` yet, you would see this result:

```shell
avalanche subnet upgrade generate testSubnet
The provided subnet name "testSubnet" does not exist
```

Again, it makes no sense to try the upgrade command if the Subnet doesn't exist. 
If that's the case, please go ahead and 
[create](/build/subnet/hello-subnet.md) the Subnet first.

If the Subnet definition exists, the tool launches a wizard.
It may feel a bit redundant, but you first see some warnings, to draw focus to the dangers involved:

```shell
avalanche subnet upgrade generate testSubnet       
Performing a network upgrade requires coordinating the upgrade network-wide.
A network upgrade changes the rule set used to process and verify blocks, such that any node that 
upgrades incorrectly or fails to upgrade by the time that upgrade goes into effect may become 
out of sync with the rest of the network.

Any mistakes in configuring network upgrades or coordinating them on validators may cause the 
network to halt and recovering may be difficult.
Please consult 
https://docs.avax.network/subnets/customize-a-subnet#network-upgrades-enabledisable-precompiles 
for more information
Use the arrow keys to navigate: ↓ ↑ → ← 
? Press [Enter] to continue, or abort by choosing 'no': 
  ▸ Yes
    No
```

Go ahead and select `Yes` if you understand everything and you agree.

You see a last note, before the actual configuration wizard starts:

```shell
Avalanchego and this tool support configuring multiple precompiles. 
However, we suggest to only configure one per upgrade.

Use the arrow keys to navigate: ↓ ↑ → ← 
? Select the precompile to configure: 
  ▸ Contract Deployment Allow List
    Manage Fee Settings
    Native Minting
    Transaction Allow List
```

Refer to [Precompiles](/build/subnet/upgrade/customize-a-subnet.md#precompiles) 
for a description of available precompiles and how to configure them. 

Make sure you understand precompiles thoroughly and how to configure them before 
attempting to continue.

For every precompile in the list, the wizard guides you to provide correct information 
by prompting relevant questions.
For the sake of this tutorial, select `Transaction Allow List`. 
The document 
[Restricting Who Can Submit 
Transactions](/build/subnet/upgrade/customize-a-subnet.md#restricting-who-can-submit-transactions) 
describes what this precompile is about.

```shell
✔ Transaction Allow List 
Set parameters for the "Manage Fee Settings" precompile
Use the arrow keys to navigate: ↓ ↑ → ← 
? When should the precompile be activated?: 
  ▸ In 5 minutes
    In 1 day
    In 1 week
    In 2 weeks
    Custom
```

This is actually common to all precompiles: they require an 
activation timestamp.
If you think about it, it makes sense: you want a synchronized activation of your precompile.
So think for a moment about when you want to set the activation timestamp to.
You can select one of the suggested times in the future, or you can pick a custom one.
After picking `Custom`, it shows the following prompt:

```shell
✔ Custom
✗ Enter the block activation UTC datetime in 'YYYY-MM-DD HH:MM:SS' format: 
```

The format is `YYYY-MM-DD HH:MM:SS`, therefore `2023-03-31 14:00:00` would be a valid timestamp.
Notice that the timestamp is in UTC. 
Please make sure you have converted the time from your timezone to UTC.
Also notice the `✗` at the beginning of the line. The CLI tool does input validation, so if you 
provide a valid timestamp, the `x` disappears:

```shell
✔ Enter the block activation UTC datetime in 'YYYY-MM-DD HH:MM:SS' format: 2023-03-31 14:00:00
```

The timestamp must be in the **future**, so make sure you use such a 
timestamp should you be running this tutorial after `2023-03-31 14:00:00` ;-) .

After you provided the valid timestamp, proceed with the precompile specific configurations:

```shell
The chosen block activation time is 2023-03-31 14:00:00
Use the arrow keys to navigate: ↓ ↑ → ← 
? Add 'adminAddresses'?: 
  ▸ Yes
    No
```

This will enable the addresses added in this section to add other admins and/or 
add enabled addresses for transaction issuance.
The addresses provided in this tutorial are fake.

:::caution

However, make sure you or someone you trust have full control over the addresses.
Otherwise, you might bring your Subnet to a halt.

:::

```shell
✔ Yes
Use the arrow keys to navigate: ↓ ↑ → ← 
? Provide 'adminAddresses': 
  ▸ Add
    Delete
    Preview
    More Info
↓   Done
```

The prompting runs with a pattern used throughout the tool: 

* Select an operation:
  * `Add`: adds a new address to the current list
  * `Delete`: removes an address from the current list
  * `Preview`: prints the current list 
* `More info` prints additional information for better guidance, if available
* Select `Done` when you completed the list

Go ahead and add your first address:

```shell
✔ Add
✔ Add an address: 0xaaaabbbbccccddddeeeeffff1111222233334444
```

Add another one:

```shell
✔ Add
Add an address: 0xaaaabbbbccccddddeeeeffff1111222233334444
✔ Add
✔ Add an address: 0x1111222233334444aaaabbbbccccddddeeeeffff
```

Select `Preview` this time to confirm the list is correct:

```shell
✔ Preview
0. 0xaaaAbbBBCccCDDddEeEEFFfF1111222233334444
1. 0x1111222233334444aAaAbbBBCCCCDdDDeEeEffff
Use the arrow keys to navigate: ↓ ↑ → ← 
? Provide 'adminAddresses': 
  ▸ Add
    Delete
    Preview
    More Info
↓   Done
```

If it looks good, select `Done` to continue:

```shell
✔ Done
Use the arrow keys to navigate: ↓ ↑ → ← 
? Add 'enabledAddresses'?: 
  ▸ Yes
    No
```

Add one such enabled address, these are addresses which can issue transactions:

```shell
✔ Add
✔ Add an address: 0x55554444333322221111eeeeaaaabbbbccccdddd█
```

After you added this address, and selected `Done`, the tool asks if you want to 
add another precompile:

```shell
✔ Done
Use the arrow keys to navigate: ↓ ↑ → ← 
? Should we configure another precompile?: 
  ▸ No
    Yes
```

If you needed to add another one, you would select `Yes` here. The wizard would guide you
through the other available precompiles, excluding already configured ones.

To avoid making this tutorial too long, the assumption is you're done here.
Select `No`, which ends the wizard.

This means you have successfully terminated the generation of the upgrade file,
often called upgrade bytes. The tool stores them internally.

:::warning

You shouldn't move files around manually.
Use the `export` and `import` commands to get access to the files.

:::

So at this point you can either:

* Deploy your upgrade bytes locally
* Export your upgrade bytes to a file, for installation on a validator running on another machine
* Import a file into a different machine running Avalanche-CLI 


## How To Upgrade a Local Network

The normal use case for this operation is that:

* You already created a Subnet
* You already deployed the Subnet locally
* You already generated the upgrade file with the preceding command or imported into the tool
* This tool already started the network

If the preceding requirements aren't met, the network upgrade command fails.

Therefore, to apply your generated or imported upgrade configuration:

```shell
avalanche subnet upgrade apply testSubnet
```

A number of checks run. For example, if you created the Subnet but didn't deploy it locally:

```shell
avalanche subnet upgrade apply testSubnet 
Error: no deployment target available
Usage:
  avalanche subnet upgrade apply [subnetName] [flags]

Flags:
      --avalanchego-chain-config-dir string   avalanchego's chain config file directory (default "/home/fabio/.avalanchego/chains")
      --config                                create upgrade config for future subnet deployments (same as generate)
      --fuji fuji                             apply upgrade existing fuji deployment (alias for `testnet`)
  -h, --help                                  help for apply
      --local local                           apply upgrade existing local deployment
      --mainnet mainnet                       apply upgrade existing mainnet deployment
      --print                                 if true, print the manual config without prompting (for public networks only)
      --testnet testnet                       apply upgrade existing testnet deployment (alias for `fuji`)

Global Flags:
      --log-level string   log level for the application (default "ERROR")
```

Go ahead and [deploy](/build/subnet/deploy/local-subnet.md) 
first your Subnet if that's your case.

If you already had deployed the Subnet instead, you see something like this:

```shell
avalanche subnet upgrade apply testSubnet     
Use the arrow keys to navigate: ↓ ↑ → ← 
? What deployment would you like to upgrade: 
  ▸ Existing local deployment
```

Select `Existing local deployment`. This installs the upgrade file on all nodes of your local 
network running in the background.

Et voilà. This is the output shown if all went well:

```shell
✔ Existing local deployment
.......
Network restarted and ready to use. Upgrade bytes have been applied to running nodes at these endpoints.
The next upgrade will go into effect 2023-03-31 09:00:00
+-------+------------+-----------------------------------------------------------------------------------+
| NODE  |     VM     |                                        URL                                        |
+-------+------------+-----------------------------------------------------------------------------------+
| node1 | testSubnet | http://0.0.0.0:9650/ext/bc/2YTRV2roEhgvwJz7D7vr33hUZscpaZgcYgUTjeMK9KH99NFnsH/rpc |
+-------+------------+-----------------------------------------------------------------------------------+
| node2 | testSubnet | http://0.0.0.0:9652/ext/bc/2YTRV2roEhgvwJz7D7vr33hUZscpaZgcYgUTjeMK9KH99NFnsH/rpc |
+-------+------------+-----------------------------------------------------------------------------------+
| node3 | testSubnet | http://0.0.0.0:9654/ext/bc/2YTRV2roEhgvwJz7D7vr33hUZscpaZgcYgUTjeMK9KH99NFnsH/rpc |
+-------+------------+-----------------------------------------------------------------------------------+
| node4 | testSubnet | http://0.0.0.0:9656/ext/bc/2YTRV2roEhgvwJz7D7vr33hUZscpaZgcYgUTjeMK9KH99NFnsH/rpc |
+-------+------------+-----------------------------------------------------------------------------------+
| node5 | testSubnet | http://0.0.0.0:9658/ext/bc/2YTRV2roEhgvwJz7D7vr33hUZscpaZgcYgUTjeMK9KH99NFnsH/rpc |
+-------+------------+-----------------------------------------------------------------------------------+
```

There is only so much the tool can do here for you. 
It installed the upgrade bytes *as-is* as you configured respectively provided them to the tool.
You should verify yourself that the upgrades were actually installed correctly, 
for example issuing some transactions - mind the timestamp!.


## Apply the Upgrade to a Public Node (Fuji or Mainnet)

For this scenario to work, you should also have deployed the Subnet 
to the public network (Fuji or Mainnet) with this tool.
Otherwise, the tool won't know the details of the Subnet, and won't be able to guide you.

Assuming the Subnet has been already deployed to Fuji, when running the `apply` command, 
the tool notices the deployment:

```shell
avalanche subnet upgrade apply testSubnet  
Use the arrow keys to navigate: ↓ ↑ → ← 
? What deployment would you like to upgrade: 
    Existing local deployment
  ▸ Fuji
```

If not, you would not find the `Fuji` entry here.

:::important

This scenario assumes that you are running the `fuji` validator on the same machine
which is running Avalanche-CLI.

:::

If this is the case, the tool tries to install the upgrade file at the expected destination.
If you use default paths, it tries to install at `$HOME/.avalanchego/chains/`, creating the 
chain id directory, 
so that the file finally ends up at `$HOME/.avalanchego/chains/<chain-id>/upgrade.json`.

If you are *not* using default paths, you can configure the path by providing the 
flag `--avalanchego-chain-config-dir` to the tool.
For example:

```shell
avalanche subnet upgrade apply testSubnet --avalanchego-chain-config-dir /path/to/your/chains
```

Make sure to identify correctly where your chain config dir is, or the node might fail to find it.

If all is correct, the file gets installed:

```shell
avalanche subnet upgrade apply testSubnet
✔ Fuji
The chain config dir avalanchego uses is set at /home/fabio/.avalanchego/chains
Trying to install the upgrade files at the provided /home/fabio/.avalanchego/chains path
Successfully installed upgrade file
```


If however the node is *not* running on this same machine where you are executing Avalanche-CLI, 
there is no point in running this command for a Fuji node.
In this case, you might rather export the file and install it at the right location.

To see the instructions about how to go about this, add the `--print` flag:

```shell
avalanche subnet upgrade apply testSubnet --print 
✔ Fuji
To install the upgrade file on your validator:

1. Identify where your validator has the avalanchego chain config dir configured.
   The default is at $HOME/.avalanchego/chains (/home/user/.avalanchego/chains on this machine).
   If you are using a different chain config dir for your node, use that one.
2. Create a directory with the blockchainID in the configured chain-config-dir (e.g. $HOME/.avalanchego/chains/ExDKhjXqiVg7s35p8YJ56CJpcw6nJgcGCCE7DbQ4oBknZ1qXi) if doesn't already exist.
3. Create an `upgrade.json` file in the blockchain directory with the content of your upgrade file.
   This is the content of your upgrade file as configured in this tool:
{
    "precompileUpgrades": [
        {
            "txAllowListConfig": {
                "adminAddresses": [
                    "0xb3d82b1367d362de99ab59a658165aff520cbd4d"
                ],
                "enabledAddresses": null,
                "blockTimestamp": 1677550447
            }
        }
    ]
}

   ******************************************************************************************************************
   * Upgrades are tricky. The syntactic correctness of the upgrade file is important.                               *
   * The sequence of upgrades must be strictly observed.                                                            *
   * Make sure you understand https://docs.avax.network/nodes/configure/chain-config-flags.md#subnet-chain-configs  *
   * before applying upgrades manually.                                                                             *
   ******************************************************************************************************************
```

The instructions also show the content of your current upgrade file, so you can just 
select that if you wish.
Or actually export the file.

## Export the Upgrade File

If you have generated the upgrade file, you can export it:

```shell
avalanche subnet upgrade export testSubnet
✔ Provide a path where we should export the file to: /tmp/testSubnet-upgrade.json
```

Just provide a valid path to the prompt, and the tool exports the file there. 

```shell
avalanche subnet upgrade export testSubnet 
Provide a path where we should export the file to: /tmp/testSubnet-upgrade.json
Writing the upgrade bytes file to "/tmp/testSubnet-upgrade.json"...
File written successfully.
```

You can now take that file and copy it to validator nodes, see preceding instructions.

## Import the Upgrade File

You or someone else might have generated the file elsewhere, or on another machine.
And now you want to install it on the validator machine, using Avalanche-CLI.

You can import the file:

```shell
avalanche subnet upgrade import testSubnet
Provide the path to the upgrade file to import: /tmp/testSubnet-upgrade.json
```

An existing file with the same path and filename would be overwritten.

After you have imported the file, you can `apply` it either to a local network or to 
a locally running validator. Follow the instructions for the appropriate use case.

