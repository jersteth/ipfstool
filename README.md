# IPFSTool

A simple solidity tool that lets you store IPFS hashes in a packed uint32 in solidity and convert 
back and forth. Saves gas and allows for gas optimized NFT collections.

IPFS hashes are 34+ bytes by default when sha2-256 is used. They include a hash code and digest length byte prefix in the MultiHash format.
If you control the hash function used (because e.g. you also generate the content) this can be 0x1220 prefix 
(Sha2-256 encoding, 0x20=32 bytes length). Only in that case this library can be used and store an ipfs hash in a bytes32, the perfect storage slot size 
for solidity, which saves gas.

You can use the tokenID of an ERC721 as address for the IPFS uri, saving gas as no extra storage map
 to map tokenId to IPFS hash is required - the tokenid is the ipfs hash itself.

When minting, upload and pin the file to IPFS from client code, convert the Base58 encoded IPFS has to bytes, 
and strip off the leading 0x1220. Call mint() with the remaining 32 bytes.

Example Usage:

```
import "./IPFSTool.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract YourContract is IERC721{

    function mint(
        address to,
        bytes32 ipfsHash) external payable {
        require(msg.value == _price, "incorrect amount paid");
        ...
        ERC721._safeMint(to, uint256(ipfsHash));
    }
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        require(_exists(tokenId), "ERC721Metadata: URI query for nonexistent token");
        return string(abi.encodePacked(_baseURI(), IPFSTool.ipfsToString(tokenId)));
    }
    ... 
}

```

The method is expected to be called from a view or pure method so not tuned for gas consumption.
