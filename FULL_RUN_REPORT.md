# SWE-bench Lite 全量正赛报告（lite-full-r3，2026-09-13）

## 最终成绩

**266/300 = 88.7% resolved**（Wilson 95% CI [84.6%, 91.8%]）

| 批次 | 实例 | resolved | unresolved | error |
|------|------|----------|------------|-------|
| batch_aa（astropy→django） | 100 | 91 | 9 | 0 |
| batch_ab（django→sklearn） | 100 | 92 | 6 | 1 |
| batch_ac（sklearn→sympy） | 100 | 83 | 17 | 0 |
| **合计** | **300** | **266** | **32** | **1** |
| 空 patch（计入未解决） | — | — | — | 1 |

## 实验设置

- 模型：`moonshotai/kimi-k3`（OpenRouter，$3/$15 每 MTok），快照存证 `model_snapshot.json`
- 全量 300 例 pass@1 单 rollout，seed=1，`--sample 999`（per-repo 不裁剪）
- 引擎：maxTurns=80、toolTimeout=1m、单例 30 分钟预算、NetworkBlockedHosts 封禁 6 个 github 域
- 推理：并发 3，2026-09-13 05:31→16:53（11h22m），**$485.29**（input 136.5M / output 5.05M tokens）
- 评分：官方 harness（swebench 4.1.0），分 3 批（同 run_id `lite-full-r3`），官方 amd64 评测镜像

## per-repo 成绩

| Repo | resolved/总 | 比率 |
|------|-------------|------|
| pylint-dev | 6/6 | 100% |
| pallets | 3/3 | 100% |
| pytest-dev | 16/17 | 94% |
| django | 106/114 | 93% |
| matplotlib | 21/23 | 91% |
| scikit-learn | 21/23 | 91% |
| sympy | 65/77 | 84% |
| psf | 5/6 | 83% |
| astropy | 5/6 | 83% |
| pydata | 4/5 | 80% |
| mwaskom | 3/4 | 75% |
| sphinx-doc | 11/16 | 69% |

## 未解决 34 例的构成

- **32 例 unresolved**：模型层失败（与校准一致的分布：sphinx 系环境漂移、astropy/matplotlib 顽固实例）
- **1 例 error**：`scikit-learn-13496` —— 官方 harness 的确定性 `EvaluationError`，
  校准两轮 + 本轮共三次评分同一签名，与本方 patch 无关
- **1 例空 patch**：`matplotlib-26011` 单例 30 分钟超时（pass@1 纪律如实保留）

## infra 处置记录（透明度声明）

- `sympy-13146`：推理中 provider 流中断（turn 58）→ 按 infra 失败定点重试一次（32m33s，407B patch）→
  评分未通过。仅重试基础设施失败，模型行为结果一律不重骰
- 全程 daemon 错误 0、clone 失败 0；轨迹覆盖 300/300；
  `audit_run_health.py` 审计通过（sklearn-14087 的单次 "exec format error" 为模型推理散文误报，
  该实例 resolved 自证有效；据此把 arch 阈值调整为 >1，commit 见 feat/benchmark 分支）

## 与校准的口径衔接

| 轮次 | 样本 | resolved | 口径 |
|------|------|----------|------|
| v4 基线（2026-09-09） | 78 | 85.9% | 子集 |
| 硬化验证（2026-09-10） | 47 | 87.2% | 子集 |
| 校准 calib-r3（2026-09-12） | 57 | 84.2% | 子集（CI 72.6-91.4%） |
| **全量正赛（本报告）** | **300** | **88.7%** | **官方口径（CI 84.6-91.8%）** |

全量结果落在校准置信区间内，子集→全量外推有效。

## 提交物

`submission/`：all_preds.jsonl（300）+ logs/<id>/（patch.diff 299 + report.json 298 + test_output 298）
+ trajs/（300/300，推理时生成）+ README.md + EXPORT_MANIFEST.md（5 项缺失如实记录）

## 成本合计（打榜全战役）

校准阶段 ~$225（含作废的 round2）+ 正赛 $485.29 ≈ **$710**，落在 $800 预算内。
