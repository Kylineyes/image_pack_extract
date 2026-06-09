# image_pack_extract

`image_pack_extract` 用于扫描指定输入目录中的 PDF 文件，提取 PDF 内嵌图片，并按配置输出为目录或压缩包。

该工具面向批量处理场景：单个 PDF 处理失败会记录错误，但不会中断后续文件。

## 功能

- 递归扫描 `.pdf` 和 `.PDF` 文件。
- 支持使用正则表达式过滤 PDF 文件名。
- 支持用正则捕获组格式化输出目录名或压缩文件名。
- 提取匹配 PDF 中的内嵌图片。
- 支持配置图片文件名，例如 `0001.png` 或 `image-0001.jpg`。
- 支持输出模式：
  - `none`：直接输出图片目录。
  - `zip`：输出 zip 压缩包，可设置 AES-256 密码。
  - `lz4`：输出 `tar.lz4` 压缩包。
  - `zstd`：输出 `tar.zst` 压缩包。

## 使用方式

基于 `config-example.yaml` 创建配置文件，然后运行：

```bash
go run ./cmd/image-pack-extract -config ./config.yaml
```

构建二进制：

```bash
go build -o image-pack-extract ./cmd/image-pack-extract
./image-pack-extract -config ./config.yaml
```

程序会输出每个 PDF 的处理结果和最终汇总：

```text
extracted: input/book.pdf -> output/book (12 images)
Processed PDFs: 1
Extracted images: 12
Failed PDFs: 0
Output: ./output
```

## 配置说明

最小目录输出配置：

```yaml
input_folder: ./input
output_folder: ./output
input_filter_template: '(\d+)\.(\d+)\.(\d+)《(.+)》.+\.pdf'
output_format: "%04d.%02d.%02d-%s"
output_filename_format: "%04d"
compress_type: none
compress_level: 0
compress_password: ""
password: ""
```

对于文件名：

```text
2009.05.29《1Q84》村上春树.pdf
```

示例正则会捕获：

```text
1=2009, 2=05, 3=29, 4=1Q84
```

最终输出：

```text
output/2009.05.29-1Q84/
  0001.png
  0002.jpg
```

### 字段

`input_folder`：PDF 输入根目录。程序会递归扫描该目录。

`output_folder`：输出根目录。根据压缩类型，里面会生成图片目录或压缩文件。

`input_filter_template`：可选的 PDF 文件名正则表达式。只匹配文件名，不匹配完整路径。空字符串表示处理所有 PDF。

`output_format`：必填，使用 `fmt.Sprintf` 规则生成输出目录名或压缩文件名。参数来自 `input_filter_template` 的捕获组；数字捕获组会自动转为整数，所以可以使用 `%04d`、`%02d`。

`output_filename_format`：必填，使用 `fmt.Sprintf` 规则生成图片文件名，不包含后缀。参数只有一个：当前 PDF 内的图片序号。

`compress_type`：压缩类型，可选值为 `none`、`zip`、`lz4`、`zstd`。

`compress_level`：压缩等级。`compress_type` 为 `none` 时忽略。`lz4` 按 `0-9` 归一化；`zstd` 映射到最快、默认、更高压缩率、最高压缩率几个档位；`zip` 当前使用默认 deflate 设置。

`compress_password`：压缩密码。空字符串表示不设置密码。目前仅 `zip` 支持密码。

`password`：PDF 打开密码。没有密码的 PDF 请设置为空字符串。

## 压缩示例

输出 zip：

```yaml
output_format: "%04d.%02d.%02d-%s.zip"
compress_type: zip
compress_password: ""
```

输出带密码的 zip：

```yaml
output_format: "%04d.%02d.%02d-%s.zip"
compress_type: zip
compress_password: "secret"
```

输出 zstd：

```yaml
output_format: "%04d.%02d.%02d-%s.tar.zst"
compress_type: zstd
compress_level: 5
```

输出 lz4：

```yaml
output_format: "%04d.%02d.%02d-%s.tar.lz4"
compress_type: lz4
compress_level: 1
```

## 开发

运行测试：

```bash
go test ./...
```

格式化代码：

```bash
gofmt -w ./cmd ./internal
```
