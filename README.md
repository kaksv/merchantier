# AI-Powered Prediction Market

An innovative prediction market platform built with Chainlink CRE (Confidential Reporting Engine) that leverages AI to enable decentralized market settlement. This project combines smart contracts, event-driven workflows, and AI-powered oracles to create autonomous prediction markets.

## 🎯 Overview

This prediction market platform allows users to:
- **Create prediction markets** for any binary question (Yes/No)
- **Make predictions** by staking ETH on their beliefs
- **Earn rewards** if their prediction is correct
- **Automate settlement** using AI and Chainlink CRE for trustless market resolution

The platform uses Chainlink's CRE infrastructure to trigger AI analysis when markets request settlement, ensuring objective and decentralized decision-making without relying on centralized authorities.

The platform is built and stress tested both on the tenderly virtual testnet and Ethereum sepolia test network.

## 🏗️ Architecture

### Smart Contracts (`/contracts/src`)

**PredictionMarket.sol**
- Core contract managing market lifecycle
- Handles market creation, predictions, and settlement
- Stores market data and user predictions
- Implements secure fund management and payout distribution

**Key Features:**
- Binary prediction markets (Yes/No outcomes)
- Pool-based reward distribution
- Confidence-weighted settlement
- Secure fund transfers with validation

### Workflow Engine (`/my-workflow`)

The workflow automates market settlement through two main triggers:

**HTTP Trigger (Market Creation)**
- Receives market creation requests
- Triggers AI analysis via Google's Gemini LLM
- Processes settlement recommendations

**Log Trigger (Event-Driven Settlement)**
- Monitors `SettlementRequested` events on-chain
- Automatically triggers AI analysis when markets request settlement
- Ensures timely and reliable market resolution

### API Integration

- **Google Gemini LLM**: Analyzes market questions and generates settlement recommendations
- **Chainlink CRE SDK**: Manages decentralized workflow execution
- **EVM Compatible**: Works with Ethereum Sepolia testnet

## 📋 Market Lifecycle

```
1. Create Market
   └─> Define binary question
       └─> Market active and accepting predictions

2. Make Predictions
   └─> Users stake ETH on Yes/No outcomes
       └─> Pools accumulate

3. Request Settlement
   └─> Market creator triggers settlement
       └─> AI analysis begins

4. Settlement via AI
   └─> Gemini LLM analyzes question
       └─> Generates outcome prediction
           └─> Smart contract receives result

5. Payout Distribution
   └─> Winning predictors claim rewards
       └─> Payouts distributed from losing pool
```

## 🚀 Getting Started

### Prerequisites

