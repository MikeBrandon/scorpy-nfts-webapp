<script>
import { onMount } from "svelte";
import { ethers } from "ethers";
import ScorpyNFT from "$lib/utils/scorpyNFT.json";
import { mintedTotalStore } from "$lib/store/store";
import { Jumper } from 'svelte-loading-spinners';

	let currentAccount = "";
    const CONTRACT_ADDRESS = '0xa81224A420951797DE86309c9FB1C988b6C23025';
    // let mintedNumber = 0;
    mintedTotalStore.set(null);
    let update = 0;
    let mintedLink = null;
    let loading = false;

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
            setupEventListener();
            setUpTotalNFTEventListener();
            if (!$mintedTotalStore) {
                askContractForTotalNumberOfNFTs('checkIfWalletIsConnected');
            }

            let chainId = await ethereum.request({ method: 'eth_chainId'});
            console.log("Conneted to chain " + chainId);

            const rinkebyChainId = '0x4';
            if (chainId !== rinkebyChainId) {
                alert("You Are Not Connected to Rinkeby Test Network.");
            }
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

            setupEventListener();
            setUpTotalNFTEventListener();
            if (!$mintedTotalStore) {
                askContractForTotalNumberOfNFTs('connectWallet');
            }
		} catch (e) {
			console.log(e);
		}
	}

    async function setUpTotalNFTEventListener() {
        try {
            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = provider.getSigner();
                const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, ScorpyNFT.abi, signer);

                connectedContract.on('TotalNFTs', (from, TotalNFTs) => {
                    // mintedNumber = TotalNFTs.toNumber();
                    mintedTotalStore.set(TotalNFTs.toNumber());
                    console.log('TotalNFTs: ', $mintedTotalStore);
                    update++;
                })
            } else {
                console.log("Ethereum object doesn't exist!");
            }
        } catch (error) {
            console.log(error);
        }
    }

    async function setupEventListener() {

        try {
            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
                const signer = provider.getSigner();
                const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, ScorpyNFT.abi, signer);

                connectedContract.on('NewNFTMinted', (from, tokenId) => {
                    console.log(from, tokenId.toNumber());
                    alert(`Hey There! We've Minted Your NFT and sent it to your wallet. It may be blank right now. It can take a max of 10 min to show up on OpenSea. Click the button to Check it out.`);
                    mintedLink = `https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`;
                })

                console.log("Event Listener Set Up");
            } else {
                console.log("Ethereum object doesn't exist!");
            }
        } catch (error) {
            console.log(error);
        }
    }

	async function askContractToMintNft() {
		try {
			const { ethereum } = window;

			if (ethereum) {
				const provider = new ethers.providers.Web3Provider(ethereum);
				const signer = provider.getSigner();
				const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, ScorpyNFT.abi, signer);

				console.log("Going to pop wallet now to pay gas...");
				let nftTxn = await connectedContract.makeNFT();

				console.log(`Mining...Please Wait.`);
                loading = true;
				await nftTxn.wait();

				console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);
                loading = false;
			} else {
				console.log('Ethereum Object Doesn\'t Exist');
			}

		} catch (error) {
			console.log(error);
		}
	}

    async function askContractForTotalNumberOfNFTs(calledFrom) {
        try {
            const { ethereum } = window;

            if (ethereum) {
                const provider = new ethers.providers.Web3Provider(ethereum);
				const signer = provider.getSigner();
				const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, ScorpyNFT.abi, signer);

                console.log(`Going to pop wallet now to pay gas... CalledFrom:${calledFrom}`);
                let nftTxn = await connectedContract.getTotalNFTsMintedSoFar();

                console.log("Mining... Please Wait.");
                await nftTxn.wait();

                console.log(`Mined, see Transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`)
            } else {
                console.log("Ethereum object doesn't exist!");
            }

        } catch (error) {
            console.log(error);
        }
    }

	onMount( () => {
		checkIfWalletIsConnected();
	})
</script>

<svelte:head>
	<title>Scorpy NFTs</title>
</svelte:head>

