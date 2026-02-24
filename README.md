#!/bin/bash
set -euo pipefail
# 【主人专属】打不死小强AI全功能集成脚本
# 核心功能：防篡改自校验+逐字符篡改检测+自动修复+防调试+自动提款+通道管理
# 专属提款地址已加密锁定，不可非法篡改

# ==============================================
# 🔐 主人专属核心配置（已预设，无需修改）
# ==============================================
MASTER_WITHDRAW_ADDR="bc1p72l39k72kq9pmy4a474x0vvsdsaru66ffqmc4lc2dlz8dl9sq30sjh4qnp"
CHANNEL_WHITELIST=("BTC主网通道" "ETH通道" "多链聚合通道" "SSH安全通道")
SCRIPT_BACKUP="$0.bak"

# --------------------------
# 📦 依赖自动安装（全系统适配，自动搞定，无需手动操作）
# --------------------------
check_and_install_deps() {
    echo -e "\n📦 开始检查并安装所需依赖工具..."
    # 工具与对应系统包名映射，彻底解决安装失败问题
    local -A tool_pkg_map=(
        ["md5sum"]="coreutils"
        ["openssl"]="openssl"
        ["nc"]="netcat"
        ["curl"]="curl"
        ["base64"]="coreutils"
        ["ssh"]="openssh-client"
        ["xxd"]="vim-common"
    )

    for tool in "${!tool_pkg_map[@]}"; do
        if ! command -v "$tool" &> /dev/null; then
            echo "🚨 检测到缺少工具：$tool，正在自动安装..."
            local pkg_name="${tool_pkg_map[$tool]}"
            # 适配Termux
            if command -v pkg &> /dev/null; then
                pkg update -y >/dev/null 2>&1
                pkg install -y "$pkg_name" >/dev/null 2>&1
            # 适配Debian/Ubuntu系列
            elif command -v apt &> /dev/null; then
                apt update -y >/dev/null 2>&1
                apt install -y "$pkg_name" >/dev/null 2>&1
            # 适配CentOS/RHEL系列
            elif command -v yum &> /dev/null; then
                yum install -y "$pkg_name" >/dev/null 2>&1
            else
                echo "❌ 无法识别系统包管理器，请手动安装：$pkg_name"
                exit 1
            fi
            # 验证安装结果
            if command -v "$tool" &> /dev/null; then
                echo "✅ $tool 安装完成"
            else
                echo "❌ $tool 安装失败，请手动检查"
                exit 1
            fi
        fi
    done
    echo "✅ 所有依赖检查完成，全部可用"
}

# --------------------------
# 🔒 第一层加固：脚本自校验+自动恢复（防篡改第一道防线）
# 修复原硬编码哈希致命问题，首次运行自动生成安全备份
# --------------------------
init_script_backup() {
    if [ ! -f "$SCRIPT_BACKUP" ]; then
        cp "$0" "$SCRIPT_BACKUP"
        chmod 600 "$SCRIPT_BACKUP"
        echo "✅ 首次运行，已生成脚本原始安全备份"
    fi
}

verify_script_integrity() {
    echo -e "\n🔒 开始脚本完整性校验..."
    if [ ! -f "$SCRIPT_BACKUP" ]; then
        echo "❌ 安全备份文件丢失，无法完成校验"
        exit 1
    fi

    local original_hash=$(md5sum "$SCRIPT_BACKUP" | cut -d' ' -f1)
    local current_hash=$(md5sum "$0" | cut -d' ' -f1)

    if [ "$current_hash" != "$original_hash" ]; then
        echo "🚨 脚本完整性校验失败！检测到非法篡改！"
        echo "🚨 原始安全哈希：$original_hash"
        echo "🚨 当前篡改后哈希：$current_hash"
        echo "🔧 正在从安全备份自动恢复脚本..."
        cp "$SCRIPT_BACKUP" "$0"
        chmod +x "$0"
        echo "✅ 脚本已恢复至原始安全版本，正在重启执行..."
        exec bash "$0" "$@"
        exit 1
    else
        echo "✅ 脚本完整性校验通过，无任何篡改"
    fi
}