- **Node.js** v20 or higher ([Download](https://nodejs.org/))
- **Bun** v1.3 or higher ([Download](https://bun.sh/docs/installation))
- **Foundry** ([Installation](https://book.getfoundry.sh/getting-started/installation))
- **CRE CLI** ([Installation](https://docs.chain.link/cre/getting-started/cli-installation))
- **Ethereum Sepolia testnet** configured in your wallet
- **Sepolia ETH** from faucet ([Chainlink Faucet](https://faucets.chain.link/sepolia))
- **Google Gemini API key** ([Get here](https://aistudio.google.com/apikey))

### Installation

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd prediction-market
   ```

2. **Set up environment variables:**
   ```bash
   # Copy the example secrets file
   cp secrets.yaml.example secrets.yaml
   
   # Edit secrets.yaml and add:
   # - Your Google Gemini API key
   # - Private keys for deployment
   # - RPC URLs (optional, defaults provided)
   ```

3. **Install dependencies:**
   ```bash
   cd my-workflow
   bun install
   cd ../contracts
   forge install
   ```

4. **Deploy smart contracts:**
   ```bash
   cd contracts
   # Deploy to Sepolia testnet
   forge create src/PredictionMarket.sol:PredictionMarket \
     --rpc-url $SEPOLIA_RPC_URL \
     --private-key $PRIVATE_KEY \
     --constructor-args 0x15fc6ae953e024d975e77382eeec56a9101f9f88
   ```

   I deployed a sample smart contract on the sepolia testnet: `This is the contract Address
    ``` 0x3c6538D54F0ab3624DE662aEbb3b1B810928e354 ```

    You can also deploy on the tenderly virtual testnet which offers the flexibility of having access to unlimited test tokens for testing purposes.
    Follow the instructions in the Contracts `Readme.md` to deploy to tenderly.
    But these are is a sample smart contract you can interact with from the tenderly testnet that I deployed.
    -   **Deployed Contract**: ``` 0x4E676f9c6aed012DB5c85f47B55b31ECce511B88 ```
-   **Contract Hash**: ``` 0xcb47936ac2ff98858da8e095cead8ff7c2fea09c600b69cfb0c200e38069771c ```

URL: `https://virtual.sepolia.eu.rpc.tenderly.co/22f8c2d3-abf5-4ec1-a737-e18d3a4757d0/address/0x4e676f9c6aed012db5c85f47b55b31ecce511b88`

5. **Configure workflow:**
   - Update `my-workflow/config.staging.json` with your deployed contract address
   - Set market address and chain configuration
   - Ensure API keys are properly configured in `secrets.yaml`

## 📦 Project Structure

```
prediction-market/
├── contracts/                 # Solidity smart contracts
│   ├── src/
│   │   ├── PredictionMarket.sol    # Main market contract
│   │   └── interfaces/
│   │       └── ReceiverTemplate.sol # Chainlink CRE receiver
│   ├── test/                  # Smart contract tests
│   └── foundry.toml           # Foundry config
├── my-workflow/               # CRE workflow automation
│   ├── main.ts               # Workflow entry point
│   ├── httpCallback.ts       # HTTP trigger handler
│   ├── logCallback.ts        # Event trigger handler
│   ├── gemini.ts             # AI integration
│   ├── config.staging.json   # Staging environment config
│   └── package.json
├── project.yaml              # CRE project configuration
├── secrets.yaml.example      # Environment variable template
└── README.md                 # This file
```

## 🔄 Workflow Usage

### Creating a Market

```bash
# Market creation via HTTP trigger
curl -X POST http://workflow-endpoint/create-market \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Will Bitcoin reach $100k by end of 2026?",
    "creator": "0x..."
  }'
```
Here is a transaction sample transaction hash created by me while creating my own market. Make sure you create the market in your `prediction-market` directory using the following commands:
 `cd prediction-market`
`cre workflow simulate my-workflow --broadcast`

a sample hash i created is `0x9649e5cbc98cf935cc8be0f07f5c1970a51e6a7ca80bb3200463ea5369ffba89`
 You can also verify that the market is created using this command: ``` cast call $MARKET_ADDRESS \
  "getMarket(uint256) returns ((address,uint48,uint48,bool,uint16,uint8,uint256,uint256,string))" \
  0 \
  --rpc-url "https://ethereum-sepolia-rpc.publicnode.com"
  ```

### Making a Prediction

```foundry
//using cast
cast send $MARKET_ADDRESS \
  "predict(uint256,uint8)" 0 0 \
  --value 0.01ether \
  --rpc-url "https://ethereum-sepolia-rpc.publicnode.com" \
  --private-key $CRE_ETH_PRIVATE_KEY
```
For my case this was the transaction hash of creating such a transaction: `0xc2c84ac4d954749839e7aba32c9593040be45da06ef043d52efda0a8af5b6d97`

### Requesting Settlement

```javascript
// Market creator triggers settlement
await marketContract.requestSettlement(marketId);
// This emits SettlementRequested event, triggering the log listener
```
```foundry
cast send $MARKET_ADDRESS \
  "requestSettlement(uint256)" 0 \
  --rpc-url "https://ethereum-sepolia-rpc.publicnode.com" \
  --private-key $CRE_ETH_PRIVATE_KEY
  ```

For my market , I got this as the transaction hash: `0x6d3d0aa6ef9cfe9f712fb667acb57f4debf0ecea9bc4a2c45606e65ca5caa09d` after market settlement.

### Claiming Winnings

```javascript
// Winners claim their payouts
const amount = await marketContract.claimWinnings(marketId);
```
```foundry
cast send $MARKET_ADDRESS \
  "claim(uint256)" 0 \
  --rpc-url "https://ethereum-sepolia-rpc.publicnode.com" \
  --private-key $CRE_ETH_PRIVATE_KEY
```


## 🤖 AI-Powered Settlement

The workflow integrates Google's Gemini LLM to analyze market questions and generate settlement recommendations:

1. **Question Analysis**: Gemini analyzes the market question context
2. **Outcome Prediction**: AI generates Yes/No prediction with confidence score
3. **On-Chain Execution**: Result is submitted to smart contract
4. **Reward Distribution**: Pools automatically distribute to winners

## 🧪 Testing

### Run Smart Contract Tests

```bash
cd contracts
forge test
```

### Deploy to Sepolia Testnet

```bash
cd contracts

# Deploy contract
forge create src/PredictionMarket.sol:PredictionMarket \
  --rpc-url https://ethereum-sepolia-rpc.publicnode.com \
  --private-key $PRIVATE_KEY \
  --constructor-args 0x15fc6ae953e024d975e77382eeec56a9101f9f88
```

## 🔐 Security Considerations

- **Forwarder Validation**: Contract validates Chainlink CRE forwarder address
- **Re-entrancy Protection**: Fund transfers use safe patterns
- **State Validation**: All market operations validate current state
- **Access Control**: Market operations restricted by role (creator, predictor)

## 📚 Resources

- **CRE Bootcamp Documentation**: https://smartcontractkit.github.io/cre-bootcamp-2026
- **Chainlink Documentation**: https://docs.chain.link/
- **Foundry Book**: https://book.getfoundry.sh/
- **Solidity Docs**: https://docs.soliditylang.org/

## 🛠️ Development

### Tech Stack

- **Smart Contracts**: Solidity 0.8.24
- **Blockchain**: Ethereum (Sepolia testnet)
- **Workflow Engine**: Chainlink CRE SDK
- **Backend**: TypeScript with Bun runtime
- **Development Framework**: Foundry + Forge

### Environment Configuration

**Staging Configuration** (`config.staging.json`):
```json
{
  "geminiModel": "gemini-1.5-flash",
  "evms": [
    {
      "marketAddress": "0x...",
      "chainSelectorName": "ethereum-testnet-sepolia",
      "gasLimit": "300000"
    }
  ]
}
```

## 📝 License

This project follows the bootcamp curriculum and is built for educational purposes.

## 🤝 Contributing

This is a bootcamp project. Contributions and improvements are welcome!

## 📧 Support

For issues and questions:
1. Check the [CRE Bootcamp Documentation](https://smartcontractkit.github.io/cre-bootcamp-2026)
2. Review the [Reference Repository](https://github.com/smartcontractkit/cre-bootcamp-2026)
3. Consult Chainlink Documentation

---

**Built with ❤️ from Kakooza using Chainlink CRE and Gemini AI**