<main>
    <div class="App">
        <div class="container">
          <div class="header-container">
            <p class="header gradient-text">Scorpy Collection</p>
            <p class="sub-text">
              Mint now for a chance to get Mega Rare Nfts.
            </p>
            {#if currentAccount === ""}
                <button on:click={connectWallet} class="cta-button connect-wallet-button">
                    Connect to Wallet
                </button>
			{:else}
				<button on:click={askContractToMintNft} class="cta-button connect-wallet-button" disabled={$mintedTotalStore>=30}>
					Mint NFT
				</button>
            {/if}
            {#if loading}
                <div class = "spinner">
                    <Jumper size="100" color="#35AEE2" unit="px" duration="1s"></Jumper>
                    <p class='footer-text'>Please Wait...</p>
                </div>
            {/if}
            {#if mintedLink}
                <a href={mintedLink}>
                    <button class="cta-button connect-wallet-button">
                        Check Out My NFT
                    </button>
                </a>
            {/if}
          </div>
          <div class="nft-icon">
              <svg version='1.0' xmlns='http://www.w3.org/2000/svg' width='250.000000pt' height='250.000000pt' viewBox='0 0 2350.000000 2350.000000' preserveAspectRatio='xMidYMid meet'> <g transform='translate(0.000000,2350.000000) scale(0.100000,-0.100000)' fill='#ffffff' stroke='none'> <path d='M0 11750 l0 -11750 11750 0 11750 0 0 11750 0 11750 -11750 0 -11750 0 0 -11750z m23420 0 l0 -11670 -3865 0 -3865 0 0 604 0 604 418 -5 c229 -2 485 -8 567 -13 83 -4 152 -6 153 -5 10 9 21 1059 22 1925 0 1165 -17 4880 -26 5498 l-6 442 -1684 0 -1684 0 0 920 0 920 -530 0 -530 0 0 -1185 0 -1185 -140 0 -140 0 0 -310 0 -310 -80 0 -80 0 0 -1190 0 -1190 -160 0 -160 0 0 1190 0 1190 -75 0 -75 0 0 310 0 309 -137 3 -138 3 -5 1185 -5 1185 -370 6 c-203 3 -389 7 -412 8 l-43 1 0 375 0 375 160 0 160 0 0 430 0 430 -215 0 -215 0 0 135 0 135 -125 0 -125 0 0 195 0 195 -500 0 -500 0 0 -285 0 -285 330 0 330 0 0 -125 0 -125 290 0 290 0 0 -345 0 -345 -625 0 -625 0 0 -380 0 -380 565 0 565 0 0 -247 c0 -137 -3 -555 -7 -930 l-6 -683 -1709 0 -1708 0 0 -3936 0 -3937 605 8 605 7 0 -565 c0 -310 -3 -579 -6 -596 l-6 -31 -3909 0 -3909 0 0 11670 0 11670 11670 0 11670 0 0 -11670z'/> <path d='M9780 20545 l0 -205 -230 0 -230 0 -2 -187 -3 -188 -282 -3 -283 -2 0 -305 0 -305 -280 0 -280 0 0 -560 0 -560 -170 0 -170 0 0 -240 0 -239 -137 -3 -138 -3 -3 -1172 -2 -1173 135 0 135 0 0 -250 0 -250 175 0 175 0 0 -425 0 -425 300 0 300 0 0 -190 0 -190 115 0 115 0 0 195 0 195 40 0 40 0 0 230 0 230 390 0 390 0 0 -230 0 -230 520 0 520 0 0 455 0 455 -950 0 -950 0 0 100 0 100 693 2 692 3 3 178 2 177 -695 0 -695 0 0 100 0 100 1700 0 1700 0 0 -345 0 -345 310 0 310 0 0 -585 0 -585 220 0 220 0 2 -502 c2 -277 4 -407 5 -290 l3 212 220 0 220 0 0 190 0 190 185 0 185 0 0 190 0 190 308 2 307 3 3 523 2 522 235 0 235 0 0 150 0 150 120 0 120 0 0 1175 0 1175 -120 0 -120 0 0 240 0 240 -235 0 -235 0 0 560 0 560 -310 0 -310 0 -2 303 -3 302 -167 3 -168 2 0 190 0 190 -275 0 -275 0 0 205 0 205 -1815 0 -1815 0 0 -205z'/> </g> <text x='50%' y='50%' font-size='10em' class='base' dominant-baseline='middle' text-anchor='middle' fill=#60c657>SuperRareScorpy</text></svg>
          </div>
          {#if $mintedTotalStore}
            {#key update}
                <p class="sub-text">
                    TotalNFTs Mined: {$mintedTotalStore ? $mintedTotalStore - 1 : 0}/30
                </p>
            {/key}
          {/if}
            <a href='https://testnets.opensea.io/collection/scorpynft-uvvpnx3gv6'>
                <button class="cta-button connect-wallet-button">
                    🌊 View Collection on OpenSea
                </button>
            </a>
          <div class="footer-container">
            <svg class='twitter-logo' viewBox="0 0 24 24" aria-hidden="true"><g fill=#1DA1F2><path d="M23.643 4.937c-.835.37-1.732.62-2.675.733.962-.576 1.7-1.49 2.048-2.578-.9.534-1.897.922-2.958 1.13-.85-.904-2.06-1.47-3.4-1.47-2.572 0-4.658 2.086-4.658 4.66 0 .364.042.718.12 1.06-3.873-.195-7.304-2.05-9.602-4.868-.4.69-.63 1.49-.63 2.342 0 1.616.823 3.043 2.072 3.878-.764-.025-1.482-.234-2.11-.583v.06c0 2.257 1.605 4.14 3.737 4.568-.392.106-.803.162-1.227.162-.3 0-.593-.028-.877-.082.593 1.85 2.313 3.198 4.352 3.234-1.595 1.25-3.604 1.995-5.786 1.995-.376 0-.747-.022-1.112-.065 2.062 1.323 4.51 2.093 7.14 2.093 8.57 0 13.255-7.098 13.255-13.254 0-.2-.005-.402-.014-.602.91-.658 1.7-1.477 2.323-2.41z"></path></g></svg>
            <a
              class="footer-text"
              href='https://twitter.com/MrScorpioX'
              target="_blank"
              rel="noreferrer"
            >{`built by @MrScorpioXX`}</a>
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

    .spinner {
        display: flex;
        justify-content: center;
        align-items: center;
        padding-top: 1rem;
        padding-bottom: 1rem;
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