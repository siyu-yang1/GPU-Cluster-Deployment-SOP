# Python 自动化日志解析与 Excel 报表生成工具 SOP

在 GPU 集群运维与硬件交付过程中，面对数百台节点的巡检日志、部署日志（如 `syslog`、`dmesg`、`nvidia-smi` 输出文件），人工排查不仅耗时且容易遗漏。本工具旨在通过 Python 实现日志数据的**自动正则提取、数据清洗、分类汇总，并最终生成可视化表格及 Excel 统计报表**。

---

## 一、 核心功能与架构设计

1. **日志正则解析：** 快速检索日志文件中的时间戳、日志级别（`INFO`/`WARN`/`ERROR`）、节点 IP 以及错误详情。
2. **错误汇总与分类：** 提取常见的硬件故障（如 GPU 掉卡、PCIe 降速、ECC 内存错误）及网络异常。
3. **自动化报表导出：** 使用 `pandas` 与 `openpyxl` 将解析结果整理为结构化的 `.xlsx` Excel 文件，并自动设置表格样式与条件高亮。

---

## 二、 完整 Python 脚本代码

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
智算中心自动化日志解析与 Excel 报表生成工具
"""

import os
import re
import pandas as pd
from openpyxl.styles import PatternFill, Font, Alignment, Border, Side

# 1. 拟定的日志正则表达式 (根据实际日志格式调整)
LOG_PATTERN = re.compile(
    r'(?P<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\s+'
    r'\[(?P<level>INFO\vert{}WARN\vert{}ERROR\vert{}FATAL)\]\s+'
    r'\[(?P<node_ip>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})\]\s+'
    r'(?P<message>.*)'
)

def parse_log_file(log_file_path):
    """解析单个日志文件并提取关键字段"""
    parsed_data = []
    if not os.path.exists(log_file_path):
        print(f"[Error] 日志文件不存在: {log_file_path}")
        return parsed_data

    with open(log_file_path, 'r', encoding='utf-8', errors='ignore') as f:
        for line_num, line in enumerate(f, 1):
            match = LOG_PATTERN.match(line.strip())
            if match:
                data = match.groupdict()
                data['line_number'] = line_num
                parsed_data.append(data)
    return parsed_data

def generate_excel_report(parsed_data, output_excel_path):
    """将解析数据生成具备样式的 Excel 报表"""
    if not parsed_data:
        print("[Warning] 无有效解析数据，跳过报表生成。")
        return

    df = pd.DataFrame(parsed_data)
    
    # 重新排列列顺序
    df = df[['timestamp', 'node_ip', 'level', 'line_number', 'message']]
    df.columns = ['发生时间', '节点IP', '日志级别', '行号', '异常与日志信息']

    # 写入 Excel
    with pd.ExcelWriter(output_excel_path, engine='openpyxl') as writer:
        df.to_excel(writer, sheet_name='日志解析明细', index=False)
        
        # 统计各级别数量
        summary_df = df['日志级别'].value_counts().reset_index()
        summary_df.columns = ['日志级别', '出现次数']
        summary_df.to_excel(writer, sheet_name='统计汇总', index=False)

    # 美化 Excel 格式
    beautify_excel(output_excel_path)
    print(f"[Success] 报表已成功导出至: {output_excel_path}")

def beautify_excel(file_path):
    """设置 Excel 样式（高亮 ERROR、调整列宽、加边框）"""
    import openpyxl
    wb = openpyxl.load_workbook(file_path)
    ws = wb['日志解析明细']

    # 填充颜色定义
    red_fill = PatternFill(start_color="FFC7CE", end_color="FFC7CE", fill_type="solid") # ERROR 浅红
    header_fill = PatternFill(start_color="4F81BD", end_color="4F81BD", fill_type="solid") # 表头蓝色

    thin_border = Border(
        left=Side(style='thin', color='D9D9D9'),
        right=Side(style='thin', color='D9D9D9'),
        top=Side(style='thin', color='D9D9D9'),
        bottom=Side(style='thin', color='D9D9D9')
    )

    # 美化表头
    for cell in ws[1]:
        cell.fill = header_fill
        cell.font = Font(name="Microsoft YaHei", size=11, bold=True, color="FFFFFF")
        cell.alignment = Alignment(horizontal="center", vertical="center")

    # 遍历数据行设置样式及高亮
    for row in ws.iter_rows(min_row=2, max_row=ws.max_row, min_col=1, max_col=ws.max_column):
        level_cell = row[2] # 日志级别所在列
        is_error = level_cell.value in ['ERROR', 'FATAL']

        for cell in row:
            cell.font = Font(name="Microsoft YaHei", size=10)
            cell.border = thin_border
            if is_error:
                cell.fill = red_fill

    # 自动调整列宽
    for col in ws.columns:
        max_len = max(len(str(cell.value or '')) for cell in col)
        col_letter = openpyxl.utils.get_column_letter(col[0].column)
        ws.column_dimensions[col_letter].width = max(max_len + 3, 12)

    wb.save(file_path)

if __name__ == "__main__":
    # 示例运行路径
    LOG_FILE = "cluster_deployment.log"
    OUTPUT_EXCEL = "GPU集群部署日志解析报表.xlsx"
    
    data = parse_log_file(LOG_FILE)
    generate_excel_report(data, OUTPUT_EXCEL)

```
## 三、 使用环境准备与运行说明
### 1. 安装依赖库
在 Python 3.8+ 环境下安装数据处理与 Excel 样式依赖：
```bash
pip install pandas openpyxl

```
### 2. 运行脚本
```bash
python parse_log_to_excel.py

```
## 四、 成果与交付效果
 1. **自动高亮标注：** 自动将 ERROR / FATAL 级别的异常整行高亮为红色，方便快速定位故障点。
 2. **多 Dimensions 汇总：** 自动生成 日志解析明细 与 统计汇总 两个 Sheet，为运维周报/月报提供直观的数据支撑。
```

---

### 第三步：提交保存

1. 点击右上角绿色的 **`Commit changes...`** 按钮。
2. 弹窗出现后，再点击一次绿色的 **`Commit changes`** 即可！

```
