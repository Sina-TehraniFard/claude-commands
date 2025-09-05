---
description: "事前調査・リサーチ"
argument-hint: "[調査対象] | @[file paths]"
allowed-tools: "Read, Glob, Grep, WebSearch, WebFetch, Task, mcp__report-manager__*"
---

# /research

要件「$ARGUMENTS」の実装前調査を実行します。

### Stage 1: Code Analysis

既存コードベースの関連箇所を調査：

- 関連する既存機能の特定
- 使用中のライブラリ・フレームワークの確認
- 類似実装パターンの調査

### Stage 2: External Research

外部リソースの調査：

- 必要な外部API・サービスの調査
- 技術選定のための情報収集
- 実装方法・ベストプラクティスの調査

### Stage 3: Summary

調査結果をまとめて保存：

- 既存コード分析結果
- 外部リソース調査結果
- 実装に向けた推奨事項
- 潜在的なブロッカーの特定

## 出力

調査結果レポートをMCPで保存し、`/plan`での実装計画立案に活用可能な形式で出力します。

## 制約

- 調査時間: 最大30分
- 焦点: 実装に直結する情報のみ
- 出力: 簡潔で actionable な結果