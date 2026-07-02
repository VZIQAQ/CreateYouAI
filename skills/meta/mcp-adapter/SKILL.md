---
name: MCP 适配器
layer: meta
category: mcp
status: unverified
description: >
  将 MCP 工具转换为本地工具，支持工具发现和调用。
  当用户需要连接 MCP 服务器、适配外部工具、构建工具链时触发。
  关键词：MCP、工具适配、工具发现、协议转换、外部工具集成。
requirements:
  - name: mcp
    version: ">=1.0.0"
    required: true
  - name: httpx
    version: ">=0.25.0"
    optional: true
  - name: websockets
    version: ">=12.0"
    optional: true
---

# MCP 工具适配器

将 Model Context Protocol (MCP) 工具转换为本地可调用的工具接口。

## 能力概览

| 能力 | 说明 |
|------|------|
| 工具发现 | 发现 MCP 服务器提供的工具 |
| 工具转换 | 将 MCP 工具转换为本地函数 |
| 参数验证 | 基于 JSON Schema 验证参数 |
| 工具调用 | 支持 HTTP/Stdio/WebSocket 传输 |
| 缓存 | 工具列表缓存和刷新 |

## 前置条件

- Python 3.8+
- MCP SDK（mcp 包）

## 安装步骤

```bash
pip install mcp>=1.0.0

# 可选：HTTP 传输
pip install httpx>=0.25.0

# 可选：WebSocket 传输
pip install websockets>=12.0
```

## 使用方法

### 发现 MCP 工具

```python
import asyncio
from typing import Dict, Any, List

async def discover_mcp_tools_http(server_url: str) -> List[Dict[str, Any]]:
    """通过 HTTP 发现 MCP 服务器提供的工具"""
    import httpx
    
    async with httpx.AsyncClient(timeout=30.0) as client:
        response = await client.get(f"{server_url}/tools")
        if response.status_code == 200:
            return response.json().get("tools", [])
    return []

# 使用示例
tools = asyncio.run(discover_mcp_tools_http("http://localhost:8000"))
```

### MCP 工具适配器

