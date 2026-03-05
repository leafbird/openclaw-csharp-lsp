---
name: csharp-lsp
slug: csharp-lsp
version: 1.1.0
description: "C# language server providing code intelligence, diagnostics, and navigation for .cs and .csx files. Uses csharp-ls (lightweight Roslyn-based). Requires .sln or .csproj."
metadata:
  openclaw:
    emoji: "💜"
    requires:
      bins: ["dotnet"]
    os: ["linux", "darwin", "win32"]
    install:
      - id: dotnet
        kind: manual
        label: "Install .NET SDK (https://dot.net/download)"
---

# C# LSP

C# code intelligence via [csharp-ls](https://github.com/razzmatazz/csharp-language-server) — lightweight Roslyn-based language server.

## Capabilities

- **Go-to-definition**: Jump to symbol definitions across solution
- **Find references**: Locate all usages of a symbol
- **Hover info**: Type signatures, XML docs, parameter info
- **Diagnostics**: Real-time compiler errors and warnings
- **Document symbols**: List classes, methods, properties in a file
- **Workspace symbol search**: Find symbols across the entire solution

Supported extensions: `.cs`, `.csx`

## Prerequisites

- **.NET SDK** (9.0+): https://dot.net/download
- **Python 3**: lsp-query 데몬 실행에 필요
- **`.sln` or `.csproj`**: 프로젝트 파일이 있어야 동작 (loose .cs 파일은 제한적)

## Setup

원타임 셋업 스크립트 실행 (멱등성 보장 — 재실행 안전):

```bash
bash {baseDir}/scripts/setup.sh           # 셋업만
bash {baseDir}/scripts/setup.sh --verify  # 셋업 + 동작 검증
```

수행 내용:
1. .NET SDK 존재 확인
2. `csharp-ls` 설치 (`dotnet tool install --global`)
3. `~/.dotnet/tools` PATH 등록
4. `lsp-query` 심볼릭 링크 생성
5. 캐시 디렉토리 생성

## Usage

```bash
# 워크스페이스 지정 (.sln이 있는 디렉토리)
export LSP_WORKSPACE=/path/to/project

# 정의로 이동
lsp-query definition src/Program.cs 15 8

# 참조 찾기
lsp-query references src/Models/User.cs 42 10

# 타입/시그니처 확인
lsp-query hover src/Services/AuthService.cs 30 22

# 파일 내 심볼 목록
lsp-query symbols src/Program.cs

# 워크스페이스 심볼 검색
lsp-query workspace-symbols "UserService"

# 진단 (컴파일 에러/경고)
lsp-query diagnostics src/Program.cs

# 실행 중인 서버 확인
lsp-query servers

# 데몬 종료
lsp-query shutdown
```

Line/column 번호는 **1-indexed**.

## Architecture

```
lsp-query CLI → Unix Socket → lsp-query 데몬 (Python)
                                   ↓
                              csharp-ls (subprocess, stdin/stdout JSON-RPC)
                                   ↓
                              Roslyn (.sln → 전체 타입 시스템)
```

- **데몬**: 첫 호출 시 자동 fork. 5분 유휴 시 자동 종료.
- **csharp-ls**: C# 쿼리 시 자동 기동. 5분 유휴 시 종료.
- **Cold start**: 솔루션 로딩에 30~60초 (대규모 프로젝트). 이후 쿼리 ~200ms.

## Project Detection

1. **Solution file** (`.sln`) — 최적: 크로스 프로젝트 참조 가능
2. **Project file** (`.csproj`) — 양호: 단일 프로젝트 분석
3. **Loose `.cs` files** — 제한적: 기본 구문만

## What's Included

```
{baseDir}/
├── SKILL.md              # This file
└── scripts/
    ├── setup.sh          # 원타임 셋업 (멱등성 보장)
    └── lsp-query.py      # LSP 데몬 + CLI (자체 포함)
```

## Troubleshooting

- **설치 실패 (`DotnetToolSettings.xml`)**: `dotnet tool install --global csharp-ls --version 0.20.0` 으로 버전 고정
- **빈 결과**: `.sln` 또는 `.csproj`가 있는 디렉토리를 `LSP_WORKSPACE`로 지정했는지 확인
- **첫 쿼리 느림**: Roslyn 프로젝트 로딩 중. 대규모 솔루션은 30~60초. 이후 빠름.
- **PATH 문제**: `export PATH="$PATH:$HOME/.dotnet/tools"` 를 shell profile에 추가
- **stale 데몬**: `lsp-query shutdown` 후 재시도

## Links

- [csharp-ls on GitHub](https://github.com/razzmatazz/csharp-language-server)
- [.NET SDK Downloads](https://dot.net/download)
- [LSP Specification](https://microsoft.github.io/language-server-protocol/)
