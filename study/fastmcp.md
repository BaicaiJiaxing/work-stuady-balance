要想实现postman debug：
1.设置无状态模式
`mcp = FastMCP("po-mcp-server",stateless_http=True)`
2.传入参数时必须加id
3.传入调方法参数之前需要先进行对话
`{

    "jsonrpc": "2.0",

    "method": "tools/list",

    "id": "list-tools-request"

}`
