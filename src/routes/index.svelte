<script>
import { onMount } from "svelte";
import { ethers } from "ethers";
import ScorpyNFT from "$lib/utils/scorpyNFT.json";

	let currentAccount = "";

	async function checkIfWalletIsConnected() {
		const { ethereum } = window;

		if (!ethereum) {
			console.log("Make sure you have metamask!");
			return;
		} else {
			console.log("We have the ethereum object", ethereum);
		}

		const accounts = await ethereum.request({ method: 'eth_accounts' });

		if (accounts.length !== 0) {
			const account = accounts[0];
			console.log("Account Authorized: ", account);
			currentAccount = account;
		} else {
			console.log("No authorized account found");
		}
	}

	async function connectWallet() {
		const { ethereum } = window;

		try {
			if (!ethereum) {
				alert("Get Metamask!");
				return;
			}

			const accounts = await ethereum.request({ method: "eth_requestAccounts" });
			console.log("Connected:  ", accounts[0]);
			currentAccount = accounts[0];
		} catch (e) {
			console.log(e);
		}
	}

	async function askContractToMintNft() {
		const CONTRACT_ADDRESS = '0x5723C7696BD2611B993A4BcAA0Fab5D9e8F7FCB8';

		try {
			const { ethereum } = window;

			if (ethereum) {
				const provider = new ethers.providers.Web3Provider(ethereum);
				const signer = provider.getSigner();
				const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, ScorpyNFT.abi, signer);

				console.log("Going to pop wallet now to pay gas...");
				let nftTxn = await connectedContract.makeNFT();

				console.log(`Mining...Please Wait.`);
				await nftTxn.wait();

				console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);
			} else {
				console.log('Ethereum Object Doesn\'t Exist');
			}

		} catch (error) {
			console.log(error);
		}
	}

	onMount( () => {
		checkIfWalletIsConnected();
	})
</script>

<main>
    <div class="App">
        <div class="container">
          <div class="header-container">
            <p class="header gradient-text">My NFT Collection</p>
            <p class="sub-text">
              Each unique. Each beautiful. Discover your NFT today.
            </p>
            {#if currentAccount === ""}
                <button on:click={connectWallet} class="cta-button connect-wallet-button">
                    Connect to Wallet
                </button>
			{:else}
				<button on:click={askContractToMintNft} class="cta-button connect-wallet-button">
					Mint NFT
				</button>
            {/if}
          </div>
          <div class="footer-container">
            <svg class='twitter-logo' viewBox="0 0 24 24" aria-hidden="true"><g fill=#1DA1F2><path d="M23.643 4.937c-.835.37-1.732.62-2.675.733.962-.576 1.7-1.49 2.048-2.578-.9.534-1.897.922-2.958 1.13-.85-.904-2.06-1.47-3.4-1.47-2.572 0-4.658 2.086-4.658 4.66 0 .364.042.718.12 1.06-3.873-.195-7.304-2.05-9.602-4.868-.4.69-.63 1.49-.63 2.342 0 1.616.823 3.043 2.072 3.878-.764-.025-1.482-.234-2.11-.583v.06c0 2.257 1.605 4.14 3.737 4.568-.392.106-.803.162-1.227.162-.3 0-.593-.028-.877-.082.593 1.85 2.313 3.198 4.352 3.234-1.595 1.25-3.604 1.995-5.786 1.995-.376 0-.747-.022-1.112-.065 2.062 1.323 4.51 2.093 7.14 2.093 8.57 0 13.255-7.098 13.255-13.254 0-.2-.005-.402-.014-.602.91-.658 1.7-1.477 2.323-2.41z"></path></g></svg>
            <a
              class="footer-text"
              href='https://twitter.com/_buildspace'
              target="_blank"
              rel="noreferrer"
            >{`built on @BuildSpace`}</a>
          </div>
        </div>
    </div>
</main>

<style>
    .App {
        height: 100vh;
        background-color: #0d1116;
        overflow: scroll;
        text-align: center;
    }

	code {
	font-family: source-code-pro, Menlo, Monaco, Consolas, 'Courier New',
		monospace;
	}

	main {
		margin: 0;
		padding: 0;
		font-family: -apple-system, Inter, BlinkMacSystemFont, 'Segoe UI', 'Roboto',
		'Oxygen', 'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
		sans-serif;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
	}

    .container {
            height: 100%;
            background-color: #0d1116;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
    }

    .header-container {
        padding-top: 30px;
    }

    .header {
        margin: 0;
        font-size: 50px;
        font-weight: bold;
    }

    .sub-text {
        font-size: 25px;
        color: white;
    }

    .gradient-text {
        background: -webkit-linear-gradient(left, #60c657 30%, #35aee2 60%);
        background-clip: text;
        -webkit-background-clip: text;
        -webkit-text-fill-color: transparent;
    }

    .cta-button {
        height: 45px;
        border: 0;
        width: auto;
        padding-left: 40px;
        padding-right: 40px;
        border-radius: 5px;
        cursor: pointer;
        font-size: 16px;
        font-weight: bold;
        color: white;
    }

    .connect-wallet-button {
        background: -webkit-linear-gradient(left, #60c657, #35aee2);
        background-size: 200% 200%;
        animation: gradient-animation 4s ease infinite;
    }

    .mint-button {
        background: -webkit-linear-gradient(left, #a200d6, #ff6fdf);
        background-size: 200% 200%;
        animation: gradient-animation 4s ease infinite;
        margin-right: 15px;
    }

    .opensea-button {
        background-color: rgb(32, 129, 226);
    }

    .mint-count {
        color: white;
        font-size: 18px;
        font-weight: bold;
    }

    .footer-container {
        display: flex;
        justify-content: center;
        align-items: center;
        padding-bottom: 30px;
    }

    .twitter-logo {
        width: 35px;
        height: 35px;
    }

    .footer-text {
        color: white;
        font-size: 16px;
        font-weight: bold;
    }

    /* KeyFrames */
    @-webkit-keyframes gradient-animation {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }
    @-moz-keyframes gradient-animation {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }
    @keyframes gradient-animation {
        0% {
            background-position: 0% 50%;
        }
        50% {
            background-position: 100% 50%;
        }
        100% {
            background-position: 0% 50%;
        }
    }
</style>