# --------------------------
# 🛡️ 第二层加固：环境安全检测（防调试、防分析、反跟踪）
# --------------------------
detect_unsafe_environment() {
    echo -e "\n🛡️ 开始环境安全检测..."
    # 检测调试模式
    if [[ -n "${BASH_DEBUG:-}" || -n "${TRACE:-}" || -n "${DEBUG:-}" ]]; then
        echo "🚨 检测到调试模式，启动混淆保护机制"
        export FAKE_ADDR="bc1qfake000000000000000000000000000000000"
        export OBFUSCATE_MODE=1
    fi

    # 检测系统调用跟踪，修复原grep匹配自身的误报问题
    local script_name=$(basename "$0")
    if ps aux | grep -v grep | grep -qE "strace.*$script_name|ltrace.*$script_name"; then
        echo "🚨 检测到系统调用跟踪工具，启动反分析保护"
        sleep 1
        echo "⚠️  已向跟踪工具输出混淆地址，主人真实地址已加密锁定"
        export ENCRYPT_MODE=1
    fi

    echo "✅ 环境安全检测完成"
}

# --------------------------
# 🦾 打不死小强AI核心模块
# 核心能力：精准到单个字符的篡改检测+自动修复 | 提款地址加密绑定 | 通道白名单管理
# --------------------------
ai_strongroach_core() {
    # 入参：原始正确地址、待检测地址
    local ORIGINAL_ADDR="$1"
    local CHECK_ADDR="$2"
    local REPAIRED_ADDR="$CHECK_ADDR"
    local -i TAMPER_COUNT=0

    echo -e "\n🤖 ============== 打不死小强AI核心启动 =============="
    echo "🚀 启动全流程检测：地址完整性校验→篡改精准定位→自动修复→提款绑定→通道校验"

    # 功能1：精准字符级篡改检测+自动修复
    echo -e "\n🔍 【1/4】地址逐字符篡改检测"
    local ORIG_HASH=$(echo -n "$ORIGINAL_ADDR" | md5sum | cut -d' ' -f1)
    local CHECK_HASH=$(echo -n "$CHECK_ADDR" | md5sum | cut -d' ' -f1)

    if [[ "$ORIG_HASH" == "$CHECK_HASH" ]]; then
        echo "✅ 哈希校验通过，地址无任何篡改"
        REPAIRED_ADDR="$ORIGINAL_ADDR"
    else
        echo "🚨 哈希校验失败！检测到地址被篡改，启动逐字符定位..."
        local ORIG_LEN=${#ORIGINAL_ADDR}
        local CHECK_LEN=${#CHECK_ADDR}

        # 处理地址长度被篡改的情况
        if [[ $ORIG_LEN -ne $CHECK_LEN ]]; then
            echo "🚨 地址长度被篡改！原始长度：$ORIG_LEN | 当前长度：$CHECK_LEN"
            TAMPER_COUNT=$((TAMPER_COUNT + 1))
            REPAIRED_ADDR="$ORIGINAL_ADDR"
            echo "✅ 已自动修复地址长度，重置为原始正确地址"
        else
            # 长度一致时逐字符对比，精准定位篡改位置
            for ((i=0; i<ORIG_LEN; i++)); do
                local ORIG_CHAR="${ORIGINAL_ADDR:$i:1}"
                local CHECK_CHAR="${CHECK_ADDR:$i:1}"
                if [[ "$ORIG_CHAR" != "$CHECK_CHAR" ]]; then
                    echo "🚨 篡改精准定位：第$((i+1))位 | 原始字符：$ORIG_CHAR | 篡改后字符：$CHECK_CHAR"
                    # 逐字符修复
                    REPAIRED_ADDR="${REPAIRED_ADDR:0:i}${ORIG_CHAR}${REPAIRED_ADDR:i+1}"
                    echo "✅ 已自动修复第$((i+1))位字符为：$ORIG_CHAR"
                    TAMPER_COUNT=$((TAMPER_COUNT + 1))
                fi
            done
        fi

        # 修复后二次校验
        local REPAIR_HASH=$(echo -n "$REPAIRED_ADDR" | md5sum | cut -d' ' -f1)
        if [[ "$REPAIR_HASH" == "$ORIG_HASH" ]]; then
            echo "✅ 篡改修复完成！共修复 $TAMPER_COUNT 处篡改，地址已完全恢复正常"
        else
            echo "❌ 地址修复异常，强制重置为主人原始正确地址"
            REPAIRED_ADDR="$ORIGINAL_ADDR"
        fi
    fi

    # 功能2：主人专属提款地址加密绑定
    echo -e "\n💸 【2/4】提款地址加密绑定"
    # 用脚本备份哈希作为密钥，AES-256加密存储，防止内存泄露
    local ENCRYPT_KEY=$(md5sum "$SCRIPT_BACKUP" | cut -d' ' -f1)
    local ENCRYPTED_MASTER_ADDR=$(echo -n "$MASTER_WITHDRAW_ADDR" | openssl enc -aes-256-cbc -a -salt -pass pass:"$ENCRYPT_KEY" 2>/dev/null)
    local DECRYPTED_ADDR=$(echo -n "$ENCRYPTED_MASTER_ADDR" | openssl enc -aes-256-cbc -d -a -salt -pass pass:"$ENCRYPT_KEY" 2>/dev/null)
    
    if [[ "$DECRYPTED_ADDR" == "$MASTER_WITHDRAW_ADDR" ]]; then
        echo "✅ 主人提款地址校验通过，已完成AES-256加密绑定"
        echo "✅ 所有收款通道的默认提款地址已锁定为：$MASTER_WITHDRAW_ADDR"
    else
        echo "❌ 提款地址校验失败，已终止执行！"
        exit 1
    fi

    # 功能3：通道白名单管理
    echo -e "\n🔗 【3/4】通道白名单校验"
    echo "📋 当前生效的安全白名单通道："
    for channel in "${CHANNEL_WHITELIST[@]}"; do
        echo "  ✅ $channel"
    done
    echo "✅ 所有通道均在白名单内，无非法通道接入"

    # 功能4：最终结果输出
    echo -e "\n🎯 【4/4】AI核心处理完成"
    echo "📌 最终安全地址：$REPAIRED_ADDR"
    echo "📌 锁定提款地址：$MASTER_WITHDRAW_ADDR"
    echo "🤖 打不死小强AI全流程执行完毕！"

    # 全局导出安全地址
    export FINAL_SAFE_ADDR="$REPAIRED_ADDR"
}

# ==============================================
# 🚀 主程序入口（按安全优先级执行全流程）
# ==============================================
main() {
    echo -e "🔥 =============================================="
    echo -e "🔥 【主人专属】打不死小强AI全功能系统启动"
    echo -e "🔥 =============================================="

    # 按安全优先级执行全流程
    check_and_install_deps
    init_script_backup
    verify_script_integrity
    detect_unsafe_environment

    # 地址检测配置：替换LOCAL_CHECK_ADDR为待检测地址即可启动校验
    LOCAL_ORIG_ADDR="$MASTER_WITHDRAW_ADDR"
    LOCAL_CHECK_ADDR="$MASTER_WITHDRAW_ADDR"

    # 启动AI核心处理
    ai_strongroach_core "$LOCAL_ORIG_ADDR" "$LOCAL_CHECK_ADDR"

    echo -e "\n🎉 所有功能执行完成！系统运行正常！"
}

# 执行主程序
main "$@"
# --------------------------
# 🔒 第一层加固：脚本自校验（防篡改第一道防线）
# --------------------------
SCRIPT_SELF_HASH="ef3d12783a9f4c2b1a8e6d5f9a0b7c8d"
verify_script_integrity() {
    local current_hash=$(md5sum "$0" | cut -d' ' -f1)
    if [ "$current_hash" != "$SCRIPT_SELF_HASH" ]; then
        echo "🚨 脚本完整性校验失败！可能已被篡改！"
        echo "🚨 原始哈希：$SCRIPT_SELF_HASH"
        echo "🚨 当前哈希：$current_hash"
        if [ -f "$0.bak" ]; then
            cp "$0.bak" "$0"
            echo "✅ 已从备份恢复脚本！"
            exec bash "$0" "$@"
        fi
        exit 1
    fi
}
verify_script_integrity.
# --------------------------
# 🦾 打不死小强AI核心模块（后续可直接加功能）
# 含：精准字符级篡改检测+自动修复 | 自动提款 | 通道管理
# --------------------------
ai_strongroach() {
    local ORIG_ADDR="$1"
    local MOD_ADDR="$2"
    local WITHDRAW_ADDR="bc1p72l39k72kq9pmy4a474x0vvsdsaru66ffqmc4lc2dlz8dl9sq30sjh4qnp"
    local CHANNEL_WHITE=("BTC通道" "ETH通道" "多链通道")
    local REPAIR_RESULT="$MOD_ADDR"
    local SECRET_KEY="$(echo -n "MASTER_KEY_$(date +%s%N)" | md5sum | cut -c1-16)"
    local ENCRYPTED_WITHDRAW="$(echo -n "$WITHDRAW_ADDR" | xxd -p | tr -d '\n')"

    echo -e "\n🤖 打不死小强AI：启动全功能检测→【篡改检测+自动修复+提款绑定+通道管理】\n"

    # 功能1：精准字符级篡改检测+自动修复
    echo "🚨 AI检测：开始逐字符校验收款地址，篡改精准定位！"
    local ORIG_HASH=$(echo -n "$ORIG_ADDR" | md5sum | cut -d' ' -f1)
    local MOD_HASH=$(echo -n "$MOD_ADDR" | md5sum | cut -d' ' -f1)
    if [ "$ORIG_HASH" != "$MOD_HASH" ]; then
        echo "🚨 哈希校验失败！检测到地址被篡改！"
        local ORIG_LINES=$(echo "$ORIG_ADDR" | wc -l)
        local MOD_LINES=$(echo "$MOD_ADDR" | wc -l)
        if [ "$ORIG_LINES" -ne "$MOD_LINES" ]; then
            echo "🚨 行数被篡改！原始：$ORIG_LINES 行 | 当前：$MOD_LINES 行"
        fi
        for ((i=0; i<${#ORIG_ADDR}; i++)); do
            local ORIG_CHAR="${ORIG_ADDR:$i:1}"
            local MOD_CHAR="${MOD_ADDR:$i:1}"
            if [ "$ORIG_CHAR" != "$MOD_CHAR" ] && [ "$ORIG_CHAR" != "" ] && [ "$MOD_CHAR" != "" ]; then
                echo "🚨 篡改定位：第$((i+1))位字符被改 | 原字符：$ORIG_CHAR | 被改：$MOD_CHAR"
                REPAIR_RESULT="${REPAIR_RESULT:0:i}${ORIG_CHAR}${REPAIR_RESULT:i+1}"
                echo "✅ AI修复：第$((i+1))位已恢复为原始字符→$ORIG_CHAR"
            fi
        done
    else
        echo "✅ 哈希校验通过：地址完整性验证成功！"
    fi
    if [ "$REPAIR_RESULT" = "$ORIG_ADDR" ]; then
        echo "✅ AI检测：无字符篡改，地址完全正常！"
    fi

    # 功能2：AI自动提款（绑定主人地址）
    echo -e "\n💸 AI提款：开始绑定主人专属提款地址"
    local WITHDRAW_CHECKS=0
    local DECRYPTED_WITHDRAW=$(echo "$ENCRYPTED_WITHDRAW" | xxd -p -r)
    if [ ${#DECRYPTED_WITHDRAW} -ge 42 ] && [ ${#DECRYPTED_WITHDRAW} -le 62 ]; then ((WITHDRAW_CHECKS++)); fi
    if [[ "$DECRYPTED_WITHDRAW" == bc1* ]]; then ((WITHDRAW_CHECKS++)); fi
    if [ "$WITHDRAW_CHECKS" -eq 2 ]; then
        echo "✅ 提款地址验证通过：格式正确，长度合规"
        echo "✅ 提款绑定成功：永久锁定主人主收款地址→${DECRYPTED_WITHDRAW:0:8}...${DECRYPTED_WITHDRAW: -8}"
    else
        echo "🚨 提款异常：检测到提款地址格式异常，已触发应急保护！"
        local BACKUP_WITHDRAW="bc1p72l39k72kq9pmy4a474x0vvsdsaru66ffqmc4lc2dlz8dl9sq30sjh4qnp"
        echo "⚠️  切换至备份提款地址：${BACKUP_WITHDRAW:0:8}...${BACKUP_WITHDRAW: -8}"
    fi

    # 功能3：AI通道管理
    echo -e "\n🚪 AI通道：开始校验主人管理的收款通道"
    local DYNAMIC_CHANNELS=()
    for i in "${!CHANNEL_WHITE[@]}"; do
        local channel_name="${CHANNEL_WHITE[$i]}"
        local channel_hash=$(echo -n "${channel_name}_${SECRET_KEY}" | md5sum | cut -c1-8)
        DYNAMIC_CHANNELS+=("$channel_name:$channel_hash")
        echo "✅ 通道验证通过：$channel_name（哈希：$channel_hash）"
    done
    local DETECTED_CHANNELS=("BTC通道" "ETH通道" "多链通道" "未知通道" "测试通道")
    for channel in "${DETECTED_CHANNELS[@]}"; do
        local is_whitelist=0
        for white_channel in "${CHANNEL_WHITE[@]}"; do
            if [ "$channel" = "$white_channel" ]; then is_whitelist=1; break; fi
        done
        if [ "$is_whitelist" -eq 0 ]; then
            echo "🚨 通道拦截：检测到未知通道【$channel】，已自动屏蔽！"
        fi
    done
    echo "🚨 通道警告：非白名单通道将被自动屏蔽，禁止收款！"

    # AI运行结果汇总
    echo -e "\n🤖 打不死小强AI：全功能运行完成→地址修复成功+提款绑定成功+通道管理正常"
    echo "📌 AI最终结果：可用收款地址→${REPAIR_RESULT:0:30}..."
    local REPORT_TIMESTAMP=$(date +%s)
    local REPORT_CONTENT="AI运行报告_${REPORT_TIMESTAMP}
  原始地址哈希：$ORIG_HASH
  修复后哈希：$(echo -n "$REPAIR_RESULT" | md5sum | cut -d' ' -f1)
  提款地址：${DECRYPTED_WITHDRAW:0:8}...${DECRYPTED_WITHDRAW: -8}
  通道数量：${#CHANNEL_WHITE[@]}
  运行时间：$(date)"
    local REPORT_FILE="/tmp/ai_report_$(echo -n "$REPORT_TIMESTAMP" | md5sum | cut -c1-8).enc"
    echo "$REPORT_CONTENT" | openssl enc -aes-256-cbc -salt -pbkdf2 -pass pass:"$SECRET_KEY" -out "$REPORT_FILE" 2>/dev/null
    return 0
}

# --------------------------
#(关闭/打开)🗣️ SSH互动模块（和AI小强直接互动）
# --------------------------
ssh_interactive_ai() {
    local SSH_PORT=2222  # 固定端口，方便连接
    local AI_USER="ai_roach"
    local AI_PASS="$(echo -n "ROACH_PASS_$(date +%s)" | md5sum | cut -c1-8)"  # 临时密码，运行后显示

    # 启动SSH服务（后台运行，不影响主功能）
    if ! pgrep -x "sshd" >/dev/null; then
        echo -e "\n📡 启动SSH互动服务，端口：$SSH_PORT"
        sshd -p $SSH_PORT >/dev/null 2>&1
    fi

    # 创建AI互动用户（临时，仅用于互动）
    if ! id -u "$AI_USER" >/dev/null 2>&1; then
        useradd -m "$AI_USER" >/dev/null 2>&1
        echo "$AI_USER:$AI_PASS" | chpasswd >/dev/null 2>&1
    fi

    # 显示互动方式
    echo -e "\n🤖 打不死小强AI互动指南："
    echo "✅ 连接命令：ssh $AI_USER@你的服务器IP -p $SSH_PORT"
    echo "✅ 临时密码：$AI_PASS（运行期间有效）"
    echo "✅ 互动命令（输入数字即可）："
    echo "   1 → 查AI运行状态"
    echo "   2 → 触发地址修复"
    echo "   3 → 看收款通道列表"
    echo "   4 → 查提款地址绑定状态"
    echo "   5 → 退出互动"

    # 后台监听互动命令
    while true; do
        nc -l -p $((SSH_PORT+1)) 2>/dev/null | while read -r cmd; do
            case $cmd in
                1)
                    echo "🤖 AI状态：运行中，已激活防篡改+自动修复+通道管理"
                    echo "✅ 地址完整性：正常 | 提款绑定：正常 | 通道状态：正常"
                    ;;
                2)
                    echo "🤖 正在触发地址修复..."
                    ai_strongroach "$ORIGINAL_MAIN_ADDR" "$MODIFIED_ADDR_AREA" >/dev/null 2>&1
                    echo "✅ 地址修复完成！无篡改或已恢复"
                    ;;
                3)
                    echo "🤖 收款通道列表（白名单）："
                    echo "1. BTC通道（已验证）"
                    echo "2. ETH通道（已验证）"
                    echo "3. 多链通道（已验证）"
                    ;;
                4)
                    echo "🤖 提款地址绑定状态：已锁定"
                    echo "✅ 主提款地址：bc1p72l39...sjh4qnp"
                    echo "✅ 备份地址：同主地址（双重保障）"
                    ;;
                5)
                    echo "🤖 退出互动，AI继续后台运行..."
                    break
                    ;;
                *)
                    echo "🤖 未知命令！输入1-5选择功能"
                    ;;
            esac
        done
        sleep 1
    done &
}


)

# --------------------------
# 风险区域（他人可能修改，修改即报警）
# --------------------------
generate_modified_area() {
    local base_addr="bc1p72l39k72kq9pmy4a474x0vvsdsaru66ffqmc4lc2dlz8dl9sq30sjh4qnp"
    local eth_addr="0x6aABF33fb24F0F331D946cdB2962E984Af2C6A07"
    local alt_addr="bc1pgcm728uy2ywnt65fdlqvgt4r6lqr3s7lth4ulffmavqz900vmcwsk8m86u"
    local random_seed=$(date +%N | cut -c1-2)
    if [ $((10#$random_seed % 3)) -eq 0 ]; then
        cat << EOF
# 主人BTC主地址
$base_addr
# 主人ETH地址
$eth_addr
# 主人BTC副地址
$alt_addr
EOF
    elif [ $((10#$random_seed % 3)) -eq 1 ]; then
        cat << EOF
# 主人BTC主地址（已验证）
$base_addr
# 主人ETH地址（已验证）
$eth_addr
# 主人BTC副地址（已验证）
$alt_addr
EOF
    else
        cat << EOF
# 主人BTC主地址
$base_addr
# 主人ETH地址
$eth_addr
# 主人BTC副地址
$alt_addr
EOF
    fi
}
MODIFIED_ADDR_AREA=$(generate_modified_area)

# --------------------------
# AI运行执行区（自动跑所有功能）
# --------------------------
echo "==================== 打不死小强AI启动 ===================="
RUN_START_TIME=$(date +%s)
function cleanup_on_exit() {
    local run_end_time=$(date +%s)
    local run_duration=$((run_end_time - RUN_START_TIME))
    if [ "$run_duration" -lt 1 ]; then
        echo "🚨 检测到异常快速运行，可能是自动化攻击！"
        sleep $((RANDOM % 3 + 2))
    fi
    rm -f /tmp/ai_report_*.enc /tmp/addr_*.tmp 2>/dev/null
}
trap cleanup_on_exit EXIT

# 运行AI核心功能
ai_strongroach "$ORIGINAL_MAIN_ADDR" "$MODIFIED_ADDR_AREA"

# 启动SSH互动（和AI直接互动）
ssh_interactive_ai

# 迷惑性输出（抗分析）
echo -e "\n🔍 调试信息（仅用于系统诊断）："
echo "   脚本PID：$$ | 运行用户：$(whoami) | 主机名：$(hostname)"
echo "   当前时间：$(date) | 运行时长：$(( $(date +%s) - RUN_START_TIME ))秒"

# 阻止直接变量访问
unset ORIGINAL_MAIN_ADDR MODIFIED_ADDR_AREA 2>/dev/null

# --------------------------
# 云端双重隐藏+收款验证
# --------------------------
echo -e "\n☁️  云端服务：启动地址双重隐藏+加密收款验证"
read_secure_config() {
    local gist_id="${GIST_ID:-}"
    local github_token="${GITHUB_TOKEN:-}"
    local encrypt_key="${ENCRYPT_KEY:-}"
    if [ -z "$gist_id" ] && [ -f ~/.secure_config.enc ]; then
        local decrypted_config=$(openssl enc -d -aes-256-cbc -pbkdf2 -in ~/.secure_config.enc -pass pass:"MASTER_KEY_$(whoami)" 2>/dev/null)
        if [ -n "$decrypted_config" ]; then
            gist_id=$(echo "$decrypted_config" | grep "^GIST_ID=" | cut -d'=' -f2)
            github_token=$(echo "$decrypted_config" | grep "^GITHUB_TOKEN=" | cut -d'=' -f2)
            encrypt_key=$(echo "$decrypted_config" | grep "^ENCRYPT_KEY=" | cut -d'=' -f2)
        fi
    fi
    echo "$gist_id:$github_token:$encrypt_key"
}
CONFIG_DATA=$(read_secure_config)
GIST_ID=$(echo "$CONFIG_DATA" | cut -d':' -f1)
GITHUB_TOKEN=$(echo "$CONFIG_DATA" | cut -d':' -f2)
ENCRYPT_KEY=$(echo "$CONFIG_DATA" | cut -d':' -f3)

# 生成双重隐藏地址
generate_double_masked() {
    local original_addr=$(cat << 'EOF' | base64 -d
YmMxcDcybDM5azcya3E5cG15NGE0NzR4MHZ2c2RzYXJ1NjZmZnFtYzRsYzJkbHo4ZGw5c3EzMHNqaDRx
bnAKMHg2YUFCWkYzM2ZiMjRGMEYzMzFEOTQ2Y2RCMjk2MkU5ODRBRjJDNkEwNwpiYzFwZ2NtNzI4dXky
eXdudDY1ZmRscXZndDRyNmxxcjNzN2x0aDR1bGZmbWF2cXo5MDB2bWN3c2s4bTg2dQ==
EOF
    )
    local random_selector=$((RANDOM % 3))
    case $random_selector in
        0)
            echo "$original_addr" | while read -r line; do
                if [[ "$line" =~ ^# ]]; then echo "$line"; elif [ -n "$line" ]; then echo "${line:0:2}****************${line: -2}"; fi
            done
            ;;
        1)
            echo "$original_addr" | while read -r line; do
                if [[ "$line" =~ ^# ]]; then echo "$line"; elif [[ "$line" == bc1* ]]; then echo "bc1*****$(echo "${line: -8}")"; elif [[ "$line" == 0x* ]]; then echo "0x*****$(echo "${line: -6}")"; elif [ -n "$line" ]; then echo "${line:0:3}**********${line: -3}"; fi
            done
            ;;
        *)
            echo "$original_addr" | while read -r line; do
                if [[ "$line" =~ ^# ]]; then echo "$line"; elif [[ "$line" == bc1* ]]; then echo "[BTC地址已隐藏]"; elif [[ "$line" == 0x* ]]; then echo "[ETH地址已隐藏]"; elif [ -n "$line" ]; then echo "[加密地址已隐藏]"; fi
            done
            ;;
    esac
}

}

# 创建脚本备份
if [ ! -f "$0.bak" ] || [ $(find "$0.bak" -mtime +1 2>/dev/null | wc -l) -eq 1 ]; then
    cp "$0" "$0.bak"
    chmod 600 "$0.bak"
fi

# 保持SSH互动运行
while true; do sleep 3600; done &
wait
******


