# Phase 1 Roadmap: 杭州多方案验证

## 1. 核心目标
在不涉及前端UI开发的前提下，基于 Docker 在本地部署 OpenTripPlanner (OTP)，加载杭州市的公交与地铁数据，跑通基础路由查询。重点验证两大核心痛点：
1. 单次查询返回至少 10 条出行方案。
2. 通过修改配置放宽过滤逻辑，展示出商业导航工具通常不推荐的“绕路或多换乘”方案。

## 2. 边界与范围
- 区域限定：仅限杭州市内的公交与地铁网络，暂不包含跨城火车与公共自行车（GBFS）。
- 运行环境：仅在本地电脑或服务器通过 Docker 运行，通过 GraphQL API 获取数据。
- 非目标：不涉及地图前端开发，不涉及国内地图合规问题（仅内部技术验证）。

## 3. 任务拆解与交付物

### 3.1 数据准备 (发起人负责)
- 获取 CPTOND-2025 数据集中杭州市的公交与地铁 Shapefile。
- 编写 Python 脚本将其转换为 GTFS 格式（注：因公开数据缺乏逐站时刻表，转换脚本将生成频率近似 GTFS，足以验证路网连通性，但不反映真实运营时间）。
- 交付物：`hangzhou.gtfs.zip`

### 3.2 环境部署 (开发者负责)
- 使用 Osmium-Tool 裁剪浙江省 OSM PBF 数据，保留杭州范围。
- 编写 Docker Compose 文件，挂载 OTP 镜像（建议 v2.9.0 及以上版本）。
- 建议 JVM 堆内存分配为 5G（视实际构建情况可进一步裁剪 OSM 数据）。
- 交付物：`docker-compose.yml` 与本地可运行的 OTP 服务。

### 3.3 参数配置 (开发者负责)
- 修改 `router-config.json`，开启调试模式以保留被过滤的方案。
- 关键配置参考：
```json
{
  "routingDefaults": {
    "numItineraries": 50,
    "itineraryFilters": {
      "debug": "LIST_ALL",
      "groupSimilarityKeepOne": 0.5,
      "transitGeneralizedCostLimit": {
        "costLimitFunction": "1.5 * x + 3600",
        "intervalRelaxFactor": 0.2
      }
    }
  }
}
