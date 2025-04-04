产品介绍
SUI-Arb是一个专为SUI区块链设计的高级自动化套利系统，能够持续监控链上交易，识别和捕获DEX之间的价格差异，并自动执行套利交易以获取利润。系统采用先进的算法和高性能架构，能够在毫秒级别响应市场变化，获取MEV（矿工可提取价值）机会。
核心特性

多协议支持：支持SUI链上几乎所有主要DEX，包括Turbos、Cetus、KriyaClmm、FlowxClmm、Aftermath等
高性能架构：基于Rust编写的核心引擎，结合多线程并行处理，能高效处理大量交易数据
智能路径寻找：自动发现和优化多跳交易路径（最多支持2跳），最大化套利收益
闪电贷集成：利用闪电贷功能，无需大量初始资金就能执行高价值套利
实时监控：通过Telegram实时通知套利执行情况和收益
自动重启：内置定时重启机制，确保系统长期稳定运行
高级模拟：使用交易模拟器预先验证交易可行性，避免失败交易
竞价机制：支持Shio协议特殊套利机会的竞价机制

技术架构
SUI-Arb采用模块化设计，主要包括以下组件：

事件监听器：监控链上交易事件，包括公共交易、私有交易和Shio协议事件
DEX索引器：收集和索引各DEX的池和价格信息
套利引擎：核心算法模块，负责寻找和评估套利机会
交易构建器：构建复杂的交易序列，包括闪电贷操作
模拟器：预测交易结果，确保可行性和盈利性
执行器：将交易提交到链上
监控系统：跟踪套利性能和利润情况

安装要求
硬件需求

CPU: 8核或更高
内存: 最小16GB，推荐32GB
存储: 最小500GB SSD
网络: 高速稳定的互联网连接

软件需求

操作系统: Ubuntu 20.04 LTS或更高版本
Rust 1.81+
Python 3.8+
SUI节点（全节点或RPC接入）
tmux

安装指南
1. 准备环境
bashCopy# 安装基础依赖
sudo apt update
sudo apt install -y build-essential pkg-config libssl-dev python3-pip python3-venv tmux

# 安装Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env

# 安装SUI CLI
cargo install --locked --git https://github.com/MystenLabs/sui.git --tag mainnet sui
2. 克隆代码库
cd sui-arb
3. 设置Python环境
bashCopycd scripts
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
4. 编译项目
bashCopy# 回到项目根目录
cd ..
cargo build --release
5. 配置
创建.env文件并配置以下参数：
Copy# RPC节点
SUI_RPC_URL=https://your-sui-rpc-url

# 套利账户私钥（小心保管！）
SUI_PRIVATE_KEY=your-private-key

# 利润监控地址（通常与套利账户相同）
PROFIT_ADDRESS=your-wallet-address

# Telegram通知配置
SUI_ARB_BOT_TOKEN=your-telegram-bot-token
GROUP_SUI_ARB=your-telegram-chat-id
THREAD_ONCHAIN_LARGE_PROFIT=your-telegram-thread-id
使用指南
启动系统

生成池相关ID文件（初次使用或更新DEX时需要）

bashCopycargo run --release --bin arb pool-ids --result-path /home/ubuntu/sui/pool_related_ids.txt

启动套利机器人

bashCopycargo run --release --bin arb start-bot \
  --private-key $SUI_PRIVATE_KEY \
  --rpc-url $SUI_RPC_URL \
  --use-db-simulator \
  --max-recent-arbs 5 \
  --workers 10 \
  --num-simulators 18 \
  --preload-path /home/ubuntu/sui/pool_related_ids.txt

启动自动重启脚本

bashCopycd scripts
source venv/bin/activate
./restart_bot.py

启动利润监控

bashCopy# 在另一个终端
cd scripts
source venv/bin/activate
python monitor_profit.py
命令行参数说明
启动机器人的主要参数

--private-key: 套利账户的私钥（必需）
--rpc-url: SUI区块链RPC节点URL（必需）
--use-db-simulator: 使用数据库模拟器（推荐）
--workers: 工作线程数量（建议8-16，取决于硬件）
--num-simulators: 模拟器数量（建议与CPU核心数相当）
--max-recent-arbs: 最近处理过的套利机会缓存数量
--preload-path: 池相关ID文件路径

Gas管理工具
合并Gas代币以优化交易成本：
bashCopy./merge_gas.sh <account_address> <primary_gas_coin_object_id>
监控和管理

查看日志

bashCopy# 查看套利机器人日志
tail -f bot_restarter.log

# 如果使用tmux运行，可以通过以下命令附加到会话
tmux attach -t mev-arb-bot

Telegram通知

系统会自动向配置的Telegram群组发送以下类型的通知：

高利润交易（>1 SUI）
一般利润交易
系统状态更新


停止系统

bashCopy# 杀死tmux会话
tmux kill-session -t mev-arb-bot

# 停止利润监控（在运行脚本的终端按CTRL+C）
进阶配置
调整套利参数
可以在config.rs文件中调整以下参数：

GAS_BUDGET: 交易Gas预算
MAX_HOP_COUNT: 最大交易跳数
MAX_POOL_COUNT: 每种代币考虑的最大池数量
MIN_LIQUIDITY: 考虑套利的最小流动性阈值

自定义DEX支持
系统已支持大多数主流DEX，如需添加新DEX支持：

在protocols目录下创建新的DEX解析器
实现相应的事件解析和交易构建逻辑
在DEX索引器中注册新的协议

自定义监控阈值
在monitor_profit.py中，可调整以下参数：

BALANCE_DIFF_THRESHOLD: 触发通知的最小利润阈值

性能优化

减少模拟器数量如果内存有限
增加工作线程数如果CPU核心充足
调整套利缓存过期时间以平衡响应速度和系统负载
定期重启以释放内存和资源（默认3小时）

故障排除
常见问题

交易持续失败

检查RPC节点连接
确保账户有足够SUI支付Gas
查看日志中的具体错误信息


模拟器错误

尝试重新生成池相关ID文件
增加数据库路径的存储空间


无法识别套利机会

检查DEX索引器是否正常运行
调低MIN_LIQUIDITY阈值
确保正确配置了支持的DEX


系统资源消耗过高

减少工作线程和模拟器数量
增加定时重启频率
限制套利搜索范围



安全注意事项

私钥保护

永远不要分享或暴露您的私钥
考虑为套利系统使用专用钱包


资金管理

不要将所有资金放在套利钱包中
定期将利润转移到更安全的钱包


系统访问

限制服务器访问权限
使用防火墙保护API端点
定期更新所有软件和依赖



法律免责声明
使用本套利系统应遵守相关司法管辖区的法律法规。用户应自行了解并承担使用本系统进行交易的所有风险，包括但不限于市场风险、技术风险和监管风险。
技术支持
如遇到技术问题，请通过以下方式获取支持：

项目GitHub仓库提交Issue
开发者社区论坛
官方Telegram群组
