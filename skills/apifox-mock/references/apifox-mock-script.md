# Apifox Mock 脚本

根据接口文件或类型定义，生成 Apifox 高级 Mock 脚本代码（JavaScript），适用于需要**字段关联**、**条件逻辑**、**读取请求参数**等复杂 Mock 场景。

## 与 Mock JSON 的区别

| 特性         | Mock JSON      | Mock 脚本                              |
| ------------ | -------------- | -------------------------------------- |
| 字段关联     | ❌ 不支持      | ✅ 支持（如 type=2 时 url 为图片链接） |
| 条件逻辑     | ❌ 不支持      | ✅ 支持（if/else 判断）                |
| 读取请求参数 | ❌ 不支持      | ✅ 支持（分页、筛选等）                |
| 复杂嵌套     | 有限           | ✅ 完全支持                            |
| 使用场景     | 简单接口       | 复杂接口、关联字段                     |

## 参数

| 参数       | 必填 | 说明                                  | 示例                                                                      |
| ---------- | ---- | ------------------------------------- | ------------------------------------------------------------------------- |
| `file`     | ✅   | 接口文件路径或类型定义文件路径        | `@src/api/chat/controller/.../getChatAiChatV1MessagesByConversationId.ts` |
| `count`    | ❌   | 生成数据条数（针对列表类型），默认 12 | `10`                                                                      |
| `关联字段` | ❌   | 指定需要关联的字段组                  | `attachments的type/url/mimeType关联`                                      |

## 脚本模板结构

```javascript
/* eslint-disable */
// ========================================
// Apifox 高级 Mock 脚本
// 接口：{接口名称}
// 路径：{接口路径}
// ========================================

// ==================== 数据模板 ====================

var xxxTemplates = {
  1: [{ /* 类型1的数据 */ }],
  2: [{ /* 类型2的数据 */ }],
};

var contentTemplates = ['模板1内容', '模板2内容'];

// ==================== 工具函数 ====================

function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

function randomPick(arr) {
  return arr[randomInt(0, arr.length - 1)];
}

function randomDate() {
  var d = new Date(Date.now() - randomInt(0, 30 * 24 * 60 * 60 * 1000));

  return d.toISOString().replace('T', ' ').slice(0, 19);
}

// ==================== 数据生成函数 ====================

function generateXxx() {
  // 生成关联数据的逻辑
}

function generateRecords(count) {
  var records = [];

  for (var i = 0; i < count; i++) {
    records.push({
      id: i + 1,
      // ... 其他字段
    });
  }

  return records;
}

// ==================== 主逻辑 ====================

var size = $$.mockRequest.getParam('size') || 12;
var current = $$.mockRequest.getParam('current') || 1;

var responseData = {
  code: 200,
  bizCode: 0,
  data: {
    records: generateRecords(Number.parseInt(size)),
    total: 36,
    size: Number.parseInt(size),
    current: Number.parseInt(current),
    searchCount: true,
    pages: 3,
  },
  msg: 'success',
};

$$.mockResponse.setBody(responseData);
```

## 字段关联示例

### 附件类型关联

当 `type` 字段决定 `url`、`mimeType`、`name` 时：

```javascript
/* eslint-disable */
var attachmentTemplates = {
  1: [
    { type: 1, url: 'https://example.com/files/readme.txt', mimeType: 'text/plain', name: '说明文档.txt', size: 2048 },
    { type: 1, url: 'https://example.com/files/guide.md', mimeType: 'text/markdown', name: '使用手册.md', size: 4096 },
  ],
  2: [
    { type: 2, url: 'https://picsum.photos/800/600', mimeType: 'image/png', name: '产品截图.png', size: 524288 },
    { type: 2, url: 'https://picsum.photos/1200/800', mimeType: 'image/jpeg', name: '设计稿.jpg', size: 1048576 },
  ],
  3: [
    { type: 3, url: 'https://example.com/files/report.pdf', mimeType: 'application/pdf', name: '项目报告.pdf', size: 2097152 },
  ],
};

function generateAttachments() {
  var count = randomInt(0, 3);
  var attachments = [];

  for (var i = 0; i < count; i++) {
    var type = randomInt(1, 3);
    var template = randomPick(attachmentTemplates[type]);

    attachments.push({
      type: template.type,
      url: template.url,
      mimeType: template.mimeType,
      name: template.name,
      size: randomInt(template.size * 0.5, template.size * 1.5),
    });
  }

  return attachments;
}
```

### 状态关联

当 `status` 决定 `statusText` 时：

```javascript
/* eslint-disable */
var statusMap = { 1: '待处理', 2: '处理中', 3: '已完成', 4: '已取消' };

function generateRecord() {
  var status = randomInt(1, 4);

  return {
    status: status,
    statusText: statusMap[status],
  };
}
```

## Apifox 脚本 API 参考

### 请求对象 `$$.mockRequest`

```javascript
/* eslint-disable */
var userId = $$.mockRequest.getParam('userId');           // query/path 参数
var token = $$.mockRequest.headers.get('Authorization');  // Header
var sessionId = $$.mockRequest.cookies.get('sessionId');  // Cookie
var body = $$.mockRequest.body.toJSON();                  // JSON Body
var bodyStr = $$.mockRequest.body.toString();              // String Body
var file = $$.mockRequest.body.formdata.get('file');       // FormData
```

### 响应对象 `$$.mockResponse`

```javascript
$$.mockResponse.setBody({ id: 1, name: 'test' });        // 设置 JSON 响应
$$.mockResponse.setBody('Hello World!');                   // 设置字符串响应
$$.mockResponse.setCode(200);                              // 设置状态码
$$.mockResponse.setDelay(1000);                            // 设置延时（ms）
$$.mockResponse.headers.add({ key: 'X-Custom', value: 'value' }); // 设置 Header
```

## 语法限制

Apifox 脚本环境兼容性有限，注意以下约束：

1. **使用 `var` 声明**：不使用 `const`/`let`
2. **使用 `function` 声明**：不使用箭头函数
3. **字符串换行**：长文本使用 `\n` 转义，不使用模板字符串
4. **避免 ES6+ 语法**：不使用解构、展开运算符等
5. **关联字段模板化**：将关联字段组织为 key-value 对象，根据 key 选择完整数据
