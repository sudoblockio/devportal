# Fees

When you're making a cross-chain call using xCall, there's a fee that covers the costs of sending the message between different blockchains. `sendCallMessage` is payable, so the fee should be included in the transaction. You can get the fee by calling `getFee`.

`getFee(String _net, boolean _rollback)` returns the total fee (relay + protocol) for sending a message to the specified network. If a rollback data is provided, the `_rollback` param should be set as true because there is an additional backward direction fee (from destination back to source) if rollback is used.

\
This fee is split into two parts:

**Relay Fee**: This fee is for the service that actually sends your message from one blockchain to another.&#x20;

**Protocol Fee**: This is the fee for using the cross-chain communication protocol underlying xCall.

### Relay Fee

* If a relay successfully delivers a cross-chain message, it can claim a reward. The first relay to successfully deliver a cross-chain message gets the reward.
* The reward claim can be made using the `claimReward` function, which requires the network address of the reward (`_network`) and the address of the receiver (`_receiver`).
* The accrued reward for each relay can be checked using the `getReward` function.

### Protocol Fee

* The protocol fee is set on the `xCall` contract by an admin wallet.

### Burning Fees and Impact to ICX Tokenomics

Native fees generated by protocol fees and relays managed by the ICON Foundation are swapped to native ICX and burned.

### Example

The process of fetching the cross chain transaction fee is done by making a readonly call to the `getFee(String _net, Boolean _rollback)` method of the xCall contract in the origin chain.

The following code sample showcase how to get the fee of a crosschain message originating on the Berlin testnet on ICON and the destination chain being the Sepolia testnet on Ethereum.

```javascript
const IconService = require("icon-sdk-js");

const { IconBuilder, HttpProvider } = IconService.default;

const { CallBuilder } = IconBuilder;
const ICON_RPC_URL = "https://berlin.net.solidwallet.io/api/v3/icon_dex";
const XCALL_PRIMARY = "cxf4958b242a264fc11d7d8d95f79035e35b21c1bb";
const NETWORK_LABEL_SECONDARY = "0xaa36a7.eth2";

const HTTP_PROVIDER = new HttpProvider(ICON_RPC_URL);
const ICON_SERVICE = new IconService.default(HTTP_PROVIDER);

/*
 * getFee - calls the getFee method of the xcall contract
 * @param {boolean} useRollback - whether to use rollback
 * @returns {String} - the fee as a hex value
 * @throws {Error} - if there is an error getting the fee
 */
async function getFee(useRollback = false) {
  try {
    const params = {
      _net: NETWORK_LABEL_SECONDARY,
      _rollback: useRollback ? "0x1" : "0x0"
    };

    const txObj = new CallBuilder()
      .to(XCALL_PRIMARY)
      .method("getFee")
      .params(params)
      .build();

    return await ICON_SERVICE.call(txObj).execute();
  } catch (e) {
    console.log("error getting fee", e);
    throw new Error("Error getting fee");
  }
}

async function main() {
  const fee = await getFee();
  console.log("fee", fee);
}

main();
```
