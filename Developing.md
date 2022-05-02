# Developing

If you have recently created a contract with this template, you probably could use some
help on how to build and test the contract, as well as prepare it for production. This
file attempts to provide a brief overview, assuming you have installed a recent
version of Rust already (eg. 1.58.1+).
如果您最近使用此模板创建了合同，您可能可以使用一些 帮助如何构建和测试合同，以及为生产做准备。这 文件试图提供一个简短的概述，假设您安装了最近的 已经是 Rust 的版本（例如 1.58.1+）

## Prerequisites

Before starting, make sure you have [rustup](https://rustup.rs/) along with a
recent `rustc` and `cargo` version installed. Currently, we are testing on 1.58.1+.

And you need to have the `wasm32-unknown-unknown` target installed as well.

You can check that via:

```sh
rustc --version
cargo --version
rustup target list --installed
# if wasm32 is not listed above, run this
rustup target add wasm32-unknown-unknown
```

## Compiling and running tests

Now that you created your custom contract, make sure you can compile and run it before
making any changes. Go into the repository and do:
现在您已经创建了自定义合约，请确保您可以在之前编译和运行它 进行任何更改。进入存储库并执行以下操作：

```sh
# this will produce a wasm build in ./target/wasm32-unknown-unknown/release/YOUR_NAME_HERE.wasm
cargo wasm

# this runs unit tests with helpful backtraces
RUST_BACKTRACE=1 cargo unit-test

# auto-generate json schema
cargo schema
```

### Understanding the tests

The main code is in `src/contract.rs` and the unit tests there run in pure rust,
which makes them very quick to execute and give nice output on failures, especially
if you do `RUST_BACKTRACE=1 cargo unit-test`.

We consider testing critical for anything on a blockchain, and recommend to always keep
the tests up to date.

主要代码在 `src/contract.rs` 中，那里的单元测试以纯 rust 的形式运行， 这使得它们可以非常快速地执行并在失败时给出很好的输出，尤其是如果您执行“RUST_BACKTRACE=1 货物单元测试”。 

我们认为测试对区块链上的任何事物都至关重要，并建议始终保持最新的测试。

## Generating JSON Schema

While the Wasm calls (`instantiate`, `execute`, `query`) accept JSON, this is not enough
information to use it. We need to expose the schema for the expected messages to the
clients. You can generate this schema by calling `cargo schema`, which will output
4 files in `./schema`, corresponding to the 3 message types the contract accepts,
as well as the internal `State`.

虽然 Wasm 调用（`instantiate`、`execute`、`query`）接受 JSON，但这还不够 使用它的信息。我们需要将预期消息的模式暴露给 客户。您可以通过调用 `cargo schema` 生成此架构，它将输出 `./schema` 中的 4 个文件，对应合约接受的 3 种消息类型， 以及内部的“状态”。

These files are in standard json-schema format, which should be usable by various
client side tools, either to auto-generate codecs, or just to validate incoming
json wrt. the defined schema.

这些文件是标准的 json-schema 格式，应该可以被各种 客户端工具，用于自动生成编解码器，或仅用于验证传入 json 写的。 定义的架构。

## Preparing the Wasm bytecode for production
## 为生产准备 Wasm 字节码

Before we upload it to a chain, we need to ensure the smallest output size possible,
as this will be included in the body of a transaction. We also want to have a
reproducible build process, so third parties can verify that the uploaded Wasm
code did indeed come from the claimed rust code.

在我们将其上传到链之前，我们需要确保尽可能小的输出大小， 因为这将包含在交易主体中。我们也想拥有一个 可重现的构建过程，因此第三方可以验证上传的 Wasm 代码确实来自声称的 rust 代码。

To solve both these issues, we have produced `rust-optimizer`, a docker image to
produce an extremely small build output in a consistent manner. The suggest way
to run it is this:

为了解决这两个问题，我们制作了 `rust-optimizer`，一个 docker 镜像 以一致的方式产生极小的构建输出。建议方式 运行它是这样的：

```sh
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer:0.12.4
```

Or, If you're on an arm64 machine, you should use a docker image built with arm64.
```sh
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/code/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/rust-optimizer-arm64:0.12.4
```

We must mount the contract code to `/code`. You can use a absolute path instead
of `$(pwd)` if you don't want to `cd` to the directory first. The other two
volumes are nice for speedup. Mounting `/code/target` in particular is useful
to avoid docker overwriting your local dev files with root permissions.
Note the `/code/target` cache is unique for each contract being compiled to limit
interference, while the registry cache is global.

This is rather slow compared to local compilations, especially the first compile
of a given contract. The use of the two volume caches is very useful to speed up
following compiles of the same contract.

This produces an `artifacts` directory with a `PROJECT_NAME.wasm`, as well as
`checksums.txt`, containing the Sha256 hash of the wasm file.
The wasm file is compiled deterministically (anyone else running the same
docker on the same git commit should get the identical file with the same Sha256 hash).
It is also stripped and minimized for upload to a blockchain (we will also
gzip it in the uploading process to make it even smaller).


我们必须将合约代码挂载到`/code`。您可以改用绝对路径 `$(pwd)` 如果你不想先 `cd` 到目录。另外两个 卷很适合加速。
挂载 `/code/target` 特别有用 避免 docker 使用 root 权限覆盖您的本地开发文件。 
请注意，`/code/target` 缓存对于每个正在编译的合约都是唯一的，以限制 干扰，而注册表缓存是全局的。 
与本地编译相比，这相当慢，特别是……对加速很有用 以下编译相同的合同。 
这会生成一个带有“PROJECT_NAME.wasm”的“artifacts”目录，以及 `checksums.txt`，包含 wasm 文件的 Sha256 哈希值。 
wasm 文件是确定性编译的（任何其他运行相同的 在同一个 git commit 上的 docker 应该得到具有相同 Sha256 哈希的相同文件）。 
它也被剥离并最小化以上传到区块链（我们还将 在上传过程中对其进行 gzip 压缩以使其更小）。
