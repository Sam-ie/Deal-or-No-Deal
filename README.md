# Deal or No Deal · 策略提示器（单文件版）

[🇨🇳 中文版](#中文版) · [🇬🇧 English](#english)

------

<a id="中文版"></a>

## 中文版

**Deal or No Deal · 策略提示器** 是一个单文件、零依赖的《Deal or No Deal》网页游戏：26 个箱子、9 轮开箱、银行家逐轮报价。原版游戏被完整内联进一个 HTML 文件，直接以浏览器打开即可进行游戏，无需服务器或任何外部资源。

在保留原版玩法的同时，本作于右侧额外集成了一块实时策略面板。面板不修改游戏本体，而是在运行时读取局面，将银行家每一轮的报价拆解为数字与图表呈现，并基于风险决策模型给出成交或继续的建议。

### 怎么玩

直接以浏览器打开 `index.html`。游戏过程中面板随每一步实时更新；进入结算时，右侧会自动浮出未开箱清单与整局报价走势。

### 策略面板能做什么

银行家每一轮的报价如何得出——面板先计算剩余奖金的期望，再以一条系数条并列呈现标准轮次系数、模型系数与实算系数，并给出 `EV × 系数 = 报价` 的即时校验。浮动部分同样被拆解：档位调整、方差惩罚与实盘 ±5% 的随机抖动各自所占比例清晰呈现。

是否接受报价，由面板的一条标尺呈现：期望 EV、等值线 CE 与当前报价三个标记同屏显示，成交区以绿色标示于 CE 之上。拖动风险厌恶滑块，CE 标记实时移动，报价标记进出成交区，结论即时可见。

对局结束，结算浮层揭示所有未开箱金额，绘制各轮报价散点连线图（金线表示报价、青色虚线表示 CE 阈值、绿红点对应每轮应成交或应继续、白色虚线表示最终奖金），并附策略逐轮建议回顾。

界面语言随游戏自动切换，中英文无需手动设置。

### 方法

策略面板建立在经典决策理论之上：风险厌恶效用刻画对金额的偏好，Bellman 方程描述成交与继续开箱两条路径的最优期望效用，蒙特卡洛序贯模拟用于处理箱子较多时难以穷举的状态空间。银行家报价本身沿用 Deal or No Deal 既有的结构——剩余期望乘轮次系数，再叠加档位与方差调整。

银行家报价：

```
Offer ≈ mean(S) · ( Vy[round] + f − g )   ± 5% 随机
```

其中轮次系数 `Vy = [.12, .22, .32, .42, .55, .68, .78, .88, .95]`，早期轮次仅给出期望的一成左右，临近终局逼近 95%；`f` 随剩余期望高低垫高或压低，`g` 在分布偏散时再作下压。

效用与续值：

```
u(x; γ) = (x^(1−γ) − 1) / (1−γ)

V(S) = max( u(B(S)),  (1/|S|)·Σⱼ V(S\{j}) )

CE = u⁻¹( QNoDeal(S; γ) )      DEAL ⟺ Offer ≥ CE
```

箱子数 `|S| ≤ 12` 时，续值以精确动态规划求解；`|S| > 12` 时改用蒙特卡洛序贯模拟——每条路径每步仅采样一个后续局面，顶层取数千条模拟平均，复杂度随箱子数线性增长，而非指数爆炸。

### 文件

| 文件             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| `index.html`     | 单文件成品，内联游戏与策略面板，直接打开即玩。               |
| `build_local.js` | 构建脚本，把原版游戏内联并注入策略面板，产出 `index.html`。改完策略逻辑后运行 `node build_local.js` 重建。 |

------

<a id="english"></a>

## English

**Deal or No Deal · Strategy Advisor** is a single-file, dependency-free build of the Deal or No Deal web game: 26 cases, 9 rounds of opening, and a banker who bids each round. The original game is fully inlined into one HTML file you can open directly in a browser — no server, no external assets.

On top of the original game, this build adds a live strategy panel on the right. The panel leaves the game untouched and instead reads the board at runtime, breaking the banker's offer down into figures and charts and advising deal-versus-no-deal through a risk-decision model.

### How to play

Open `index.html` in a browser. The panel updates with every move; at settlement the right side floats out the unopened amounts and the whole-game offer trajectory.

### What the panel does

How the banker's offer for the round is derived — the panel first computes the remaining expected value, then renders a coefficient bar placing the standard round coefficient, the model coefficient, and the reverse-engineered actual coefficient side by side, with a live verification `EV × coefficient = offer`. The float is likewise decomposed: the shares from the level adjustment, the spread penalty, and the game's ±5% randomness are shown clearly.

Whether to accept the offer is shown on a gauge: expected value EV, your certainty-equivalent CE, and the current offer appear on one track, the deal zone shaded above CE. Dragging the risk-aversion slider moves the CE mark live while the offer mark crosses in and out of the deal zone, so the decision is immediate.

At the end of a game, the settlement overlay reveals every unopened amount and plots the offer trajectory (gold line for offers, cyan dashed for the CE threshold, green/red dots for deal-or-continue each round, white dashed for final winnings), with a round-by-round recap of the strategy's advice.

The interface follows the game's language automatically, with no manual switch between Chinese and English.

### The method

The panel rests on classical decision theory: a risk-averse utility captures how money is valued, the Bellman equation describes the optimal expected utility between dealing now and keeping cases open, and Monte-Carlo sequential simulation handles the state space that grows intractable with many cases. The banker's offer follows Deal or No Deal's own structure — remaining expectation times a round coefficient, plus level and spread adjustments.

Banker's offer:

```
Offer ≈ mean(S) · ( Vy[round] + f − g )   ± 5% random
```

The round coefficients `Vy = [.12, .22, .32, .42, .55, .68, .78, .88, .95]` yield just over a tenth of the mean early and approach 95% near the end; `f` pads up or cuts down with the remaining mean, and `g` pulls down slightly when the pool is dispersed.

Utility and continuation:

```
u(x; γ) = (x^(1−γ) − 1) / (1−γ)

V(S) = max( u(B(S)),  (1/|S|)·Σⱼ V(S\{j}) )

CE = u⁻¹( QNoDeal(S; γ) )      DEAL ⟺ Offer ≥ CE
```

For `|S| ≤ 12` the continuation value is computed by exact dynamic programming; for `|S| > 12` it switches to Monte-Carlo sequential simulation — each path samples one next outcome per step, averaged over thousands of rollouts at the top level, so cost grows linearly with the number of cases rather than exploding exponentially.

### Files

| File             | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| `index.html`     | The single-file deliverable: the inlined game plus the strategy panel. Open to play. |
| `build_local.js` | Build script that inlines the original game and injects the panel, producing `index.html`. Run `node build_local.js` to rebuild after changing strategy logic. |
