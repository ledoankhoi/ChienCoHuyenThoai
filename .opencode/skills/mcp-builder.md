---
name: mcp-builder
description: Xây dựng MCP (Model Context Protocol) server — từ ý tưởng đến triển khai
---

Skill này được lấy từ [anthropics/skills/mcp-builder](../../anthropic-skills/skills/mcp-builder/SKILL.md).

## Mô tả
Hướng dẫn thiết kế và xây dựng MCP server: resources, tools, prompts, authentication, deployment.

## Cách dùng
- Load skill khi cần tạo MCP server mới
- Dùng cho cả Node.js và Python implementations

## Kiến trúc
- Resources: dữ liệu có cấu trúc (files, DB records)
- Tools: functions AI có thể gọi
- Prompts: template cho LLM
- Transport: stdio, SSE, hoặc HTTP