```python
import asyncio
import json
import hashlib
import logging
from typing import Dict, Any, List, Optional, Callable
from dataclasses import dataclass, field
from datetime import datetime
from pathlib import Path

@dataclass
class MCPTool:
    """MCP 工具描述"""
    name: str
    description: str
    parameters: Dict[str, Any]
    server_url: str
    transport_type: str = "http"  # http, stdio, websocket
    last_updated: datetime = field(default_factory=datetime.now)

@dataclass
class ToolCallResult:
    """工具调用结果"""
    success: bool
    result: Any = None
    error: Optional[str] = None
    duration_ms: float = 0
    tool_name: str = ""


class MCPAdapter:
    """MCP 工具适配器"""
    
    def __init__(self, cache_dir: str = ".mcp_cache"):
        self.cache_dir = Path(cache_dir)
        self.cache_dir.mkdir(exist_ok=True)
        self.tools: Dict[str, MCPTool] = {}
        self.logger = logging.getLogger(__name__)
    
    async def connect_server(self, server_url: str, transport: str = "http", **kwargs) -> bool:
        """
        连接到 MCP 服务器
        
        Args:
            server_url: 服务器地址
            transport: 传输类型（http, stdio, websocket）
            **kwargs: 额外参数（stdio 模式需要 command 参数）
        """
        try:
            if transport == "http":
                return await self._connect_http(server_url)
            elif transport == "stdio":
                command = kwargs.get("command", server_url)
                return await self._connect_stdio(command)
            elif transport == "websocket":
                return await self._connect_websocket(server_url)
            else:
                self.logger.error(f"不支持的传输类型: {transport}")
                return False
        except Exception as e:
            self.logger.error(f"连接失败: {e}")
            return False
    
    async def _connect_http(self, server_url: str) -> bool:
        """HTTP 传输连接"""
        try:
            import httpx
        except ImportError:
            raise ImportError("请安装 httpx: pip install httpx>=0.25.0")
        
        async with httpx.AsyncClient(timeout=30.0) as client:
            response = await client.get(f"{server_url}/tools")
            if response.status_code == 200:
                tools_data = response.json()
                for tool_data in tools_data.get("tools", []):
                    tool = MCPTool(
                        name=tool_data["name"],
                        description=tool_data.get("description", ""),
                        parameters=tool_data.get("parameters", {}),
                        server_url=server_url,
                        transport_type="http"
                    )
                    self.tools[tool.name] = tool
                
                self._cache_tools(server_url)
                return True
        return False
    
    async def _connect_stdio(self, command: str) -> bool:
        """Stdio 传输连接"""
        try:
            from mcp import ClientSession, StdioServerParameters
            from mcp.client.stdio import stdio_client
            
            parts = command.split()
            server_params = StdioServerParameters(
                command=parts[0],
                args=parts[1:] if len(parts) > 1 else [],
            )
            
            async with stdio_client(server_params) as (read, write):
                async with ClientSession(read, write) as session:
                    await session.initialize()
                    tools = await session.list_tools()
                    
                    for tool in tools.tools:
                        mcp_tool = MCPTool(
                            name=tool.name,
                            description=tool.description or "",
                            parameters=tool.inputSchema or {},
                            server_url=command,
                            transport_type="stdio"
                        )
                        self.tools[tool.name] = mcp_tool
                    
                    self._cache_tools(command)
                    return True
        except Exception as e:
            self.logger.error(f"Stdio 连接失败: {e}")
            return False
    
    async def _connect_websocket(self, server_url: str) -> bool:
        """WebSocket 传输连接"""
        try:
            import websockets
            
            async with websockets.connect(server_url) as websocket:
                await websocket.send(json.dumps({"type": "discover"}))
                response = await websocket.recv()
                data = json.loads(response)
                
                for tool_data in data.get("tools", []):
                    tool = MCPTool(
                        name=tool_data["name"],
                        description=tool_data.get("description", ""),
                        parameters=tool_data.get("parameters", {}),
                        server_url=server_url,
                        transport_type="websocket"
                    )
                    self.tools[tool.name] = tool
                
                self._cache_tools(server_url)
                return True
        except Exception as e:
            self.logger.error(f"WebSocket 连接失败: {e}")
            return False
    
    def list_tools(self) -> List[MCPTool]:
        """列出所有已发现的工具"""
        return list(self.tools.values())
    
    def get_tool(self, tool_name: str) -> Optional[MCPTool]:
        """获取指定工具"""
        return self.tools.get(tool_name)
    
    async def call_tool(self, tool_name: str, **kwargs) -> ToolCallResult:
        """调用 MCP 工具"""
        tool = self.tools.get(tool_name)
        if not tool:
            return ToolCallResult(
                success=False,
                error=f"工具未找到: {tool_name}",
                tool_name=tool_name
            )
        
        start_time = datetime.now()
        
        try:
            if tool.transport_type == "http":
                result = await self._call_http(tool, kwargs)
            elif tool.transport_type == "stdio":
                result = await self._call_stdio(tool, kwargs)
            elif tool.transport_type == "websocket":
                result = await self._call_websocket(tool, kwargs)
            else:
                raise ValueError(f"未知传输类型: {tool.transport_type}")
            
            duration = (datetime.now() - start_time).total_seconds() * 1000
            return ToolCallResult(success=True, result=result, duration_ms=duration, tool_name=tool_name)
        except Exception as e:
            duration = (datetime.now() - start_time).total_seconds() * 1000
            return ToolCallResult(success=False, error=str(e), duration_ms=duration, tool_name=tool_name)
    
    async def _call_http(self, tool: MCPTool, params: Dict[str, Any]) -> Any:
        """通过 HTTP 调用工具"""
        import httpx
        
        async with httpx.AsyncClient(timeout=30.0) as client:
            response = await client.post(
                f"{tool.server_url}/tools/{tool.name}/call",
                json={"parameters": params}
            )
            
            if response.status_code == 200:
                return response.json().get("result")
            else:
                raise Exception(f"HTTP 错误: {response.status_code}")
    
    async def _call_stdio(self, tool: MCPTool, params: Dict[str, Any]) -> Any:
        """通过 Stdio 调用工具"""
        raise NotImplementedError(
            "Stdio 调用需要完整的会话管理。"
            "请使用 HTTP 或 WebSocket 传输，或实现完整的 stdio 会话。"
        )
    
    async def _call_websocket(self, tool: MCPTool, params: Dict[str, Any]) -> Any:
        """通过 WebSocket 调用工具"""
        import websockets
        
        async with websockets.connect(tool.server_url) as websocket:
            await websocket.send(json.dumps({
                "type": "call",
                "tool": tool.name,
                "parameters": params
            }))
            
            response = await websocket.recv()
            data = json.loads(response)
            
            if data.get("success"):
                return data.get("result")
            else:
                raise Exception(data.get("error", "调用失败"))
    
    def _get_cache_key(self, server_url: str) -> str:
        """生成稳定的缓存文件名"""
        return hashlib.md5(server_url.encode()).hexdigest()
    
    def _cache_tools(self, server_url: str) -> None:
        """缓存工具列表"""
        cache_file = self.cache_dir / f"{self._get_cache_key(server_url)}.json"
        tools_data = [
            {
                "name": tool.name,
                "description": tool.description,
                "parameters": tool.parameters,
                "server_url": tool.server_url,
                "transport_type": tool.transport_type,
                "last_updated": tool.last_updated.isoformat()
            }
            for tool in self.tools.values()
            if tool.server_url == server_url
        ]
        
        with open(cache_file, 'w', encoding='utf-8') as f:
            json.dump(tools_data, f, ensure_ascii=False, indent=2)
    
    def load_cached_tools(self, server_url: str) -> bool:
        """从缓存加载工具"""
        cache_file = self.cache_dir / f"{self._get_cache_key(server_url)}.json"
        
        if not cache_file.exists():
            return False
        
        try:
            with open(cache_file, 'r', encoding='utf-8') as f:
                tools_data = json.load(f)
            
            for tool_data in tools_data:
                tool = MCPTool(
                    name=tool_data["name"],
                    description=tool_data.get("description", ""),
                    parameters=tool_data.get("parameters", {}),
                    server_url=tool_data.get("server_url", server_url),
                    transport_type=tool_data.get("transport_type", "http"),
                    last_updated=datetime.fromisoformat(tool_data.get("last_updated", datetime.now().isoformat()))
                )
                self.tools[tool.name] = tool
            
            return True
        except Exception as e:
            self.logger.error(f"加载缓存失败: {e}")
            return False
    
    def _generate_annotations(self, parameters: Dict[str, Any]) -> Dict[str, type]:
        """根据参数 schema 生成类型注解"""
        annotations = {}
        
        # 兼容 JSON Schema 格式和扁平格式
        properties = parameters.get("properties", parameters)
        
        for param_name, param_info in properties.items():
            param_type = param_info.get("type", "string") if isinstance(param_info, dict) else "string"
            
            type_map = {
                "string": str,
                "integer": int,
                "number": float,
                "boolean": bool,
                "array": list,
                "object": dict,
            }
            annotations[param_name] = type_map.get(param_type, Any)
        
        return annotations
    
    def wrap_as_local_function(self, tool_name: str) -> Optional[Callable]:
        """将 MCP 工具包装为本地函数"""
        tool = self.tools.get(tool_name)
        if not tool:
            return None
        
        async def tool_function(**kwargs):
            result = await self.call_tool(tool_name, **kwargs)
            if result.success:
                return result.result
            else:
                raise Exception(result.error)
        
        tool_function.__name__ = tool_name
        tool_function.__doc__ = tool.description
        tool_function.__annotations__ = self._generate_annotations(tool.parameters)
        
        return tool_function
    
    def create_tool_wrapper(
        self, 
        tool_name: str, 
        validator: Optional[Callable] = None
    ) -> Callable:
        """
        创建带验证的工具包装器
        
        Args:
            tool_name: 工具名称
            validator: 参数验证函数，返回 {"valid": bool, "error": str} 或 bool
        """
        original_func = self.wrap_as_local_function(tool_name)
        if not original_func:
            raise ValueError(f"工具未找到: {tool_name}")
        
        async def validated_wrapper(**kwargs):
            if validator:
                validation_result = validator(kwargs)
                # 支持 bool 或 dict 返回值
                if isinstance(validation_result, dict):
                    if not validation_result.get("valid", True):
                        raise ValueError(f"参数验证失败: {validation_result.get('error')}")
                elif isinstance(validation_result, bool):
                    if not validation_result:
                        raise ValueError("参数验证失败")
            
            return await original_func(**kwargs)
        
        validated_wrapper.__name__ = f"validated_{tool_name}"
        validated_wrapper.__doc__ = f"经过验证的 {tool_name} 包装器"
        
        return validated_wrapper
```

