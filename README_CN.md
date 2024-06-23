# A Sui Coin Example

[English](./README.md) | 中文版

在这个例子里，我们将展示：

* 修改一个简单的 Coin 合约，并部署到 Sui 网路；
* 给自己 mint 一些这种 Coin；
* 在一个叫做 Flex 的去中心交易所（DEX）中，将这种 Coin 和其他 Coin 组成一个交易对并提供初始的流动性。

--------

提示：Flex 在 Movement M2 devnet 中发布了他们的测试合约。其中 Exchange 对象的信息如下：

```text
ObjectType: 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd::exchange::Exchange
ObjectID: 0x39a5098d25482d8948f9f1eef3f43cc6ec5b39ddc53c6057af3650a06c5539ea
```

Flex 在 Sui testnet 中发布的测试合约的信息：

```text
objectType: 0x1fbb91bd77221cf17450a4378f2d93100cf65725e0099e4da71f62070ce4b729::exchange::Exchange
objectId: 0xa556bc09e966ab42ddcc98b84bc1d26c00cc6438d8dc61a787cfc696200099e7
```


## 准备工作

我们建议，你可以考虑配置 Sui CLI 工具，切换到 Movement M2 devnet，进行下面的测试。

* 安装 [Sui CLI](https://docs.sui.io/build/install)。
* [配置你的 Sui CLI 工具](https://docs.movementlabs.xyz/developers/sui-developers/using-sui-cli)，
  这样，如果你是一个 Sui 开发者，你基本不需要改变你的工作流程，就可以将你的应用部署到 Movement 网络上。



## 修改 Coin 合约以及部署

将 `./sources/my_coin.move` 中的 Coin 相关的信息改为你喜欢的样子。主要是以下几个地方：

```move
    // ...
    const COIN_DECIMALS: u8 = 9; // The number of decimal places for the coin

    /// Name of the coin
    struct MY_COIN has drop {}

    fun init(otw: MY_COIN, ctx: &mut TxContext) {
        // Get a treasury cap for the coin and give it to the transaction sender
        let (treasury_cap, metadata) = coin::create_currency<MY_COIN>(
            otw,
            COIN_DECIMALS,
            b"MY_COIN",
            b"My coin name",
            b"My coin description",
            option::none(),
            ctx
        );
        // ...
    }
    // ...
```

下面行文中，为了方便，我们可能直接将“你想要发布的 Coin”称为 `MY_COIN`，而不再多做解释，希望你注意到这一点。

将修改后的合约部署到 Sui 网路：

```shell
sui client publish --gas-budget 200000000 --skip-fetch-latest-git-deps --skip-dependency-verification
```

部署如果成功，输出信息下面这样：

```text
Transaction Digest: 5TobcdrTY35aJupfw1UsaQ2BsJ9bgYc9dQwgXzSNh7aj

│ Created Objects:                                                                                                              │
│  ┌──                                                                                                                          │
│  │ ObjectID: 0x27518522f67c0f8161116a9f93ba3c75a488449ac93876177f7cc5a103b41b82                                               │
│  │ Sender: 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2                                                 │
│  │ Owner: Account Address ( 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2 )                              │
│  │ ObjectType: 0x2::coin::TreasuryCap<0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN>   │


│ Published Objects:                                                                                                            │
│  ┌──                                                                                                                          │
│  │ PackageID: 0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49                                              │
│  │ Version: 1                                                                                                                 │
│  │ Digest: FFfcVavBko2QCxLzCbwYxbxaevbeKqU3rucrPbgz7iYx                                                                       │

```

记录输出中的“已创建（Created）”的 `TreasuryCap` 对象的 ID
（在上面的例子中是 `0x27518522f67c0f8161116a9f93ba3c75a488449ac93876177f7cc5a103b41b82`），
后面会用到。

记录输出中的“已发布的（Published）”包的 ID
（在上面的示例中是 `0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49`），
后面会用到。

## 给自己 mint 一些 Coin

假设我们想要给自己 mint 100 万个 `MY_COIN`，我们可以使用下面的命令
（注意将 Package ID 以及 `TreasuryCap` 对象的 ID 的值 
`0x27518522f67c0f8161116a9f93ba3c75a488449ac93876177f7cc5a103b41b82` 替换为你发布合约时得到的实际的值）：

```shell
sui client call --package 0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49 \
--module my_coin --function mint \
--args 0x27518522f67c0f8161116a9f93ba3c75a488449ac93876177f7cc5a103b41b82 \
1000000000000000 \
--gas-budget 20000000
```

如果 mint 成功，输出类似下面这样：

```text

╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                                               │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                                             │
│  ┌──                                                                                                                         │
│  │ ObjectID: 0x4f8f7415357f31da4df9e713084d72ef1fb455186824db9df7b5bb5fa42f84d1                                              │
│  │ Sender: 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2                                                │
│  │ Owner: Account Address ( 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2 )                             │
│  │ ObjectType: 0x2::coin::Coin<0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN>         │

```

记录输出中的“已创建（Created）”的 `Coin` 对象的 ID，
（在上面的示例中是 `0x4f8f7415357f31da4df9e713084d72ef1fb455186824db9df7b5bb5fa42f84d1`），
后面会用到。

### 查看你拥有哪些 `MY_COIN` 对象

你也可以事后使用下面的命令查看你拥有的 `MY_COIN` 对象：

```shell
curl -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1,"method":"suix_getCoins","params":["{YOUR_ADDRESS}","{MY_COIN_PACKAGE_ID}::my_coin::MY_COIN"]}' https://sui.devnet.m2.movementlabs.xyz
```

## 在 Flex DEX 中提供交易对并初始化流动性

下面我们以 Flex DEX 部署在 Movement M2 devnet 网络上的合约为例，展示如何在 Flex DEX 中提供交易对并初始化流动性。
当然，你需要先按照上面的步骤将 `MY_COIN` 合约部署到 Movement M2 devnet。

然后，看看你的 Sui CLI 钱包中有多少当前网络的“本币”对象：

```shell
sui client gas
```

输出类似下面这样：

```text
╭────────────────────────────────────────────────────────────────────┬────────────────────┬──────────────────╮
│ gasCoinId                                                          │ mistBalance (MIST) │ suiBalance (SUI) │
├────────────────────────────────────────────────────────────────────┼────────────────────┼──────────────────┤
│ 0x1ed9b740efd757ed9135b4e1d53ea8974ee4fa7dda566ae9b9cce32c4f56dba4 │ 1357383478         │ 1.35             │
│ 0x4130234b30141d0003f0f005c1e28b231dfc8a5653e4641b5e3b88ec4e61a829 │ 200000000          │ 0.20             │
```

记录其中的一个 coin 对象的 ID，后面会用到。

---

提示：如果返回的对象少于一个，你可以给自己的 Sui CLI 钱包账户转一些 coin，已达到将一个 coin 对象 split 为两个的目的。
你可以这样执行转账命令，（注意将占位符 `{YOUR_ADDRESS}` 和 `{YOUR_COIN_OBJECT_ID}` 替换为你实际的值）：

```shell
sui client pay-sui --amounts 200000000 --recipients {YOUR_ADDRESS} --gas-budget 10000000 \
--input-coins {YOUR_COIN_OBJECT_ID}
```

---

现在，可以使用下面的命令在 Flex DEX 中创建交易对并初始化流动性了。
在下面的示例命令中，我们假设了以下几个参数的值（你需要将它们替换为实际的值）：

* Flex DEX 的合约包 ID 为 `0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd`；
* 你的 Coin 的类型为 `0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN`；
* 你的 Sui CLI 钱包拥有的“本币”对象的 ID 为 `0x4130234b30141d0003f0f005c1e28b231dfc8a5653e4641b5e3b88ec4e61a829`；
* 你的 Sui CLI 钱包拥有的 `MY_COIN` 对象的 ID 为 `0x4f8f7415357f31da4df9e713084d72ef1fb455186824db9df7b5bb5fa42f84d1`；
* 你想要提供的初始化流动性为 0.1 个网络“本币”和 1 个 `MY_COIN`。
* “池子”的手续费率为 3/1000。

执行：

```shell
sui client call --package 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd \
--module token_pair_service --function initialize_liquidity \
--type-args '0x2::sui::SUI' \
0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN \
--args \
0x4130234b30141d0003f0f005c1e28b231dfc8a5653e4641b5e3b88ec4e61a829 \
100000000 \
0x4f8f7415357f31da4df9e713084d72ef1fb455186824db9df7b5bb5fa42f84d1 \
1000000000 \
3 1000 \
--gas-budget 30000000
```

如果执行成功，输出类似下面这样：

```text
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                                                                                                                                           │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                                                                                                                                         │
│  ┌──                                                                                                                                                                                                                     │
│  │ ObjectID: 0x31ee0a05a8a1348da363255e4eb8eeac19a6440f7f33ff7796f1d2e01dce8052                                                                                                                                          │
│  │ Sender: 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2                                                                                                                                            │
│  │ Owner: Shared                                                                                                                                                                                                         │
│  │ ObjectType: 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd::token_pair::TokenPair<0x2::sui::SUI, 0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN>            │
│  │ Version: 2961795                                                                                                                                                                                                      │
│  │ Digest: B4Up72dnoKVMCTFFbGfVUNsDpAb8Nip9ZBddtKkzarBf                                                                                                                                                                  │
│  └──                                                                                                                                                                                                                     │
│  ┌──                                                                                                                                                                                                                     │
│  │ ObjectID: 0x52fc66b4bb96f8a7cef27db9ece28f0509b26fe0c6b000d6a027006f900ca6b5                                                                                                                                          │
│  │ Sender: 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2                                                                                                                                            │
│  │ Owner: Account Address ( 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2 )                                                                                                                         │
│  │ ObjectType: 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd::liquidity_token::LiquidityToken<0x2::sui::SUI, 0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN>  │
│  │ Version: 2961795                                                                                                                                                                                                      │
│  │ Digest: 2bWVe2CpwXmEUQ6361DHbihDQGyWrgB5Cj8QBe2VpDDp                                                                                                                                                                  │
│  └──                                                                                                                                                                                                                     │
│  ┌──                                                                                                                                                                                                                     │
│  │ ObjectID: 0xa52d9f92f0a12f85f1108cc612f214e13e564d51b3b340633a8bac150e7f910c                                                                                                                                          │
│  │ Sender: 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2                                                                                                                                            │
│  │ Owner: Account Address ( 0xfc50aa2363f3b3c5d80631cae512ec51a8ba94080500a981f4ae1a2ce4d201c2 )                                                                                                                         │
│  │ ObjectType: 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd::token_pair::AdminCap                                                                                                                  │
│  │ Version: 2961795                                                                                                                                                                                                      │
│  │ Digest: 8jRe4jG3aLqvZNaYXEFezfKzpuraLP69o7Vx4tExMgGb                                                                                                                                                                  │
│  └──                                                                                                                                                                                                                     │

```

在上面的示例中，你创建的交易对（也就是所谓的“池子”）的 ID 是 `0x31ee0a05a8a1348da363255e4eb8eeac19a6440f7f33ff7796f1d2e01dce8052`。
你可以使用 Sui CLI 查看这个“池子”的信息：

```shell
sui client object 0x31ee0a05a8a1348da363255e4eb8eeac19a6440f7f33ff7796f1d2e01dce8052
```

注意输出中的 `AdminCap` 对象的 ID。
这个对象是你创建交易对的时候，合约向你发送（transfer）的一个对象，代表了对这个“池子”管理权限。
如果你想要更新这个“池子”的费率，你需要用到。

```text
│               │ │ fields            │ ╭─────────────────┬───────────────────────────────────────────────────────────────────────────────╮                                                                                              │ │
│               │ │                   │ │ admin_cap       │  0xa52d9f92f0a12f85f1108cc612f214e13e564d51b3b340633a8bac150e7f910c           │                                                                                              │ │
│               │ │                   │ │ fee_denominator │  1000                                                                         │                                                                                              │ │
│               │ │                   │ │ fee_numerator   │  3                                                                            │ 
```

## 以“本币”兑换 `MY_COIN`

该函数的参数如下：

* `token_pair`: `&mut TokenPair<X, Y>`.
* `x_coin`: `Coin<X>`. 在 CLI 中传入“本币”的 Object ID。
* `x_amount`: `u64`. 打算兑换（换入）的 X 代币数量。
* `y_coin`: `&mut Coin<Y>`. 用于接受换出的 Y 代币（`MY_COIN`）的 Coin 对象 Id。
* `expected_y_amount_out`: `u64`. 你能接受的 Y 代币的最小兑出数量。如果合约计算发现实际可获得的 Y 代币数量小于这个值，则交易失败。关于如何计算这个值，我们这里先忽略。

示例命令：

```shell
sui client call --package 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd --module token_pair_service --function swap_x \
--type-args '0x2::sui::SUI' \
0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN \
--args \
'0x31ee0a05a8a1348da363255e4eb8eeac19a6440f7f33ff7796f1d2e01dce8052' \
'0x4130234b30141d0003f0f005c1e28b231dfc8a5653e4641b5e3b88ec4e61a829' \
'"100"' \
'0x4f8f7415357f31da4df9e713084d72ef1fb455186824db9df7b5bb5fa42f84d1' \
'"1"' \
--gas-budget 30000000
```

也许你要问，兑换的用户如果钱包里面没有 `MY_COIN` 对象，那么如何传入 `y_coin` 这个参数呢？
好问题。如果你是一名前端开发者，此时可以使用 [PTBs](https://docs.sui.io/concepts/transactions/prog-txn-blocks)，
先[创建一个“零”`MY_COIN` 对象](https://docs.sui.io/references/framework/sui-framework/coin#function-zero)，
然后将这个对象的引用传入到这个函数中；最后，不要忘记将这个 `MY_COIN` 对象转移到用户的钱包中。


## 修改交易对的费率

如果你想要修改“池子”的费率，假设池子的对象 ID 是 `0x31ee0a05a8a1348da363255e4eb8eeac19a6440f7f33ff7796f1d2e01dce8052`，
它的 `AdminCap` 对象的 ID 是 `0xa52d9f92f0a12f85f1108cc612f214e13e564d51b3b340633a8bac150e7f910c`（你需要拥有这个对象），
你想要将费率修改为 1/1000，可以这样执行命令：

```shell
sui client call --package 0x71ec440c694153474dd2a9c5c19cf60e2968d1af51aacfa24e34ee96a2df44dd \
--module token_pair_aggregate --function update_fee_rate \
--type-args '0x2::sui::SUI' \
0xa666a577f4b1c4eda0e4113a8ded8fb1002c2fc5f8ce676e097c8e0be9694e49::my_coin::MY_COIN \
--args 0x31ee0a05a8a1348da363255e4eb8eeac19a6440f7f33ff7796f1d2e01dce8052 \
0xa52d9f92f0a12f85f1108cc612f214e13e564d51b3b340633a8bac150e7f910c \
1 1000 \
--gas-budget 30000000
```
