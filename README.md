# -sah10k
sah10k/
├── contracts/
│   └── Sah10k.sol
├── scripts/
│   └── deploy.js
├── hardhat.config.js
├── package.json
├── .env.example
├── README.md
└── metadata/          (optional – for later)
{
  "name": "sah10k",
  "version": "1.0.0",
  "description": "sah10k NFT collection – 10,000 supply, $69 mint",
  "scripts": {
    "compile": "hardhat compile",
    "deploy": "hardhat run scripts/deploy.js --network <your-network>",
    "test": "hardhat test"
  },
  "devDependencies": {
    "@nomicfoundation/hardhat-toolbox": "^5.0.0",
    "hardhat": "^2.22.0",
    "dotenv": "^16.4.5"
  },
  "dependencies": {
    "@openzeppelin/contracts": "^5.0.2"
  }
}PRIVATE_KEY=your_wallet_private_key
SEPOLIA_RPC_URL=https://...
PRIVATE_KEY=your_wallet_private_key
SEPOLIA_RPC_URL=https://...
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/ReentrancyGuard.sol";

contract Sah10k is ERC721, ERC721Enumerable, Ownable, ReentrancyGuard {
    uint256 public constant MAX_SUPPLY = 10_000;
    uint256 public mintPrice;          // set in constructor (in wei)
    string private _baseTokenURI;
    bool public mintEnabled = false;

    constructor(
        uint256 _mintPriceInWei,
        string memory baseURI_
    ) ERC721("sah10k", "SAH10K") Ownable(msg.sender) {
        mintPrice = _mintPriceInWei;
        _baseTokenURI = baseURI_;
    }

    function mint(uint256 quantity) external payable nonReentrant {
        require(mintEnabled, "Minting is not enabled");
        require(quantity > 0 && quantity <= 10, "Invalid quantity");
        require(totalSupply() + quantity <= MAX_SUPPLY, "Sold out");
        require(msg.value >= mintPrice * quantity, "Insufficient payment");

        for (uint256 i = 0; i < quantity; i++) {
            uint256 tokenId = totalSupply() + 1;
            _safeMint(msg.sender, tokenId);
        }
    }

    // Owner functions
    function setMintEnabled(bool _enabled) external onlyOwner {
        mintEnabled = _enabled;
    }

    function setMintPrice(uint256 _newPrice) external onlyOwner {
        mintPrice = _newPrice;
    }

    function setBaseURI(string memory baseURI_) external onlyOwner {
        _baseTokenURI = baseURI_;
    }

    function withdraw() external onlyOwner {
        (bool success, ) = payable(owner()).call{value: address(this).balance}("");
        require(success, "Withdraw failed");
    }

    // Required overrides
    function _baseURI() internal view override returns (string memory) {
        return _baseTokenURI;
    }

    function _update(address to, uint256 tokenId, address auth)
        internal
        override(ERC721, ERC721Enumerable)
        returns (address)
    {
        return super._update(to, tokenId, auth);
    }

    function _increaseBalance(address account, uint128 value)
        internal
        override(ERC721, ERC721Enumerable)
    {
        super._increaseBalance(account, value);
    }

    function supportsInterface(bytes4 interfaceId)
        public
        view
        override(ERC721, ERC721Enumerable)
        returns (bool)
    {
        return super.supportsInterface(interfaceId);
    }
}
const hre = require("hardhat");

async function main() {
  // Example: $69 at current ETH price. Update this value!
  // e.g. if 1 ETH = $2500 → 69 / 2500 ≈ 0.0276 ETH
  const mintPriceInWei = hre.ethers.parseEther("0.0276"); // ← change this

  const baseURI = "ipfs://YOUR_CID/"; // replace later

  const Sah10k = await hre.ethers.getContractFactory("Sah10k");
  const nft = await Sah10k.deploy(mintPriceInWei, baseURI);

  await nft.waitForDeployment();
  console.log("sah10k deployed to:", await nft.getAddress());
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
# sah10k

10,000 supply NFT collection  
Mint price: **$69** (adjustable)

## Setup
1. `npm install`
2. Copy `.env.example` → `.env` and fill in your keys
3. `npx hardhat compile`
4. Update the mint price in `scripts/deploy.js` according to current ETH (or other token) price
5. Deploy: `npx hardhat run scripts/deploy.js --network sepolia`

## Features
- Max supply: 10,000
- Owner-controlled mint enable/disable
- Adjustable mint price
- Withdraw function
- ERC-721 Enumerable
- 
