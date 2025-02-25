```ts
import { RpcEndPoint } from "@/config/address";
import { Metaplex } from "@metaplex-foundation/js";
import { Connection, PublicKey } from "@solana/web3.js";

// 创建与Solana节点的连接
const connection = new Connection(RpcEndPoint, "confirmed");
// 使用Metaplex SDK
const metaplex = Metaplex.make(connection);

// 定义NFT类型
interface NFT {
  mintAddress: string;
  uri: string;
}

/**
 * 获取用户钱包地址的NFT，并根据collection地址过滤
 * @param walletAddress - 用户的钱包地址
 * @returns 返回符合条件的NFT列表
 */
export async function fetchNftsByWallet(walletAddress: string): Promise<NFT[]> {
  try {
    // 转换钱包地址为PublicKey对象
    const owner = new PublicKey(walletAddress);

    // 获取该钱包拥有的所有NFT
    const nfts = await metaplex.nfts().findAllByOwner({ owner });

    // 过滤出属于指定Collection的NFT并返回
    return nfts
      .filter((nft) => nft.collection?.address?.toString() === process.env.NEXT_PUBLIC_COLLECTION_ADDRESS)
      .map((nft) => ({
        mintAddress: nft.mintAddress?.toString(),
        uri: nft.uri,
      }));
  } catch (error) {
    // 捕获并打印错误信息
    console.error("Error fetching NFTs:", error);
    return [];
  }
}
