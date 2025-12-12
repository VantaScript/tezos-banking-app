<script>
import { BeaconWallet } from "@taquito/beacon-wallet";
import { NetworkType } from "@airgap/beacon-types";
import { TezosToolkit } from "@taquito/taquito";

const rpcUrl = "https://rpc.shadownet.teztnets.com";
const Tezos = new TezosToolkit(rpcUrl);

let wallet;
let address;
let balance;

const connectWallet = async () => {
  const newWallet = new BeaconWallet({
    name: "Simple dApp tutorial",
    network: {
     type: NetworkType.SHADOWNET,
   },
  });
  await newWallet.requestPermissions();
  address = await newWallet.getPKH();
  getWalletBalance(address);
  wallet = newWallet;
  depositButtonActive = true;
};

const disconnectWallet = () => {
  wallet.client.clearActiveAccount();
  wallet = undefined;
};

const getWalletBalance = async (walletAddress) => {
  const balanceMutez = await Tezos.tz.getBalance(walletAddress);
  balance = balanceMutez.div(1000000).toFormat(2);
};

const contractAddress = "KT1HtZfNKVcgPYCTuVPKm3cjEVAC4CKYrfjX";
let depositAmount = 1;
let depositButtonActive = false;
let depositButtonLabel = "Deposit";

const deposit = async () => {
  depositButtonActive = false;
  depositButtonLabel = "Depositing...";

  Tezos.setWalletProvider(wallet);
  const contract = await Tezos.wallet.at(contractAddress);
  
  const transactionParams = await contract.methodsObject
    .deposit()
    .toTransferParams({
      amount: depositAmount,
    });

  const estimate = await Tezos.estimate.transfer(transactionParams);
  
  const operation = await Tezos.wallet
    .transfer({
      ...transactionParams,
      ...estimate,
    })
    .send();

  console.log(`Waiting for ${operation.opHash} to be confirmed...`);

  await operation.confirmation(2);

  console.log(
    `Operation injected: https://shadownet.tzkt.io/${operation.opHash}`
  );

  await getWalletBalance(address);
  depositButtonActive = true;
  depositButtonLabel = "Deposit";
};

let withdrawButtonActive = true;
let withdrawButtonLabel = "Withdraw";


const withdraw = async () => {
  withdrawButtonActive = false;
  withdrawButtonLabel = "Withdrawing...";

  Tezos.setWalletProvider(wallet);
  const contract = await Tezos.wallet.at(contractAddress);

  const transactionParams = await contract.methodsObject
    .withdraw()
    .toTransferParams();
  const estimate = await Tezos.estimate.transfer(transactionParams);

  const operation = await Tezos.wallet
    .transfer({
      ...transactionParams,
      ...estimate,
    })
    .send();

  console.log(`Waiting for ${operation.opHash} to be confirmed...`);

  await operation.confirmation(2);

  console.log(
    `Operation injected: https://shadownet.tzkt.io/${operation.opHash}`
  );

  await getWalletBalance(address);
  withdrawButtonActive = true;
  withdrawButtonLabel = "Withdraw";
}

</script>

