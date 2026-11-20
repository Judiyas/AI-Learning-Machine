# PCB 设计规范（AI 协作约束文件）

> 每一块板开工前，先读这份文件；后续所有设计评审、DRC 规则、器件选型都以它为准。改动需要显式更新本文件。

## 打样与制造

- 打样厂：**嘉立创（JLC）**，每月优先用免费打样券（1-2 层、限尺寸）
- 默认层数：L0/L1/L2 阶段 **2 层**；L3 载板 **4 层**
- 板厚：1.6mm；最小线宽/间距：**6/6 mil**（免费券工艺上限内）；载板阶段可放宽到 5/5 mil
- 阻焊：绿色；丝印：白色
- 焊盘：手焊阶段用大焊盘；QFN 必须做热焊盘 + 焊后检查

## 器件选型偏好

- 阻容：0603（手焊友好），能 0805 更好
- MCU：优先 QFN 封装 + 有公开参考设计的型号（ESP32-C3、STM32G0）
- 连接器：USB-C（正反插）、Screw terminal / XH2.54 用于电源
- **禁止**第一块板就上 BGA/等长 DDR/HDMI/MIPI 高速接口
- 每颗 IC 必须配去耦电容：100nF 靠近电源脚 + 10uF 每模块一颗

## 每块板必含（可调设计原则）

- 测试点（TP）：所有电源轨、关键信号至少一个 TP
- 0Ω 可调电阻：不确定的分压/串阻先放 0Ω
- 电源轨 LED 指示（每轨一个，串 10k 电阻）
- 电源入口加防呆：TVS 或至少留位置

## 命名规则

- 项目目录：`~/pcb-training/L{n}-{功能名}/`，如 `L1-esp32c3-min`
- 参考位号：R 电阻、C 电容、L 电感、D 二极管、Q 三极管/MOS、U IC、J 连接器、TP 测试点、Y 晶振
- 网络命名：电源用 `VCC_5V`/`VCC_3V3`/`GND`，信号用功能名如 `LED_DRIVE`

## 仿真（L0 起，每块板下单前必跑）

- 引擎：**ngspice 47**（系统级，已装）；KiCad GUI 内置 Simulator 即基于它，直接 GUI 用
- 无头仿真（AI 跑）：`kicad-cli sch export netlist --format spice --output x.spice x.kicad_sch` → `ngspice -b x.spice`
- 仿真源符号：从 `Simulation_SPICE` 库取（VDC/VAC/VSIN），阻容从 Device 库
- 555 板：先用 .tran 确认振荡频率/占空比；运放/RF 板：先 .ac
- **禁忌**：SKiDL 3.0 的 `generate_netlist(tool='spice')` 有 bug（器件映射不到 PySpice，只吐 .title），一律走 kicad-cli 路线
- 仿真用的是理想模型；R/C 误差、焊装寄生、温漂仿真不管，实测为准

## AI 协作流程（每块板必走）

1. 先读本规范 → 出风险清单
2. SKiDL 脚本生成 → 我审电源连接/悬空脚/去耦遗漏
3. KiCad 里核对 → `kicad-cli sch erc --format json` → 我逐条修
4. DRC → DFM 清单（按打样厂工艺）→ 下单
5. 我出 bring-up 检查清单 → 上电 → debug 树

## 环境

- venv：`source ~/pcb-training/.venv/bin/activate`（已内置 `KICAD10_SYMBOL_DIR`）
- KiCad 10.0.8，`kicad-cli` 可用；ERC 报告格式为 `--format json`
- 符号库：`/usr/share/kicad/symbols`；封装库：`/usr/share/kicad/footprints`