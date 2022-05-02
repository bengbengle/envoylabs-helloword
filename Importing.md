# Importing

In [Publishing](./Publishing.md), we discussed how you can publish your contract to the world.
This looks at the flip-side, how can you use someone else's contract (which is the same
question as how they will use your contract). Let's go through the various stages.

## Verifying Artifacts

Before using remote code, you most certainly want to verify it is honest.

The simplest audit of the repo is to simply check that the artifacts in the repo
are correct. This involves recompiling the claimed source with the claimed builder
and validating that the locally compiled code (hash) matches the code hash that was
uploaded. This will verify that the source code is the correct preimage. Which allows
one to audit the original (Rust) source code, rather than looking at wasm bytecode.

在使用远程代码之前，您肯定想验证它是否真实。 最简单的 repo 审计是简单地检查 repo 中的工件 是正确的。
这涉及使用声明的构建器重新编译声明的源 并验证本地编译的代码（哈希）是否与之前的代码哈希匹配 上传。 
这将验证源代码是正确的原像。 这使得 审计原始（Rust）源代码，而不是查看 wasm 字节码。

We have a script to do this automatic verification steps that can
easily be run by many individuals. Please check out
[`cosmwasm-verify`](https://github.com/CosmWasm/cosmwasm-verify/blob/master/README.md)
to see a simple shell script that does all these steps and easily allows you to verify
any uploaded contract.

## Reviewing

Once you have done the quick programatic checks, it is good to give at least a quick
look through the code. A glance at `examples/schema.rs` to make sure it is outputing
all relevant structs from `contract.rs`, and also ensure `src/lib.rs` is just the
default wrapper (nothing funny going on there). After this point, we can dive into
the contract code itself. Check the flows for the execute methods, any invariants and
permission checks that should be there, and a reasonable data storage format.

You can dig into the contract as far as you want, but it is important to make sure there
are no obvious backdoors at least.

一旦你完成了快速的程序检查，最好至少给一个快速的 查看代码。
查看 `examples/schema.rs` 以确保它正在输出 来自 `contract.rs` 的所有相关结构，并确保 `src/lib.rs` 只是 默认包装器（那里没什么好笑的）。 
在这一点之后，我们可以潜入 合约代码本身。 检查执行方法的流程，任何不变量和 应该存在的权限检查，以及合理的数据存储格式。 
您可以根据需要深入了解合同，但重要的是要确保 至少没有明显的后门。

## Decentralized Verification

It's not very practical to do a deep code review on every dependency you want to use,
which is a big reason for the popularity of code audits in the blockchain world. We trust
some experts review in lieu of doing the work ourselves. But wouldn't it be nice to do this
in a decentralized manner and peer-review each other's contracts? Bringing in deeper domain
knowledge and saving fees.

Luckily, there is an amazing project called [crev](https://github.com/crev-dev/cargo-crev/blob/master/cargo-crev/README.md)
that provides `A cryptographically verifiable code review system for the cargo (Rust) package manager`.

I highly recommend that CosmWasm contract developers get set up with this. At minimum, we
can all add a review on a package that programmatically checked out that the json schemas
and wasm bytecode do match the code, and publish our claim, so we don't all rely on some
central server to say it validated this. As we go on, we can add deeper reviews on standard
packages.

If you want to use `cargo-crev`, please follow their
[getting started guide](https://github.com/crev-dev/cargo-crev/blob/master/cargo-crev/src/doc/getting_started.md)
and once you have made your own *proof repository* with at least one *trust proof*,
please make a PR to the [`cawesome-wasm`]() repo with a link to your repo and
some public name or pseudonym that people know you by. This allows people who trust you
to also reuse your proofs.

There is a [standard list of proof repos](https://github.com/crev-dev/cargo-crev/wiki/List-of-Proof-Repositories)
with some strong rust developers in there. This may cover dependencies like `serde` and `snafu`
but will not hit any CosmWasm-related modules, so we look to bootstrap a very focused
review community.

对您要使用的每个依赖项进行深入的代码审查并不是很实际， 这是代码审计在区块链世界中流行的一个重要原因。
我们相信 一些专家审查代替自己做这项工作。
但是这样做不是很好吗 以分散的方式和同行评审彼此的合同？引入更深的领域  
…但不会碰到任何与 CosmWasm 相关的模块，所以我们希望引导一个非常专注的模块 审查社区。