<main>
  <div class="container">
    <div class="header">
      <h1>
        <span class="logo">₿</span>
        Tezos Bank dApp
      </h1>
      <p class="subtitle">Decentralized Banking on Tezos</p>
    </div>

    <div class="card">
      {#if wallet}
        <div class="wallet-info">
          <div class="status-badge connected">
            <span class="status-dot"></span>
            Wallet Connected
          </div>
          
          <div class="info-section">
            <div class="info-item">
              <div class="info-label">Wallet Address</div>
              <div class="address-display">
                <code>{address}</code>
                <button class="copy-btn" on:click={() => navigator.clipboard.writeText(address)} title="Copy address">
                  📋
                </button>
              </div>
            </div>
            
            <div class="info-item">
              <div class="info-label">Balance</div>
              <div class="balance-display">
                <span class="balance-amount">{balance}</span>
                <span class="balance-unit">XTZ</span>
              </div>
            </div>
          </div>

          <div class="faucet-link">
            <p>Need more tez?</p>
            <a href="https://faucet.shadownet.teztnets.com/" target="_blank" class="faucet-btn">
              🚰 Get Testnet Tez
            </a>
          </div>
        </div>

        <div class="actions-section">
          <div class="action-card deposit">
            <h3>💰 Deposit</h3>
            <div class="input-group">
              <div class="amount-controls">
                <input 
                  type="number" 
                  bind:value={depositAmount} 
                  min="1" 
                  max="100" 
                  class="amount-input"
                  placeholder="Amount"
                />
                <span class="input-unit">XTZ</span>
              </div>
              <input 
                type="range" 
                bind:value={depositAmount} 
                min="1" 
                max="100" 
                class="slider"
              />
              <div class="slider-labels">
                <span>1</span>
                <span>100</span>
              </div>
            </div>
            <button 
              on:click={deposit} 
              disabled={!depositButtonActive}
              class="action-btn deposit-btn"
            >
              {depositButtonLabel}
            </button>
          </div>

          <div class="action-card withdraw">
            <h3>💸 Withdraw</h3>
            <p class="action-description">Withdraw all your deposited tez</p>
            <button 
              on:click={withdraw} 
              disabled={!withdrawButtonActive}
              class="action-btn withdraw-btn"
            >
              {withdrawButtonLabel}
            </button>
          </div>
        </div>

        <button on:click={disconnectWallet} class="disconnect-btn">
          Disconnect Wallet
        </button>
      {:else}
        <div class="connect-section">
          <div class="connect-icon">🔐</div>
          <h2>Connect Your Wallet</h2>
          <p>Connect your Tezos wallet to start banking</p>
          <button on:click={connectWallet} class="connect-btn">
            Connect Wallet
          </button>
        </div>
      {/if}
    </div>
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    color: #333;
  }

  main {
    width: 100%;
    padding: 2rem;
    box-sizing: border-box;
  }

  .container {
    max-width: 600px;
    margin: 0 auto;
    animation: fadeIn 0.6s ease-in;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .header {
    text-align: center;
    margin-bottom: 2rem;
    color: white;
  }

  .header h1 {
    font-size: 2.5rem;
    font-weight: 700;
    margin: 0 0 0.5rem 0;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.5rem;
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  }

  .logo {
    font-size: 2.5rem;
    animation: pulse 2s ease-in-out infinite;
  }

  @keyframes pulse {
    0%, 100% {
      transform: scale(1);
    }
    50% {
      transform: scale(1.1);
    }
  }

  .subtitle {
    font-size: 1rem;
    opacity: 0.9;
    margin: 0;
    font-weight: 300;
  }

  .card {
    background: white;
    border-radius: 24px;
    padding: 2.5rem;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    backdrop-filter: blur(10px);
    animation: slideUp 0.6s ease-out;
  }

  @keyframes slideUp {
    from {
      opacity: 0;
      transform: translateY(30px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .status-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    border-radius: 20px;
    font-size: 0.875rem;
    font-weight: 600;
    margin-bottom: 1.5rem;
  }

  .status-badge.connected {
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    color: white;
  }

  .status-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: white;
    animation: blink 2s ease-in-out infinite;
  }

  @keyframes blink {
    0%, 100% {
      opacity: 1;
    }
    50% {
      opacity: 0.5;
    }
  }

  .wallet-info {
    margin-bottom: 2rem;
  }

  .info-section {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    margin-bottom: 1.5rem;
  }

  .info-item {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .info-label {
    font-size: 0.875rem;
    font-weight: 600;
    color: #6b7280;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .address-display {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    background: #f3f4f6;
    padding: 0.75rem 1rem;
    border-radius: 12px;
    font-family: 'Courier New', monospace;
  }

  .address-display code {
    flex: 1;
    font-size: 0.875rem;
    color: #1f2937;
    word-break: break-all;
  }

  .copy-btn {
    background: none;
    border: none;
    cursor: pointer;
    font-size: 1rem;
    padding: 0.25rem;
    transition: transform 0.2s;
  }

  .copy-btn:hover {
    transform: scale(1.1);
  }

  .balance-display {
    display: flex;
    align-items: baseline;
    gap: 0.5rem;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 1rem 1.5rem;
    border-radius: 16px;
    color: white;
  }

  .balance-amount {
    font-size: 2rem;
    font-weight: 700;
  }

  .balance-unit {
    font-size: 1rem;
    opacity: 0.9;
    font-weight: 500;
  }

  .faucet-link {
    text-align: center;
    padding: 1rem;
    background: #fef3c7;
    border-radius: 12px;
    margin-top: 1rem;
  }

  .faucet-link p {
    margin: 0 0 0.5rem 0;
    color: #92400e;
    font-size: 0.875rem;
  }

  .faucet-btn {
    display: inline-block;
    padding: 0.5rem 1rem;
    background: #f59e0b;
    color: white;
    text-decoration: none;
    border-radius: 8px;
    font-weight: 600;
    font-size: 0.875rem;
    transition: all 0.2s;
  }

  .faucet-btn:hover {
    background: #d97706;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(245, 158, 11, 0.4);
  }

  .actions-section {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    margin-bottom: 1.5rem;
  }

  .action-card {
    padding: 1.5rem;
    border-radius: 16px;
    border: 2px solid #e5e7eb;
    transition: all 0.3s;
  }

  .action-card:hover {
    border-color: #667eea;
    box-shadow: 0 4px 20px rgba(102, 126, 234, 0.1);
    transform: translateY(-2px);
  }

  .action-card h3 {
    margin: 0 0 1rem 0;
    font-size: 1.25rem;
    color: #1f2937;
  }

  .action-description {
    margin: 0 0 1rem 0;
    color: #6b7280;
    font-size: 0.875rem;
  }

  .input-group {
    margin-bottom: 1rem;
  }

  .amount-controls {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }

  .amount-input {
    flex: 1;
    padding: 0.75rem 1rem;
    border: 2px solid #e5e7eb;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 600;
    transition: all 0.2s;
  }

  .amount-input:focus {
    outline: none;
    border-color: #667eea;
    box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  }

  .input-unit {
    font-weight: 600;
    color: #6b7280;
    font-size: 0.875rem;
  }

  .slider {
    width: 100%;
    height: 8px;
    border-radius: 4px;
    background: #e5e7eb;
    outline: none;
    -webkit-appearance: none;
    appearance: none;
    margin-bottom: 0.5rem;
  }

  .slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    cursor: pointer;
    transition: all 0.2s;
  }

  .slider::-webkit-slider-thumb:hover {
    transform: scale(1.2);
    box-shadow: 0 0 0 8px rgba(102, 126, 234, 0.1);
  }

  .slider::-moz-range-thumb {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    cursor: pointer;
    border: none;
    transition: all 0.2s;
  }

  .slider::-moz-range-thumb:hover {
    transform: scale(1.2);
  }

  .slider-labels {
    display: flex;
    justify-content: space-between;
    font-size: 0.75rem;
    color: #9ca3af;
  }

  .action-btn {
    width: 100%;
    padding: 1rem 2rem;
    border: none;
    border-radius: 12px;
    font-size: 1rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    position: relative;
    overflow: hidden;
  }

  .action-btn::before {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 0;
    height: 0;
    border-radius: 50%;
    background: rgba(255, 255, 255, 0.3);
    transform: translate(-50%, -50%);
    transition: width 0.6s, height 0.6s;
  }

  .action-btn:hover::before {
    width: 300px;
    height: 300px;
  }

  .action-btn:active {
    transform: scale(0.98);
  }

  .deposit-btn {
    background: linear-gradient(135deg, #10b981 0%, #059669 100%);
    color: white;
    box-shadow: 0 4px 15px rgba(16, 185, 129, 0.3);
  }

  .deposit-btn:hover:not(:disabled) {
    box-shadow: 0 6px 20px rgba(16, 185, 129, 0.4);
    transform: translateY(-2px);
  }

  .withdraw-btn {
    background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
    color: white;
    box-shadow: 0 4px 15px rgba(245, 158, 11, 0.3);
  }

  .withdraw-btn:hover:not(:disabled) {
    box-shadow: 0 6px 20px rgba(245, 158, 11, 0.4);
    transform: translateY(-2px);
  }

  .action-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
  }

  .disconnect-btn {
    width: 100%;
    padding: 0.75rem 1.5rem;
    background: #ef4444;
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 0.875rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s;
  }

  .disconnect-btn:hover {
    background: #dc2626;
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
  }

  .connect-section {
    text-align: center;
    padding: 2rem 0;
  }

  .connect-icon {
    font-size: 4rem;
    margin-bottom: 1rem;
    animation: float 3s ease-in-out infinite;
  }

  @keyframes float {
    0%, 100% {
      transform: translateY(0);
    }
    50% {
      transform: translateY(-10px);
    }
  }

  .connect-section h2 {
    margin: 0 0 0.5rem 0;
    font-size: 1.75rem;
    color: #1f2937;
  }

  .connect-section p {
    margin: 0 0 2rem 0;
    color: #6b7280;
  }

  .connect-btn {
    padding: 1rem 3rem;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    border-radius: 12px;
    font-size: 1.125rem;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .connect-btn:hover {
    transform: translateY(-3px);
    box-shadow: 0 6px 25px rgba(102, 126, 234, 0.4);
  }

  .connect-btn:active {
    transform: translateY(-1px);
  }

  @media (max-width: 640px) {
    main {
      padding: 1rem;
    }

    .card {
      padding: 1.5rem;
    }

    .header h1 {
      font-size: 2rem;
    }

    .balance-amount {
      font-size: 1.5rem;
    }
  }
</style>