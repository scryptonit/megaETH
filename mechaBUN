from web3 import Web3
from loguru import logger
import asyncio

logger.add("tx_simple.log", format="{time} {level} {message}", level="INFO")

rpc_url = "https://6342.rpc.thirdweb.com/"
chain_id = 6342
to_address = "0x6a811146a81183393533602dd9fb98e2f66a8d10"

tx_data = "0x1249c58b"


def get_eip1559_fees(w3: Web3):
    try:
        fee_history = w3.eth.fee_history(5, "latest", [50])
        base_fee = fee_history["baseFeePerGas"][-1]
        priority_fee = int(Web3.to_wei(1.5, 'gwei'))
        max_fee_per_gas = int(base_fee * 1.5 + priority_fee)

        return {
            "maxFeePerGas": max_fee_per_gas,
            "maxPriorityFeePerGas": priority_fee
        }
    except Exception as e:
        logger.error(f"Fee history fetch failed {e}")

        return {
            "maxFeePerGas": Web3.to_wei(20, 'gwei'),
            "maxPriorityFeePerGas": Web3.to_wei(1.5, 'gwei')
        }


def build_transaction(w3, address):
    nonce = w3.eth.get_transaction_count(address)
    fees = get_eip1559_fees(w3)
    value = Web3.to_wei("0.0042", "ether")
    tx = {
        "chainId": chain_id,
        "from": address,
        "to": Web3.to_checksum_address(to_address),
        "nonce": nonce,
        "value": value,
        "data": tx_data,
        **fees
    }
    gas_limit = w3.eth.estimate_gas(tx)
    tx["gas"] = int(gas_limit * 1.2)

    return tx


async def send_simple_transaction(address, private_key):
    w3 = Web3(Web3.HTTPProvider(rpc_url))

    if not w3.is_connected():
        logger.error("RPC connection failed")
        return
    try:
        address = Web3.to_checksum_address(address)
        tx = build_transaction(w3, address)
        signed_tx = w3.eth.account.sign_transaction(tx, private_key)
        tx_hash = w3.eth.send_raw_transaction(signed_tx.raw_transaction)
        logger.info(f"Transaction sent: {tx_hash.hex()}")
    except Exception as e:
        logger.error(f"Error sending transaction: {e}")


if __name__ == "__main__":
    wallet = ""
    private_key = ""

    asyncio.run(send_simple_transaction(wallet, private_key))
