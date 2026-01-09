---
id: strategy-draco-compress
type: strategy
related_ids: [draco_compress_safe.py]
created: 2026-01-08
---

# Strategy: USDZ Draco 压缩

## 目标

将 `/Users/mac/Downloads/1.usdz` 使用 Draco 压缩，输出到当前工作目录：
- `1.usda` (ASCII 格式)
- `1.usdc` (Crate 二进制格式)

## 侦察结果

### 输入文件分析

| 属性 | 值 |
|:---|:---|
| 路径 | `/Users/mac/Downloads/1.usdz` |
| 格式 | ZIP 归档 (USDZ) |
| 主文件 | `1.usdc` (144 MB) |
| 纹理 | 4 个 JPEG + 1 个 HDR |

### 可用工具

[`draco_compress_safe.py`](../../draco_compress_safe.py:1) - 现有脚本

**默认参数:**
- `qp=14` (Position 量化位数)
- `qt=12` (Texture Coord 量化位数)
- `qn=10` (Normal 量化位数)
- `cl=10` (Compression Level)

**用法:**
```bash
python draco_compress_safe.py <input> <output>
```

## 执行计划

<MathSpec>
```
PROCEDURE compress_usdz_to_draco(input_path, output_dir):
    # Step 1: 生成 USDA
    output_usda = output_dir + "/1.usda"
    RUN: python draco_compress_safe.py input_path output_usda
    # 结果: 1.usda + 1.usda.draco/ 目录
    
    # Step 2: 生成 USDC
    output_usdc = output_dir + "/1.usdc"
    RUN: python draco_compress_safe.py input_path output_usdc
    # 结果: 1.usdc + 1.usdc.draco/ 目录
    
    # Step 3: 验证
    VERIFY: 文件存在且大小 > 0
END
```
</MathSpec>

## 命令序列

```bash
# Step 1: 生成 USDA
python draco_compress_safe.py /Users/mac/Downloads/1.usdz ./1.usda

# Step 2: 生成 USDC  
python draco_compress_safe.py /Users/mac/Downloads/1.usdz ./1.usdc

# Step 3: 验证输出
ls -la ./1.usda ./1.usdc ./1.usda.draco/ ./1.usdc.draco/
```

## 预期输出

```
./
├── 1.usda              # ASCII USD 文件 (引用 draco)
├── 1.usda.draco/       # Draco 压缩的 mesh 数据
│   └── *.drc
├── 1.usdc              # Binary USD 文件 (引用 draco)
└── 1.usdc.draco/       # Draco 压缩的 mesh 数据
    └── *.drc
```

## 风险与约束

### DO NOTs
- ❌ 不要修改原始 usdz 文件
- ❌ 不要更改默认压缩参数（用户要求使用默认）
- ❌ 不要删除已存在的同名文件（先检查）

### 注意事项
- 输入文件较大 (144 MB mesh)，压缩可能需要时间
- 脚本会自动创建 `.draco` 目录