## 使用示例

```python
async def main():
    adapter = MCPAdapter()
    
    # 连接到 HTTP 服务器
    await adapter.connect_server("http://localhost:8000", transport="http")
    
    # 连接到 stdio 服务器
    await adapter.connect_server(
        server_url="python -m mcp_server",
        transport="stdio",
        command="python -m mcp_server"
    )
    
    # 列出工具
    tools = adapter.list_tools()
    print(f"可用工具: {[t.name for t in tools]}")
    
    # 包装为本地函数
    local_func = adapter.wrap_as_local_function("example_tool")
    
    # 调用工具
    result = await adapter.call_tool("example_tool", input="test")

asyncio.run(main())
```

## 问题排查

| 问题 | 原因 | 解决 |
|------|------|------|
| 连接超时 | 服务器未启动或网络问题 | 检查服务器状态 |
| 工具未发现 | 协议版本不兼容 | 确认 mcp>=1.0.0 |
| 工具调用失败 | 参数不匹配 | 检查参数 schema |
| Stdio 调用失败 | 未实现完整会话 | 使用 HTTP 或 WebSocket |
| 缓存失效 | hash() 随机化 | 已改用 hashlib.md5 |

## 依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| Python | 3.8+ | 运行环境 |
| mcp | ≥1.0.0 | MCP 协议 SDK |
| httpx | ≥0.25.0 | HTTP 传输（可选） |
| websockets | ≥12.0 | WebSocket 传输（可选） |
