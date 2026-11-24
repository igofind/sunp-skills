# 批量调拨 — 数据导入

<!-- ═══ SECTION: 2 | 数据导入 | status: done ═══ -->
<!-- DEPENDS_ON: none -->
<!-- REFERENCES: none -->

## 2 数据导入

用户上传 Excel 文件，系统解析并导入数据。

<!-- ═══ SECTION: 2.1 | ImportController | status: done ═══ -->
<!-- DEPENDS_ON: none -->

### 2.1 ImportController

处理导入的 HTTP 请求，调用 Service 层。

<!-- ═══ SECTION: 2.1.1 | importData | status: done ═══ -->
<!-- DEPENDS_ON: 2.2.1 -->

#### 2.1.1 importData

接收上传的 Excel 文件，调用 Service 导入。

##### 实现细节
- [x] 接收 MultipartFile 参数，校验文件非空
- [x] 校验文件后缀为 .xlsx/.xls
- [x] 校验文件大小 ≤ 5MB
- [x] 调用 ImportService.importData(file)
- [x] 返回导入结果（成功数、失败数、错误文件下载链接）

##### 已知边界条件
- [x] 文件为空时，返回 400 + "文件不能为空"
- [x] 文件格式错误时，返回 400 + "仅支持 xlsx/xls 格式"

<!-- ═══ END: 2.1.1 ═══ -->

<!-- ═══ END: 2.1 ═══ -->

<!-- ═══ SECTION: 2.2 | ImportService | status: done ═══ -->
<!-- DEPENDS_ON: none -->

### 2.2 ImportService

处理导入的核心业务逻辑。

<!-- ═══ SECTION: 2.2.1 | importData | status: done ═══ -->
<!-- DEPENDS_ON: none -->
<!-- REFERENCES: shared#name-to-code -->

#### 2.2.1 importData

解析 Excel 并逐行导入。

##### 实现细节
- [x] 使用 EasyExcel 读取文件，逐行解析
- [x] 空行过滤：跳过所有字段为空的行
- [x] 名称→编码转换：调用 NameToCodeConverter 转换业务字段
- [x] 名称找不到时：标记为校验错误，不中断导入
- [x] 名称有多个匹配时：取第一个，记录警告日志
- [x] 必填字段校验：名称、数量不能为空
- [x] 重复数据检测：同一编码+同一日期不允许重复
- [x] 超过 1000 行时，返回进度回调
- [x] 所有行解析完成后，批量插入数据库
- [x] 返回导入结果（成功数、失败数、错误详情列表）

##### 已知边界条件
- [x] Excel 中有合并单元格时，按首行值填充
- [x] 编码转换失败的行，归入错误列表
- [x] 数据库插入部分失败时，已成功的行回滚

##### 校验记录
- ✅ 全部通过

<!-- ═══ END: 2.2.1 ═══ -->

<!-- ═══ END: 2.2 ═══ -->

<!-- ═══ END: 2 ═══ -->
