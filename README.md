# basebimport time
from collections import deque
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

WINDOW = 20  # number of blocks for average


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Monitoring block gas usage...\n")

    last_block = w3.eth.block_number
    gas_history = deque(maxlen=WINDOW)

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block > last_block:

                for block_num in range(last_block + 1, current_block + 1):

                    block = w3.eth.get_block(block_num)

                    gas_used = block.gasUsed
                    gas_limit = block.gasLimit
                    usage_percent = (gas_used / gas_limit) * 100

                    gas_history.append(usage_percent)

                    avg_usage = sum(gas_history) / len(gas_history)

                    print("Block:", block_num)
                    print("Gas used:", gas_used)
                    print("Gas limit:", gas_limit)
                    print("Usage:", round(usage_percent, 2), "%")
                    print("Avg usage (last blocks):", round(avg_usage, 2), "%")
                    print()

                last_block = current_block

            time.sleep(2)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()
