# 历史声场绘图数据

从本机 24 个既有三维场提取绘图切片和轴线。原有报告、图、指标表和小型复算输入保留在各算例目录。每份 NPZ 重新读取后与原矩阵切片逐元素一致。另核对了 12 个目标框峰值的轴向剖面和 4 个焦深轴线，与已发布 CSV 一致。源文件路径和归档文件的 SHA-256 见 [清单](manifest.json)。

- `x_m`、`y_m`、`z_m` 是以米为单位的原生网格，`dt_s` 和 `Nt` 记录时步和步数。
- `pressure_xz_center_pa` 和 `pressure_xz_target_box_peak_pa` 的维度为 `[z, x]`。`pressure_xy_*_pa` 和 `qp_xy_source_pa2_s` 的维度为 `[y, x]`。实际切片坐标见 `y_mid_m`、`z_source_m`、`z_target_m`、`target_box_peak_m` 和 `forward_peak_m`。
- `pressure_on_axis_pa` 对应最接近零横向坐标的轴线。数据使用未标定的 Pa 和 Pa²·s。Qp 采用离散的 `Nt × dt × p_rms²`。新增截面及全域指标需要重算。
